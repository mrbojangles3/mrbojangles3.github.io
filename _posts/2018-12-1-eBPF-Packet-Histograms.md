---
title: "Packet Size Histograms with eBPF and XDP"
categories:
  - Blog
tags:
  - eBPF
  - networking
  - network visibility
---
# Packet Histograms with eBPF XDP

## Intro
I have been reading a lot of articles about the eXpress Data Path(XDP) and I would like to learn my about it. I decided to write a program that will track the size of packets inbound to my NIC, and increment a counter for the corresponding "bin" size. I have my MTU set to 1500 and I not worried about packets that are smaller than 64 bytes.

## Quick intro to eBPF and XDP
eBPF is kind of like an in-kernel virtual machine. It allows a programmer to write code that the kernel will execute in kernel space at various hook-points in the kernel. There is a good FAQ of what eBPF is in the [kernel docs](https://www.kernel.org/doc/html/latest/bpf/bpf_design_QA.html?highlight=bpf#bpf-design-q-a)
### 2 Types of code in XDP examples

I adapted my code from the `xdp1` example given in the kernel source located in the `samples/bpf`. That directory has C files named similarly, with the difference being the suffix, `_kern.c` and `_user.c`. The `_kern` code is the code that will be translated to the eBPF instructions then compiled to machine native code and executed in the kernel, the `_user` is code that is executed in user-space.

## XDP Code
The code examples below utilize the API for XDP programs. From the FAQ above, we know that not all kernel code is available to XDP programs. The example kernel code does some simple math for the packet size and will atomically increment the appropriate counter. The counters are values in the eBPF maps. The user space program will read those counters from the map via file descriptors. Links to documentation are provided as comments to the code. The userspace code include code from `bpf_load.h` which assists in in communicating with the eBPF map.
### XDP Kernel Code

```
/* Copyright (c) 2016 PLUMgrid
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of version 2 of the GNU General Public
 * License as published by the Free Software Foundation.
 */
#define KBUILD_MODNAME "foo"
#include <uapi/linux/bpf.h>
#include <linux/in.h>
#include <linux/if_ether.h>
#include <linux/if_packet.h>
#include <linux/if_vlan.h>
#include <linux/ip.h>
#include <linux/ipv6.h>
#include "bpf_helpers.h"

struct bpf_map_def SEC("maps") size_hist_map = {
	.type = BPF_MAP_TYPE_ARRAY,
	.key_size = sizeof(u32),
	.value_size = sizeof(u64),
	.max_entries = 5,
};


// Still looking for where this call is attached to in the networking stack
SEC("xdp1")
// xdp_md is documented in the kernel
// https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/bpf.h#L2416
int xdp_prog1(struct xdp_md *ctx)
{
	long* value;
	u32 len = ctx->data_end - ctx->data;
	u32 key = 0;
	if(len >= 64 && len < 128){
		key = 0;
	}else if(len >= 128 && len < 256){
		key = 1;
	}else if(len >= 256 && len < 512){
		key = 2;
	}else if(len >= 512 && len < 1024){
		key = 3;
	}else if(len >= 1024 && len <=1500){
		key = 4;
	}
	value = bpf_map_lookup_elem(&size_hist_map, &key);
	if(value){
		// LLVM maps this to built-in bpf atomic add instructions,
		// BPF_STX | BPF_XADD | BPF_W for word sizes
		// according to https://cilium.readthedocs.io/en/latest/bpf/#llvm
		__sync_fetch_and_add(value,1);
	}
	return XDP_PASS;
}

char _license[] SEC("license") = "GPL";
```

### XDP User Code

```
/* Copyright (c) 2016 PLUMgrid
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of version 2 of the GNU General Public
 * License as published by the Free Software Foundation.
 */
#include <linux/bpf.h>
#include <linux/if_link.h>
#include <assert.h>
#include <errno.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <libgen.h>
#include <sys/resource.h>

#include "bpf_util.h"
#include "bpf/bpf.h"
#include "bpf/libbpf.h"

static int ifindex;
static __u32 xdp_flags;

static void int_exit(int sig)
{
	bpf_set_link_xdp_fd(ifindex, -1, xdp_flags);
	exit(0);
}
char* str_arr[5] = {
	"64 - 127",
	"128 - 255",
	"256 - 511",
	"512 - 1023",
	"1024 - 1500"};
/* simple size counter
 */
static void poll_stats(int map_fd, int interval)
{
	const unsigned int nr_keys = 5;
	__u64 values[nr_keys];
	__u32 key;

	while (1) {
		sleep(interval);

		for (key = 0; key < nr_keys; key++) {
			assert(bpf_map_lookup_elem(map_fd, &key, (&values[key])) == 0);
			printf("range %s: %10llu\n",str_arr[key],values[key]);
		}
	}
}

static void usage(const char *prog)
{
	fprintf(stderr,
		"usage: %s [OPTS] IFINDEX\n\n"
		"OPTS:\n"
		"    -S    use skb-mode\n"
		"    -N    enforce native mode\n",
		prog);
}

int main(int argc, char **argv)
{
	struct rlimit r = {RLIM_INFINITY, RLIM_INFINITY};
    // This is an XDP program
	struct bpf_prog_load_attr prog_load_attr = {
		.prog_type	= BPF_PROG_TYPE_XDP,
	};
	const char *optstr = "SN";
	int prog_fd, map_fd, opt;
	struct bpf_object *obj;
	struct bpf_map *map;
	char filename[256];

	while ((opt = getopt(argc, argv, optstr)) != -1) {
		switch (opt) {
		case 'S':
			xdp_flags |= XDP_FLAGS_SKB_MODE;
			break;
		case 'N':
			xdp_flags |= XDP_FLAGS_DRV_MODE;
			break;
		default:
			usage(basename(argv[0]));
			return 1;
		}
	}

	if (optind == argc) {
		usage(basename(argv[0]));
		return 1;
	}

	if (setrlimit(RLIMIT_MEMLOCK, &r)) {
		perror("setrlimit(RLIMIT_MEMLOCK)");
		return 1;
	}

	ifindex = strtoul(argv[optind], NULL, 0);

	/* argv[0] is the name of the executable, good way to find the object file */
	snprintf(filename, sizeof(filename), "%s_kern.o", argv[0]);
	prog_load_attr.file = filename;

	if (bpf_prog_load_xattr(&prog_load_attr, &obj, &prog_fd))
		return 1;

	map = bpf_map__next(NULL, obj);
	if (!map) {
		printf("finding a map in obj file failed\n");
		return 1;
	}
	map_fd = bpf_map__fd(map);

	if (!prog_fd) {
		printf("load_bpf_file: %s\n", strerror(errno));
		return 1;
	}

	signal(SIGINT, int_exit);
	signal(SIGTERM, int_exit);

    // this function is defined in bpf_load.c like all the user-space helpers
	if (bpf_set_link_xdp_fd(ifindex, prog_fd, xdp_flags) < 0) {
		printf("link set xdp fd failed\n");
		return 1;
	}

	poll_stats(map_fd, 2);

	return 0;
}
```

### XDP compile and running
To compile the code, modify the Makefile in `samples/bpf` to include your `_kern.c` and your `_user.c` code. Grepping the Makefile shows the lines I had to add:
``` Makefile
hostprogs-y += xdp_count
xdp_count-objs := xdp_count_user.o
always += xdp_count_kern.o
```
Ensure that you follow the directions in the readme of the `samples/bpf` directory and run `make headers_install` at the top level of your kernel source directory before compile the samples directory.
#### Compile Errors
I did have some compile errors. The errors came from my `_kernel` code, googling did help, but the answers weren't as easy to find. I ended up seeing the errors as github issues. It is possible that the bpftool could have been of help. My errors came from not using the correct map access function. The other xdp examples and the cillium docs helped.

#### Running
To run: `sudo ./xdp_count -S 3`. The `-S 3` tells the xdp program to use "skb" mode as opposed to a mode that is hardware accelerated. The ifindex of the NIC can be found using the `ip` command.

### Output
This is the output when I am streaming music, the user-space program polls the map every two seconds:
```
range 64 - 127:        506
range 128 - 255:         17
range 256 - 511:         17
range 512 - 1023:          0
range 1024 - 1500:          1
range 64 - 127:        511
range 128 - 255:         17
range 256 - 511:         20
range 512 - 1023:          0
range 1024 - 1500:          1
range 64 - 127:        516
range 128 - 255:         17
range 256 - 511:         20
range 512 - 1023:          0
range 1024 - 1500:          1
```
This is the output when I am watching an XDP related youtube video:
```
range 64 - 127:         10
range 128 - 255:          6
range 256 - 511:          1
range 512 - 1023:          1
range 1024 - 1500:         79
range 64 - 127:         37
range 128 - 255:         11
range 256 - 511:          1
range 512 - 1023:          4
range 1024 - 1500:       1488
range 64 - 127:         57
range 128 - 255:         17
range 256 - 511:          2
range 512 - 1023:          6
range 1024 - 1500:       2454
```
It makes sense that the youtube video would have bigger packets than music streaming.

## XDP Attachment Point
I am still searching for the attachment point of the XDP program. I read that the program executes when the packet "comes off the NIC" but I am trying to find where specifically.

## References/ Further reading
There are a lot of resources out there I will link to several that have helped me:
* [the elixir from bootlin](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/bpf.h)
* [the cillium docs](https://cilium.readthedocs.io/en/latest/bpf/)
* [prototype-kernel docs](https://prototype-kernel.readthedocs.io/en/latest/bpf/index.html)
* [the tc-bpf man page](http://man7.org/linux/man-pages/man8/tc-bpf.8.html)
* [the various presentations on the iovisor github](https://github.com/iovisor/bpf-docs)
