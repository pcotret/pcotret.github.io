# Activate Coresight components in a Yocto environment for the Zedboard


## Activate Coresight components - Software side - Method #1

### Generate a `core-image-minimal`

Just follow https://pcotret.github.io/yocto-zedboard-101/. When booting this image:

```bash
Poky (Yocto Project Reference Distro) 3.0.3 zedboard-zynq7 /dev/ttyPS0

zedboard-zynq7 login: root
root@zedboard-zynq7:~# ls /sys/bus/
amba          container     event_source  i2c           media         nvmem         scsi          soc           usb
clockevents   cpu           gpio          iio           mmc           pci           sdio          spi           virtio
clocksource   edac          hid           mdio_bus      mmc_rpmb      platform      serio         ulpi          workqueue
```

No Coresight components :cry: ​

### Customize a `linux-xlnx` kernel `uImage`

The idea is to create a custom kernel where the support of Coresight components is enabled. Unfortunately, we cannot customize the usual `core-image-minimal` image with Yocto tools. Fortunately, we can do it on a `linux-xlnx` image!

```bash
bitbake linux-xlnx -c menuconfig
```

![menu](./img/menu.png)

Go to `Kernel hacking => CoreSight Tracing Support` and enable the components you need (everything is enabled here). Then, save and exit.

![menu-coresight](./img/menu-coresight.png)

```bash
bitbake linux-xlnx -c compile -f
bitbake linux-xlnx -c deploy
```

Now, you should have a new `uImage` file in `build/tmp/deploy/images/zedboard-zynq7`. Just replace it in the first partititon of the SD card and boot the board.

```bash
Poky (Yocto Project Reference Distro) 3.0.3 zedboard-zynq7 /dev/ttyPS0

zedboard-zynq7 login: root
root@zedboard-zynq7:/sys/bus/coresight# ll       
drwxr-xr-x    2 root     root           0 Jan  1  1970 devices
drwxr-xr-x    2 root     root           0 Jun 29 21:56 drivers
-rw-r--r--    1 root     root        4.0K Jun 29 21:56 drivers_autoprobe
--w-------    1 root     root        4.0K Jun 29 21:56 drivers_probe
--w-------    1 root     root        4.0K Jun 29 21:56 uevent
root@zedboard-zynq7:/sys/bus/coresight# ll devices/
```

We can see some Coresight support in the kernel but no devices detected...

## Activate Coresight components - Hardware side

:fearful:
