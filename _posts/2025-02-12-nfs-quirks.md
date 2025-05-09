---
title: "Quirks resulting from mounting an NFS at $HOME"
author: Christopher Milan
---

While mounting our NFS at `/home/` is convenient for users to share their data
between machines relatively transparently, there are some issues that crop up
from time to time. I want to document a list of these issues and mitigations
that I've come up with, so I'll put that here.

 * `opam` can't enable switches:
   It seems that `opam` is not designed to run on an NFS, or maybe more
   precisely, our NFS has too high latency for `opam` to work well. As such,
   putting your `OPAMROOT` on a different filesystem (namely `/scratch`) is
   recommended. This can be done by adding the following line to your `.bashrc`,
   `.zshrc`, etc:
```
export OPAMROOT=/scratch/$USER/.opam
```
   Alternatively, you can check out something like
   [`nfsopam.sh`](//github.com/UnixJunkie/nfsopam).
* `ghc` via `ghcup`:
  `ghcup` is the recommended tool to manage `ghc` versions. Upon install, it
  reads the environment variable `GHCUP_INSTALL_BASE_PREFIX`, which allows you
  to select an alternative base installation location, probably a good idea,
  because Haskell compilation is already slow enough. Notably, this needs to be
  set when running `ghcup` commands as well as during install, so it is best to
  put this in your `.bashrc`/`.zshrc`/etc:
```
export GHCUP_INSTALL_BASE_PREFIX=/scratch/$USER
```

