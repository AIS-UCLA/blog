---
title: "Scalable authentication with LDAP"
author: Christopher Milan
---

Our server cluster uses the Lightweight Directory Access Protocol (LDAP) to
manage users simultaneously across machines. Setting up LDAP is non-trivial,
but also quite useful, so here I will document our setup process.

# OpenLDAP Server Setup

Our LDAP server is hosted on `sullivan`, a FreeBSD host. Many guides exist
showing how to setup an OpenLDAP server on FreeBSD, so it would serve no one
for me to write another copy. However, I will detail a few of our specific
configuration choices.

Firstly, **YOU SHOULD USE TLS!** Without TLS/SSL, passwords will be transmitted
in the clear. TLS is recommended for OpenLDAP, so this is what we've decided to
use. Since we only have 4 servers acting as OpenLDAP clients, we found self-
signed certificates to be a relatively minor inconvience (each client must
manually trust the root CA), but you may consider setting up Lets Encrypt's
`certbot` if you'd prefer to not self-sign.

Next, `slapd` configuration options. Remember to set these in `slapd.ldif`,
as `slapd.conf` is deprecated. Also, changing these options down the line is
doable, but much trickier than just getting them right the first time! We use
the dynamic `mdb` config module to allow later editing, but it's much more
annoying than changing the first-time setup options in `slapd.ldif`. As for our
schema includes, we use `core.ldif`, `cosine.ldif`, `inetorgperson.ldif`, and
`nis.ldif`. Remember to create a rootdn with password!

Moving on to our LMDB database, we set some basic ACLs. Here's one you
definitely want, which allows users to authenticate, but also prevents
others from viewing their passwords:

```
olcAccess: to dn.subtree="ou=Users,dc=ais-ucla,dc=org"
        attrs=userPassword
        by self write
        by anonymous auth
        by * none
```

We also set the `objectClass` for `ou=Users` to be `posixAccount` (which
includes neccesary data like `uid` and `loginShell`) and `inetOrgPerson`,
which allows us to store additional information about users, such as phone
numbers, short descriptions, etc. The full specification for inetOrgPerson is
available [here](https://datatracker.ietf.org/doc/html/rfc2798).

# Ldapscripts Setup

Now that the OpenLDAP server is setup, while we could manage users entirely
through the `ldapsearch` and `ldapmodify` tools, it's easier to use scripts
that abstract away these clunky interfaces. Most operating systems include a
copy of `martymac`'s `ldapscripts` collection in their package manager, which
provides a collection of tools we find quite useful. In particular,
`ldapadduser`, `ldapaddgroup`, and the like, are all great for those used to
`adduser` and `usermod`.

`ldapscripts` is quite customizable, and while obviously you must set it up to
be able to connect to your LDAP server (usually with rootdn access), there are
some specific configuration options that may be useful to explicitly note:

 * Setting `CREATEHOMES` to `"yes"`, surprisingly enough, creates home
   directories automatically upon user creation.
 * We use `PASSWORDGEN="cat /dev/random | LC_ALL=C tr -dc 'a-zA-Z0-9' | head -c8"`
   to create randomized passwords (users connecting over ssh are forced to use
   pubkey authentication anyway).
 * Enabling `UTEMPLATE` allows for additonal configuration of user accounts
   when `ldapadduser` is run. Here is the template we use:
```
dn: uid=<user>,<usuffix>,<suffix>
objectClass: posixAccount
objectClass: inetOrgPerson
cn: <ask>
sn: <ask>
uid: <user>
uidNumber: <uid>
gidNumber: <gid>
homeDirectory: <home>
loginShell: /bin/bash
gecos: <user>
description: <ask>
o: <ask>
mobile: <ask>
```

# OpenLDAP client setup

Finally, each machine on your network (that you want users to access), must
have an OpenLDAP client installed properly. While you should look for a guide
specific to your operating system, here are some brief notes on configuration
options.

 * Remember to enable TLS/SSL (eg. `ssl start_tls`)!
 * Make sure to reconfigure `nsswitch.conf` to ensure your ldap client is
   actually being used.
 * On Linux, setup PAM to use ldap too. For instance, on Ubuntu, it suffices
   to edit `/etc/pam.d/common-session`, which is included by the other PAM
   config files. Also ensure that ssh is actually using PAM!

# Final notes

Remember that writing your own LDAP scripts is also an option, if `ldapscripts`
isn't flexible enough for you, or is missing features. For instance, we provide
a tool, `pb`, which allows users to view information about others based on
their username (see [here](https://github.com/ais-ucla/sys)). Additionally,
ensuring you have physical access to these machines, with at least one account
that isn't an LDAP account is paramount: in the event of a network failure or
misconfigured firewall you don't want to be locked out of all your servers!

