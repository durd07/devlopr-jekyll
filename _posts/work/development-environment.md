---
title: Development Environment
---
# Development Environment

## CNCS Build Server

### Server Information

| Key      | Value                          |
|----------|--------------------------------|
| HostName | cncs-build.dynamic.nsn-net.net |
| OS       | Ubuntu 20.04.1                 |
| UserName | your nsn-intra login           |
| Password | your nsn-intra password        |

**Notes**:

1. all users have sudoer permission, you can do sudo command like install packages, but avoid do anything that is harmful.
2. all users have been added into docker group, you can run docker commands without sudo.
3. the default shell has been changed to zsh, which is more powerful than bash.
4. some tiny configuration copys from what I am using, like vim config, zsh config, tmux config, you can change them as you like or ask me for more complete configurations.
5. http server is enabled, you can create your own page at `$HOME/public_html` and visit them by http://cncs-build.dynamic.nsn-net.net/~<username> like http://cncs-build.dynamic.nsn-net.net/~felixd.
6. there are some packages that already installed, if you don't know what they are, just ignore them.
	- git
	- vim
	- fzf
	- ripgrep
	- global (to replace cscope)
	- universal-ctags


### Procedures of how to build this server
#### Setup domain name for your host
```bash
yum -y install bind-utils ## install nsupdate/dig
[felixdu@Fedora ~]# nsupdate
> update delete  felixdu.hz.dynamic.nsn-net.net
> update add felixdu.hz.dynamic.nsn-net.net 3600 A 10.182.70.198 ## floating ip
> send
> quit
```

#### Configure LDAP Client in order to share user accounts in your local networks
[https://www.server-world.info/en/note?os=Ubuntu_20.04&p=openldap&f=3](https://www.server-world.info/en/note?os=Ubuntu_20.04&p=openldap&f=3)

> don't create root from ldap

#### Ubuntu 20.04 nginx user directory
[https://www.server-world.info/en/note?os=Ubuntu_20.04&p=nginx&f=4](https://www.server-world.info/en/note?os=Ubuntu_20.04&p=nginx&f=4)


> All the configuration are stored at cncs-build.dynamic.nsn-net.net:/home/felixd/ldap_server_config  
> vim/zsh configuration [https://github.com/durd07/linux_config.git](https://github.com/durd07/linux_config.git)


## Development Container Image

```bash
docker run -it durd07/dev zsh
docker run -it felixdu.hz.dynamic.nsn-net.net/dev zsh

docker run --name dev -td --network host -p 222:22 -p 5902:5902 -p 8888:8888 -v /dev/shm:/dev/shm -v /root/data:/home/felixdu/data felixdu.hz.dynamic.nsn-net.net/dev:fedora-34
```

## Development with Vscode and Docker Desktop

refer to : [https://confluence.ext.net.nokia.com/display/~ryliu/NTAS+Development%2C+Compile+and+Bringup+in+Windows+Environment+Using+VSCode](https://confluence.ext.net.nokia.com/display/~ryliu/NTAS+Development%2C+Compile+and+Bringup+in+Windows+Environment+Using+VSCode)

```bash
docker run -td --name ntas-build --privileged -v ntas-root-volume:/root/volume -v //var/run/docker.sock:/var/run/docker.sock  ntas-docker-releases.repo.lab.pl.alcatel-lucent.com/ntas/build:2.7.0 bash
```


# Fedora
----------------------------------------------------------------------------------------------------
## install softwares
dnf -y install vim zsh git tmux java htop npm universal-ctags global

## create user
useradd -d /home/felixdu -c "Felix Du" felixdu

## add felixdu to sudoers
vi /etc/sudoers and add `felixdu ALL=(ALL)       ALL`

## switcher user
su - felixdu

## use zsh as default shell
sudo dnf -y install util-linux-user # to install chsh
sudo chsh -s $(which zsh)

## install ohmyzsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

## install docker
https://docs.docker.com/install/linux/docker-ce/fedora/

## enable docker for felixdu
https://docs.docker.com/install/linux/linux-postinstall/

## enable docker for Fedora 31+
https://medium.com/nttlabs/cgroup-v2-596d035be4d7

## Vim Configuration
### Install essential packages

- universal-ctags
  ```sh
  sudo dnf install kernel-modules  # add kernel module support for squashfs
  sudo dnf install snapd
  sudo ln -s /var/lib/snapd/snap /snap
  sudo snap install universal-ctags
  ```

