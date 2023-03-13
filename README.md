# elimo-yocto-manifest

Elimo Initium/Impetus manifest repo for OpenEmbedded/Yocto.

## Description

This is the Elimo manifest repo for the Initium EVK and, more generally, Elimo Impetus based devices; this is our approach to bringing together the various layers required for a complete Yocto build.

As you know a Yocto build is, at its core, a set of layers and configuration that is given to bitbake as an input to get images as an output; in most cases, these elements will be spread over several `git` repos, which is a maintenance nightmare unless you're just testing a one-off build. There are various approaches to bringing this all together; this repo currently supports two of these:

- Google Repo
- git submodules

If you want to build an image to get started with your Initium, this repo is what you are looking for. 

:warning: If you want to create your own distribution or customise the BSP, you should have a look at our other two repos that this one brings together: [meta-elimo-distro](https://github.com/elimo-engineering/meta-elimo-distro) (the distro layer) and [meta-elimo](https://github.com/elimo-engineering/meta-elimo) (the BSP layer). 


## Getting started

This repo is essentially two repos rolled into one, so that you can choose two different approaches depending on your preferences or development environment needs.

In this repo you have submodules that point to the repos of all required layers. This allows you to just `git clone --recursive` this repo and get everything in one go.

In the root directory you also have `elimo-yocto-manifest.xml`, a manifest file for the Google Repo tool. We won't go into the details here, but `repo` is a tool to clone and maintain a set of related git repos, as defined in an XML file (the manifest) that references all of them and the revisions that need to be checked out. You can use`repo init` and `repo sync` to clone all layer repos at the correct revision. [Here](https://medium.com/@villebaillie25/revision-control-of-your-embedded-linux-system-34bf4d5c7979) is a good quick intro to `repo`.

Let's go into the practicalities of making a build. 
You need to choose between the two approaches described above in step 1. Both approaches work the same way after that.

Note: 

### Step 1a: get the layers (submodules edition!)

```shell
git clone --recursive https://github.com/elimoengineering/elimo-yocto-manifest.git -b kirkstone-v1.0.0
```

### Step 2: clone the dependencies

```shell
git clone git://git.yoctoproject.org/poky -b kirkstone-4.0.5
git clone https://git.openembedded.org/meta-openembedded.git -b kirkstone
git clone https://github.com/linux-sunxi/meta-sunxi.git -b kirkstone
git clone https://github.com/elimoengineering/meta-elimo.git -b kirkstone-v1.0.0
```

### Step 3: initialise a build environment

```shell
. meta-elimo-distro/oe-init-build-env build-elimo-initium
```

You are probably familiar with this if you have used `poky` before.

This script will initialise the environment of your shell so that you can call `bitbake`; it will also give you a default build configuration in `build-elimo-initium/conf/local.conf` and set up your build to reference the appropriate layer dependencies in `build-elimo-initium/conf/bblayers.conf`.

### Step 4: build!

You are now ready to start a build; all usual builds from Poky, from which Elimo Poky is derived, are possible. For a (relatively!) quick test build, you can use `core-image-minimal`, which will give you a textual console on both the USB-UART bridge and the LCD on the Initium EVK.

```shell
bitbake core-image-minimal
```

### Step 5: deploying to SD card

If you're doing this in a Linux environment, you can use the following process to transfer the image to an SD Card.
In this example we're using the `core-image-minimal-elimo-initium.wic` image, if you have built a different image, update the paths accordingly.

First check your SD card path using `lsblk`; in the following commands, replace <X> with your results from `lsblk`.

#### With `dd`

`dd` is already installed on most systems, so this is probably the quicest way to go:

```shell
tmp/deploy/images/elimo-initium
sudo dd if=core-image-minimal-elimo-initium.wic of=/dev/sd<X> bs=4M iflag=fullblock oflag=direct conv=fsync status=progress
```

#### With `bmaptool`

[BMAP Tools](https://github.com/intel/bmap-tools) is a tool specifically designed by Intel to flash embedded system images to mass storage. It has several advantages over `dd`, including efficiency over sparse images, built-in verification and the ability to fetch images remotely over HTTPS, ssh and other protocols.
It is available on Debian based systems to be installed with a simple `sudo apt install bmap-tools`

```shell
cd tmp/deploy/images/elimo-initium
sudo umount /dev/sd<X>
sudo bmaptool copy core-image-minimal-elimo-initium.wic.gz --bmap core-image-minimal-elimo-initium.wic.bmap /dev/sd<X>
```


### Step 6: run!

That is all, you should be able to insert the SD Card in the SD Card slot on the Impetus and run.


## Defining your own machine

If you are using the Impetus SoM on your own carrier board, you will most likely want to create a custom `MACHINE` to support the specific hardware on your carrier board. The approach we suggest to this end is to create your own BSP layer based on our one, [meta-elimo](https://github.com/elimo-engineering/meta-elimo)`, either by:

- forking `meta-elimo` and modifying your fork. This is quicker and probably more convenient for small changes.
- creating a new layer altogether and making it depend on `meta-elimo`. You can model this on how `meta-elimo` depends on `meta-sunxi`. This approach is probably more flexible, but requires a bit more initial work.

Once that is done, you can replace `meta-elimo` with your own BSP in the layer config for your build, using `bitbake-layers` (or manually editing `conf/bblayers.conf`)


## Troubleshooting and bug reporting

We hope troubleshooting won't be necessary, but the reality of engineering is that likely it will be. 
If you find an issue in using this code, please create a bug report on the [GitHub issue tracker](https://github.com/elimo-engineering/meta-elimo-distro/issues)


## Submitting patches

Please submit any patches against the `meta-elimo-distro` layer to the [GitHub repo](https://github.com/elimo-engineering/meta-elimo-distro/pulls) as a PR.

Maintainer: Matteo Scordino <matteo@elimo.io>k
