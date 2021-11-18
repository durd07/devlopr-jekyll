---
title: Linux
layout: page
search: true
---

# Linux

## Debian

### debian build deb
```
mkdir tmp
dpkg-deb -R original.deb tmp
# edit DEBIAN/postinst
dpkg-deb -b tmp fixed.deb
```

## Ubuntu
### snapd set proxy
```
$ sudo snap set system proxy.http="http://<proxy_addr>:<proxy_port>"
$ sudo snap set system proxy.https="http://<proxy_addr>:<proxy_port>"
```

## MISC
### How to see the order of shared library loading
```
LD_DEBUG=files ./a.out
```

## Start VNC
```
sudo /usr/sbin/vncsession felixdu :2
```

# openSUSE

## proxy

```
/etc/sysconfig/proxy
```

# Fedora

```sh
dnf provides "su"
```

