# Activate Coresight components in a Yocto environment for the Zedboard board


## Activate Coresight components - Software side - Method #1

### Generate a `core-image-minimal`

Just follow https://pcotret.github.io/yocto-zedboard-101/

### Customize a `linux-xlnx` kernel `uImage`

The idea is to create a custom kernel where the support of Coresight components is enabled. Unfortunately, we cannot customize the usual `core-image-minimal` image with Yocto tools. Fortunately, we can do it on a `linux-xlnx` image!

## Activate Coresight components - Hardware side

:fearful:
