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

### Customize a `linux-xlnx-dev` kernel `uImage`

The idea is to create a custom kernel where the support of Coresight components is enabled. Unfortunately, we cannot customize the usual `core-image-minimal` image with Yocto tools. Fortunately, we can do it on a `linux-xlnx-dev` image!

```bash
bitbake linux-xlnx-dev -c menuconfig
```

![menu](./img/menu.png)

Go to `Kernel hacking => CoreSight Tracing Support` and enable the components you need (everything is enabled here). Then, save and exit.

![menu-coresight](./img/menu-coresight.png)

```bash
bitbake linux-xlnx-dev -c compile -f
bitbake linux-xlnx-dev -c deploy
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

We can see some Coresight support in the kernel but no devices detected... And, well, there are several reasons for that :fearful:

## Activate Coresight components - Hardware side
### Yocto `meta-xilinx` layer and Xilinx Linux kernel
During the compilation process, the [meta-xilinx](https://github.com/Xilinx/meta-xilinx/tree/zeus/) was cloned (`zeus` branch).

[https://github.com/Xilinx/meta-xilinx/tree/zeus/meta-xilinx-bsp/conf/machine/zedboard-zynq7.conf](https://github.com/Xilinx/meta-xilinx/tree/zeus/meta-xilinx-bsp/conf/machine/zedboard-zynq7.conf):
```bash
require conf/machine/include/machine-xilinx-default.inc
# Skipping a few lines
KERNEL_DEVICETREE = "zynq-zed.dtb"
```

[https://github.com/Xilinx/meta-xilinx/tree/zeus/meta-xilinx-bsp/conf/machine/include/machine-xilinx-default.inc](https://github.com/Xilinx/meta-xilinx/tree/zeus/meta-xilinx-bsp/conf/machine/include/machine-xilinx-default.inc):

```bash
# Kernel Configuration
XILINX_DEFAULT_KERNEL := "linux-xlnx"
XILINX_DEFAULT_KERNEL_microblaze := "linux-yocto"
XILINX_DEFAULT_KERNEL_zynqmp := "linux-yocto"
PREFERRED_PROVIDER_virtual/kernel ??= "${XILINX_DEFAULT_KERNEL}"
```

According to the BitBake files, it will clone the `xlnx_rebase_v4.19` branch of the Linux kernel provided by Xilinx ([https://github.com/Xilinx/linux-xlnx/tree/xlnx_rebase_v4.19](https://github.com/Xilinx/linux-xlnx/tree/xlnx_rebase_v4.19)).

[https://github.com/Xilinx/meta-xilinx/tree/zeus/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx_2019.2.bb](https://github.com/Xilinx/meta-xilinx/tree/zeus/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx_2019.2.bb):
```bash
LINUX_VERSION = "4.19"
KBRANCH ?= "xlnx_rebase_v4.19"
```
Then, if we look at the device tree file common to all Zynq 7000-based boards ([https://github.com/Xilinx/linux-xlnx/blob/xlnx_rebase_v4.19/arch/arm/boot/dts/zynq-7000.dtsi](https://github.com/Xilinx/linux-xlnx/blob/xlnx_rebase_v4.19/arch/arm/boot/dts/zynq-7000.dtsi)):
**No mention of Coresight components at all** :fearful:

Which in not the case in the `master` branch ([https://github.com/Xilinx/linux-xlnx/blob/master/arch/arm/boot/dts/zynq-7000.dtsi](https://github.com/Xilinx/linux-xlnx/blob/master/arch/arm/boot/dts/zynq-7000.dtsi)):
```bash
# [...]
		etb@f8801000 {
			compatible = "arm,coresight-etb10", "arm,primecell";
			reg = <0xf8801000 0x1000>;
			clocks = <&clkc 27>, <&clkc 46>, <&clkc 47>;
			clock-names = "apb_pclk", "dbg_trc", "dbg_apb";
			in-ports {
				port {
					etb_in_port: endpoint {
						remote-endpoint = <&replicator_out_port1>;
					};
				};
			};
		};

		tpiu@f8803000 {
			compatible = "arm,coresight-tpiu", "arm,primecell";
			reg = <0xf8803000 0x1000>;
			clocks = <&clkc 27>, <&clkc 46>, <&clkc 47>;
			clock-names = "apb_pclk", "dbg_trc", "dbg_apb";
			in-ports {
				port {
					tpiu_in_port: endpoint {
						remote-endpoint = <&replicator_out_port0>;
					};
				};
			};
		};

		funnel@f8804000 {
			compatible = "arm,coresight-static-funnel", "arm,primecell";
			reg = <0xf8804000 0x1000>;
			clocks = <&clkc 27>, <&clkc 46>, <&clkc 47>;
			clock-names = "apb_pclk", "dbg_trc", "dbg_apb";

			/* funnel output ports */
			out-ports {
				port {
					funnel_out_port: endpoint {
						remote-endpoint =
							<&replicator_in_port0>;
					};
				};
			};
# [...]
```
This is mainly due to the fact that the kernel version in the [meta-xilinx](https://github.com/Xilinx/meta-xilinx/blob/master/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx_2020.1.bb) layer has been updated later in the `master branch`:
```diff
commit 48998ea5b79af13fb1b181037cd3a12990463e8a
Author: Sai Hari Chandana Kalluri <chandana.kalluri@xilinx.com>
Date:   Fri Feb 21 11:20:37 2020 -0800

    Update KERNEL_VERSION to 5.4
    
    Update KERNEL_VERSION to 5.4
    
    Signed-off-by: Sai Hari Chandana Kalluri <chandana.kalluri@xilinx.com>

diff --git a/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx_2020.1.bb b/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx_2020.1.bb
index 58f163c..2b8763a 100644
--- a/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx_2020.1.bb
+++ b/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx_2020.1.bb
@@ -1,4 +1,4 @@
-LINUX_VERSION = "4.19"
+LINUX_VERSION = "5.4"
 KBRANCH ?= "xlnx_rebase_v4.19"
 SRCREV ?= "b983d5fd71d4feaf494cdbe0593ecc29ed471cb8"
```

Furthermore, this has been updated only in the `5.6` release of the Linux kernel: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/commit/?id=9b0b308a1586f620a49c50605ae8abf509190661&h=master

```diff
--- a/arch/arm/boot/dts/zynq-7000.dtsi
+++ b/arch/arm/boot/dts/zynq-7000.dtsi
@@ -59,6 +59,39 @@
 		regulator-always-on;
 	};
 
+	replicator {
+		compatible = "arm,coresight-static-replicator";
+		clocks = <&clkc 27>, <&clkc 46>, <&clkc 47>;
+		clock-names = "apb_pclk", "dbg_trc", "dbg_apb";
+
+		out-ports {
+			#address-cells = <1>;
+			#size-cells = <0>;
+
+			/* replicator output ports */
+			port@0 {
+				reg = <0>;
+				replicator_out_port0: endpoint {
+					remote-endpoint = <&tpiu_in_port>;
+				};
+			};
+			port@1 {
+				reg = <1>;
+				replicator_out_port1: endpoint {
+					remote-endpoint = <&etb_in_port>;
+				};
+			};
+		};
+		in-ports {
+			/* replicator input port */
+			port {
+				replicator_in_port0: endpoint {
+					remote-endpoint = <&funnel_out_port>;
+				};
+			};
+		};
+	};
+
 	amba: amba {
 		compatible = "simple-bus";
 		#address-cells = <1>;
@@ -365,5 +398,107 @@
 			reg = <0xf8005000 0x1000>;
 			timeout-sec = <10>;
 		};
+
+		etb@f8801000 {
+			compatible = "arm,coresight-etb10", "arm,primecell";
+			reg = <0xf8801000 0x1000>;
+			clocks = <&clkc 27>, <&clkc 46>, <&clkc 47>;
+			clock-names = "apb_pclk", "dbg_trc", "dbg_apb";
+			in-ports {
+				port {
+					etb_in_port: endpoint {
+						remote-endpoint = <&replicator_out_port1>;
+					};
+				};
+			};
+		};
+
+		tpiu@f8803000 {
+			compatible = "arm,coresight-tpiu", "arm,primecell";
+			reg = <0xf8803000 0x1000>;
+			clocks = <&clkc 27>, <&clkc 46>, <&clkc 47>;
+			clock-names = "apb_pclk", "dbg_trc", "dbg_apb";
+			in-ports {
+				port {
+					tpiu_in_port: endpoint {
+						remote-endpoint = <&replicator_out_port0>;
+					};
+				};
+			};
+		};
+
+		funnel@f8804000 {
+			compatible = "arm,coresight-static-funnel", "arm,primecell";
+			reg = <0xf8804000 0x1000>;
+			clocks = <&clkc 27>, <&clkc 46>, <&clkc 47>;
+			clock-names = "apb_pclk", "dbg_trc", "dbg_apb";
+
+			/* funnel output ports */
+			out-ports {
+				port {
+					funnel_out_port: endpoint {
+						remote-endpoint =
+							<&replicator_in_port0>;
+					};
+				};
+			};
+
+			in-ports {
+				#address-cells = <1>;
+				#size-cells = <0>;
+
+				/* funnel input ports */
+				port@0 {
+					reg = <0>;
+					funnel0_in_port0: endpoint {
+						remote-endpoint = <&ptm0_out_port>;
+					};
+				};
+
+				port@1 {
+					reg = <1>;
+					funnel0_in_port1: endpoint {
+						remote-endpoint = <&ptm1_out_port>;
+					};
+				};
+
+				port@2 {
+					reg = <2>;
+					funnel0_in_port2: endpoint {
+					};
+				};
+				/* The other input ports are not connect to anything */
+			};
+		};
+
+		ptm@f889c000 {
+			compatible = "arm,coresight-etm3x", "arm,primecell";
+			reg = <0xf889c000 0x1000>;
+			clocks = <&clkc 27>, <&clkc 46>, <&clkc 47>;
+			clock-names = "apb_pclk", "dbg_trc", "dbg_apb";
+			cpu = <&cpu0>;
+			out-ports {
+				port {
+					ptm0_out_port: endpoint {
+						remote-endpoint = <&funnel0_in_port0>;
+					};
+				};
+			};
+		};
+
+		ptm@f889d000 {
+			compatible = "arm,coresight-etm3x", "arm,primecell";
+			reg = <0xf889d000 0x1000>;
+			clocks = <&clkc 27>, <&clkc 46>, <&clkc 47>;
+			clock-names = "apb_pclk", "dbg_trc", "dbg_apb";
+			cpu = <&cpu1>;
+			out-ports {
+				port {
+					ptm1_out_port: endpoint {
+						remote-endpoint = <&funnel0_in_port1>;
+					};
+				};
+			};
+		};
 	};
 };
```

As a consequence, we need to clone the `master` branch of [linux-xlnx](https://github.com/Xilinx/linux-xlnx/branches) in order to get a correct device tree.
This can be done by modifying a parameter in [machine-xilinx-default.inc](https://github.com/Xilinx/meta-xilinx/blob/master/meta-xilinx-bsp/conf/machine/include/machine-xilinx-default.inc):
```diff
# Kernel Configuration
- PREFERRED_PROVIDER_virtual/kernel ??= "linux-xlnx"
+ PREFERRED_PROVIDER_virtual/kernel ??= "linux-xlnx-dev"
```
This information was given in a kernel recipe file of the [meta-xilinx](https://github.com/Xilinx/meta-xilinx/blob/master/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx-dev.bb) layer.

### Recompiling everything...

Now that we've found when and where the device tree has been updated with Coresight components, we have two tasks to do:

- Recompile a default `core-image-minimal` with `linux-xlnx-dev` kernel.
- Customize `linux-xlnx-dev` in order to include Coresight drivers.

Then, once everything is copied on the SD card:

```bash
Starting kernel ...

Booting Linux on physical CPU 0x0
Linux version 5.4.0-xilinx-dev (oe-user@oe-host) (gcc version 9.2.0 (GCC)) #1 SMP PREEMPT Mon Jul 6 16:40:45 UTC 2020
[...]
coresight etm0: PTM 1.0 initialized
coresight etm1: PTM 1.0 initialized
fpga_manager fpga0: Xilinx Zynq FPGA Manager registered
[...]
Starting syslogd/klogd: done

Poky (Yocto Project Reference Distro) 3.0.3 zedboard-zynq7 /dev/ttyPS0

zedboard-zynq7 login: root
root@zedboard-zynq7:~# ls /sys/bus/coresight/devices/
etb0         etm0         etm1         funnel0      replicator0  tpiu0
```

Yay, we can see Coresight components on the Zedboard! :dark_sunglasses:

[![asciicast](https://asciinema.org/a/345848.png)](https://asciinema.org/a/345848)
