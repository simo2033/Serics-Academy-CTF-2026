# [PWN] Overflow the Gate

# Description 

A remote service is running on the server and accepts user input over a TCP connection.

You can connect to it using:

```ruby
nc overflow-the-gate.sactf.staccahstaccah.it 1337
```

The service appears simple, but it was compiled with modern protections enabled. Careful analysis and precise control of execution flow will be required to retrieve the flag.

You are provided with the binary and the required runtime files to reproduce the service locally.
