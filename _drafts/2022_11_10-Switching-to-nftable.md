---
title: "Switching to nftables"
categories:
  - Blog
tags:
  - System Administration
  - Networking
  - firewall
  - nft
  - firewalld
---
# Why switch to nftables
Short Answer: Cause I want to.
Its not better or worse than firewalld just different in my opinion. I want to know how things work under the covers and I like working with networking. I also like knowing that I am using all the advancements from the netfilter team, instead of waiting for it to work its way into firewalld.

## Uninstalling firewalld
Pretty simple, `dnf remove firewalld`. So far on my laptop that always lives on WiFi at home, doesn't do any podman/docker/virtualization. I haven't had too many issues with another service or process needing the functionality that firewalld provides. But do be aware libvirt and podman / docker probably benefit from firewalld being up and running on the system.

## Installing nftables
It is already installed if you are using firewalld. If not, `dnf install -y nftables`. Take a look at the files included in the rpm with `rpm -ql nftables`. Your new firewall config file lives in `/etc/nftables/`. The package also includes a `systemd` unit file as well. Take a look at it with `systemctl edit nftables.service`.

# Setting up rules
There is a default rule set that is included in the Fedora package of `nftables` it lives in `/etc/nftables/main.nft`. But this is probably not sufficient for your needs. So additional rules need to be created. For inspiration there are a few places to look. The [netfilter wiki](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page#Examples) has some good examples. There is also an entire firewall built with nftables from the openwrt project, [fw4](https://git.openwrt.org/?p=project/firewall4.git;a=blob;f=root/etc/nftables.d/10-custom-filter-chains.nft;hb=HEAD), but it is configured via `uci` which can be seen [here](https://git.openwrt.org/?p=project/firewall4.git;a=blob;f=root/etc/config/firewall;hb=HEAD) as well as posts all over the openwrt wiki.

