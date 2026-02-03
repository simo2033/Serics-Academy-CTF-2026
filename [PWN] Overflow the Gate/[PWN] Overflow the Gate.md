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









