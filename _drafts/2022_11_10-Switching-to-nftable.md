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
Short Answer: cause I want to.
Its not better or worse than firewalld just different in my opinon. I like to know how things work under the covers and I like working with networking. I also like knowing that I am using all the advancements, instead of waiting for it to owork its way into firewalld.
## Uninstalling firewalld
Pretty simple, `dnf remove firewalld`. So far on my laptop that always lives on wifi at home I haven't had too many issues with another service or process needing the functionality that firewalld provides.
## Installing nftables
It is already installed if you are using firewalld. If not, `dnf install -y nftables`. Take a look at the files included in the rpm with `rpm -ql nftables`. Your new firewall config file lives in `/etc/nftables/`. The package also includes a `systemd` unit file as well. Take a look at it with `systemctl edit nftables.service`.
# Setting up rules

