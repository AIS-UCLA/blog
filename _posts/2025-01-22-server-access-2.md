---
title: "(NEW) Connecting to the cluster with cherf"
author: Christopher Milan
---

# Disclaimer

If you haven't already read it, please go read the original instructions on
how to connect to the cluster, which is available
[here](/2024/11/18/server-access.html). This new method is not entirely
foolproof, so the old method should be used as a fallback. Also, the old blog
post has important details on the cluster topology.

# New connection method

I spent some time writing a new tool, [cherf](//github.com/ais-ucla/cherf), to
make connecting to the server easier. The readme over on github has more details
on how it works, but the basic steps to use it are as follows:

1. Build `cherf` from scratch, or download an pre-built binary (on the releases
   page.
2. Add `cherf` binary to `PATH` (and set as executable if you downloaded it).
3. Create an X509 certificate signing request:
```
mkdir ~/.cherf
openssl req -newkey rsa:2048 -keyout ~/.cherf/key -out csr
```
4. Send `csr` to a system administrator (see AI Safety Discord), and you will'
   receive a certificate, which you should save as `~/.cherf/cert`.
5. Download `sullivan`'s SHA1 signature, and save it as
   `~/.cherf/sullivan.sha1`. This is available
   [here](//ais-ucla.org/assets/sullivan.sha1).
5. Edit your ssh config:
```
Host sullivan
  ProxyCommand cherf client ssh cherf.ais-ucla.org 1234 sullivan
  ProxyUseFdpass yes
  ...
```
6. You can now connect (ie. `ssh sullivan`).

## Not working?

Experiencing issues? First visit [this website](//www.checkmynat.com), and
your NAT type is not "Symmetric NAT". If it is not, please open an issue
[here](//github.com/ais-ucla/cherf/issues).

