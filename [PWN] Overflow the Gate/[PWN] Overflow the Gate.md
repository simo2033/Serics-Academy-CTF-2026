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

To do so will use a cyclic pattern, which is a special string that does not repeat itself and allows us to understand where we are in the stack.

We can check the RIP register that contains the address of the next instruction to be executed.
In a stack buffer overflow, the return address is overwritten and loaded into RIP.
By analyzing the value of RIP after the crash, it is possible to determine when control of the execution flow is obtained.

Execute the program with GDB

`gdb ./chall`

than we run it passing the pattern

```GDB
run < <(python3 -c "from pwn import *; print(cyclic(200))")
```

the program will crash

```GDB
(gdb) run < <(python3 -c "from pwn import *; print(cyclic(200))")
Starting program: /home/simux/OverflowTheGate/chall < <(python3 -c "from pwn import *; print(cyclic(200))")
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
=== Overflow the Gate ===
Welcome!
puts @ 0x7ffff7e0de50
Send your payload:
[*] Checking for new versions of pwntools
    To disable this functionality, set the contents of /home/simux/.cache/.pwntools-cache-3.10/update to 'never' (old way).
    Or add the following lines to ~/.pwn.conf or ~/.config/pwn.conf (or /etc/pwn.conf system-wide):
        [update]
        interval=never
[*] You have the latest version of Pwntools (4.15.0)
Bye!

Program received signal SIGSEGV, Segmentation fault.
0x000000000040125f in vuln ()
```






















