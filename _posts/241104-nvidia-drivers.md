# Setting up nvidia server drivers on Ubuntu 24.04

Christopher Milan
11/04/2024

## Normal installation:

Normally, nvidia drivers can installed simply by installing the corresponding
metapackage for your GPU(s). These can be found using something like:

```bash
apt-cache search ^nvidia server metapackage
```

This should give you a list of potential packages to install. You probably want
to have DKMS and the open kernel version, so assuming you want version `550`,
you'd install `nvidia-driver-550-server-open`.

Remember to also installed the corresponding version of `nvidia-utils` (ie.
`nvidia-utils-550-server`) and also `nvidia-cuda-toolkit`.

## P2P installation:

It stands to reason that you may benefit from installing tinygrad's
[P2P-enabled driver](https://github.com/tinygrad/open-gpu-kernel-modules).
At time of writing, this kernel driver patch only exists for two driver versions.
However, applying the patch to a newer driver version should be relatively
straightforward (in fact, easier than convincing `apt` to install an old version
of the nvidia driver, which is required to ensure the your userspace will work).
Here are some existing patch versions (not necessarily up to date):

 * [550.54.15](https://github.com/tinygrad/open-gpu-kernel-modules/tree/550.54.15-p2p)
 * [550.90.07](https://github.com/tinygrad/open-gpu-kernel-modules/tree/550.90.07-p2p)
 * [550.90.12](https://github.com/AIS-UCLA/open-gpu-kernel-modules/tree/550.90.12-p2p)
 * [550.127.05](https://github.com/AIS-UCLA/open-gpu-kernel-modules/tree/550.127.05-p2p)

Follow the README at these links for installation instructions. If you installed
a DKMS driver, don't forget to remove that driver with `dkms remove`.

