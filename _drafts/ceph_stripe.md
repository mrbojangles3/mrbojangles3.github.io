---
layout: post
title: "Using the Ceph libradosstriping"
author: MrBoJangles3
date: 2017-05-28
categories: ceph systems striping
---

# Introduction

This article is going to discuss the libradosstriping API. Striping across objects is desireable as it spreads the IO load across the OSDs. The ceph docs have a [good illustration](http://docs.ceph.com/docs/master/architecture/#data-striping) of this, so I won't go over it here. The libradosstriping API can be seen [here](https://github.com/ceph/ceph/blob/master/src/include/radosstriper/libradosstriper.hpp). For this article I am going to assume that you already have access to a cluster, and it has an erasure coded pool created. If you need steps on created an erasure coded pool,see [here](http://docs.ceph.com/docs/master/rados/operations/pools/). To run the example code below you need to have the rados libraries installed, specifically `libradosstriper1-devel`. This article is using a Ceph Jewel (10.2.5) cluster, with 5 OSDs, and a pool with 256 placement groups.

# The Code

The makefile:

{% gist f77291dad36fbf79691e0a9ba24a64aa %}

The source file:

{% gist 8c95e3072e892195b3cf98f13155cdfc %}

# Rados CLI

The Rados CLI has a `--striper` option. It can be used to cleanup the files you made with a simple `rados -p pool_name --striper rm obj_name`. Also, you can use the `ls` option to see the object you have put into Ceph. If the `--striper` option is not suppiled you can see all of the ceph objects that the stripping API created. I have a feeling this will be usefule in the future when I am looking into the alignment details.


# Commentary

I am still wrapping my head around the idea of `alignment` I know that it is realted to the erasure coding that is done on the pool, but I don't know the factors that influence it. I will be digging into this in a future blog post.


When I ran the example code and used `ceph -w`, I saw both read and write traffic reported, when I was only doing a write. I will be digging into the details of why libradosstriper does this.


I was happy to discover the `bl.read_file` method. That made this example code much cleaner, as I didn't have to worry about reading the file, Ceph takes care of it for me! I have no idea how stable the buffer api is, but you can find it [here](https://github.com/ceph/ceph/blob/master/src/common/buffer.cc#L2117)


Since I am writing to an Erasure Coded pool and don't have the EC overwrite turned on, I use the `write_full` call inside of libradosstriper.


# Thanks

Thanks to James Norman for writing the [post](https://blog.storagemadeeasy.com/writing-to-an-erasure-coded-pool-in-ceph-rados/) at the Storage Made Easy blog. The Alignment tip is excellent.

I would also like to thank [CERN](https://gitlab.cern.ch/castor/CASTOR/blob/e3500a660f718d595cf2cbe7ccffa8a630e6b7b5/ceph/ceph_posix.cpp) for their source file that also uses the striping API, which helped me write this post.

