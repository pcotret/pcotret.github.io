# Yocto on the Zedboard 101


## 1. Introduction
Yocto provides tools and metadata for creating custom embedded systems with following main features :

- Images are tailored to specific hardware and use cases
- But metadata is generally arch-independent
- Unlike a distro, *kitchen sink* is not included (we know what we need in advance)

Other keywords and their meanings are explained here:

- An image is a collection of *baked* recipes (packages)
- A 'recipe' is a set of instructions for building *packages*
  - Where to get the source and which patches to apply
  - Dependencies (on libraries or other recipes, for example)
  - Config/compile options, *install* customization
- A *layer* is a logical collection of recipes representing the core, a board support package (BSP), or an application stack

### 1.1. Packages to be installed
```bash
sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
     build-essential chrpath socat cpio python python3 python3-pip python3-pexpect \
     xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev \
     pylint3 xterm
```

### 1.2. Clone Yocto and recipes (zeus branch)
```bash
git clone -b zeus https://git.yoctoproject.org/git/poky
cd poky
# Next clones must be done in the poky folder
git clone -b zeus https://github.com/Xilinx/meta-xilinx
git clone -b zeus https://github.com/openembedded/meta-openembedded.git
```

### 1.3. Build configuration
```bash
source oe-init-build-env
```
- Modifications in `conf/local.conf`:

```bash
# Change the default target
$ echo "MACHINE??=\"zedboard-zynq7\"" >> conf/local.conf
# Add a few lines
$ echo "IMAGE_FEATURES += \"package-management\"" >> conf/local.conf
$ echo "DISTRO_HOSTNAME = \"zynq\"" >> conf/local.conf
```

- You may also like to change the package type from `rpm` to `deb`: modify `PACKAGE_CLASSES '= "package_rpm"` by `PACKAGE_CLASSES '= "package_deb"`

- Modifications in `conf/bblayers.conf` in order to add the cloned layers:
```
$ bitbake-layers add-layer ../meta-xilinx/meta-xilinx-bsp/
$ bitbake-layers add-layer ../meta-openembedded/meta-oe/
```
### 1.4. Generate image (and take a coffee...)
```bash
bitbake core-image-minimal
```
Once complete the images for the target machine will be available in the output directory `poky/build/tmp/deploy/images/zedboard-zynq7`.

## 1.5 Modify the `uEnv.txt` file

We need to modify `uEnv.txt` a little bit according to a Xilinx layer [README-booting.md](https://github.com/Xilinx/meta-xilinx/blob/master/meta-xilinx-bsp/README.booting.md#preparing-sdmmc)

```bash
echo "ramdisk_image=core-image-minimal-zedboard-zynq7.cpio.gz.u-boot" >> tmp/deploy/images/zedboard-zynq7/uEnv.txt
```

## 2. Creating the SD card

## Partitions

Create two partitions on the SD card:

- Partition #1 : 100 MB, fat16.
- Partition #2 : remaining space, ext3.

Copy the following files in partition #1:

```bash
sudo cp boot.bin core-image-minimal-zedboard-zynq7.cpio.gz.u-boot u-boot.img uEnv.txt uImage zynq-zed.dtb /mnt/partition1
```

Extract the `core-image-minimal` archive in partition #2:

```bash
sudo cp core-image-minimal-zedboard-zynq7.tar.gz /mnt/partition2
cd /mnt/partition2
sudo tar xvzf core-image-minimal-zedboard-zynq7.tar.gz 
```

Now, everything should be OK!

```
Poky (Yocto Project Reference Distro) 3.0.3 zedboard-zynq7 /dev/ttyPS0

zedboard-zynq7 login: root
root@zedboard-zynq7:~# cat /proc/cpuinfo 
processor       : 0
model name      : ARMv7 Processor rev 0 (v7l)
BogoMIPS        : 666.66
Features        : half thumb fastmult vfp edsp neon vfpv3 tls vfpd32 
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x3
CPU part        : 0xc09
CPU revision    : 0

processor       : 1
model name      : ARMv7 Processor rev 0 (v7l)
BogoMIPS        : 666.66
Features        : half thumb fastmult vfp edsp neon vfpv3 tls vfpd32 
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x3
CPU part        : 0xc09
CPU revision    : 0

Hardware        : Xilinx Zynq Platform
Revision        : 0003
```

## References
- https://www.youtube.com/watch?v=XPnmB-THjiY

## Other sources (to check later)
- https://blog.mbedded.ninja/programming/embedded-linux/zynq/building-linux-for-the-zynq-zc702-eval-kit-using-yocto/
- https://dwjbosman.github.io/yocto-linux-on-the-xilinx-zynq-zed-board

