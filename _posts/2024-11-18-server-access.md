---
title: "Connecting to the AI Safety and ACM AI cluster"
author: Christopher Milan
---

I've found myself repeating the same instructions to many people over discord,
so I've decided to compile a list of steps to connect to our servers in one
place.

# Requisites

In order to connect to the server, you will first (obviously) need to have an
account created. Find someone who looks in charge on the AI Safety discord
server, and give them your full name, phone number, desired username, and
ssh public key (if you don't already have one, your friendly neighborhood
LLM can teach you how to create one). Additionally, you will need a computer
with the OpenSSH client installed (most computers are now shipped with this
software pre-installed).

# Network Topology

Before connecting, it is worth understanding how the network is setup. We
currently have 4 computers, three of which are GPU servers (`temescal`,
`ynez`, and `serrano`), and the last of which acts as the login node
(`sullivan`). While all computers are connected to SEASnet and the greater
internet, you must first connect to `sullivan` before you can connect to
the other computers. Additionally, SEASnet has installed a firewall between
`sullivan` and the greater internet, so it is neccesary that you first aquire
an authorized IP address (for most people, this means connecting to the UCLA
VPN). The set of authorized IP addresses may be suprisingly restrictive, so if
you are struggling to connect, your first troubleshooting step should be to
verify your VPN connection.

# Actually connecting!

As described above, you must first connect to `sullivan` before you can connect
to any of the other computers. Doing this manually every time would be quite
annoying, so fortunately OpenSSH provides configuration options to make this
easier:

# SSH Config

Please copy below into your SSH config (ie. `~/.ssh/config`), replacing `USER`
with your (server) username.

```
Host sullivan
  ForwardAgent yes
  HostName sullivan.seas.ucla.edu
  User USER

Host temescal
  ForwardAgent yes
  ProxyJump sullivan
  HostName temescal.seas.ucla.edu
  User USER

Host ynez
  ForwardAgent yes
  ProxyJump sullivan
  HostName ynez.seas.ucla.edu
  User USER

Host serrano
  ForwardAgent yes
  ProxyJump sullivan
  HostName serrano.seas.ucla.edu
  User USER
```

You should now be able to log in to all four computers. Please note that
`sullivan` hosts most of our infrastructure, and is not meant to be used
for development, please use the three GPU servers instead. Additionally,
upon first login, we ask that you run `passwd` to reset your password
(your inital password should have been given to you by the person who
setup your account). You can change your login shell using `chsh.ldap`,
although please note that this change may take a while to replicate.

