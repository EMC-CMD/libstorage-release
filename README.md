# libstorage-release
This repository is a [bosh release](https://bosh.io/releases) for [libstorage](https://github.com/emccode/libstorage).

##Prerequisites

The usage of this repository assumes that the developer have context using BOSH, and motiviation to use Libstorage for a storage solution.

This bosh release is open to use any adapter available libstorage client now or in the future. At the time of this writing, Libstorage supports VirtualBox, Isilon, and ScaleIO adapters for storage solutions. We will cover how to write a manifest for this release using Isilon and ScaleIO, and step by step deployment with that manifest.
__NOTE: Using two adapters in our manifest is for clear documentation. One could easily use only Isilon, or only ScaleIO with similar steps.__

#####STEP 1: Clone & upload bosh release

Use the command `git clone https://github.com/EMC-CMD/libstorage-release.git` to clone this repository. 
