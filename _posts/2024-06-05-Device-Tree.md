---
title: "Getting dtbs_check Working"
categories:
  - Blog
tags:
  - Kernel Builds
  - Embedded
  - Device Tree
  - Cross Compile
---
# Just a Quick Note for Myself
I have off and on been working on getting a device tree files submitted for a Solid Run board that I own. I would like to have an easier time running OpenWrt on the board. Uptreaming for the win.

I had a [well short attempt](https://lore.kernel.org/linux-devicetree/ef8eec8a-2ce5-ad1a-afcf-86ee78231017@kernel.org/) to add the device tree file as it came from Solid Run. So after working out the small naming problems I decided I want to try to use the `dtbs_check` functionality in the kernel makefile. In case I don't get this merged upstream I want to document how to get this check running:

1. Get the kernel source tree
2. `make CROSS_COMPILE=aarch64-linux-gnu- ARCH=arm64 KBUILD_OUTPUT=out/ defconfig`
3. `make CHECK_DTBS=y marvell/cn9130-cf-pro.dtb`


# Links for later
* [Devicetree names](https://devicetree-specification.readthedocs.io/en/latest/chapter2-devicetree-basics.html#generic-names-recommendation)
* [Linaro Blog](https://www.linaro.org/blog/tips-and-tricks-for-validating-devicetree-sources-with-the-devicetree-schema/)
