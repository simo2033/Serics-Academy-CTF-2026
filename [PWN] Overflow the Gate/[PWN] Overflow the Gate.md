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

The content of the folder is the following

```
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        19/01/2026     10:09          16360 chall
-a----        19/01/2026     11:03            520 exploit.py
-a----        19/01/2026     10:10         215000 ld-linux-x86-64.so.2
-a----        19/01/2026     10:09        2220400 libc.so.6
```

