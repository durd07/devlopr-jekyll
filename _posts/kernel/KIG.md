---
title: KIG
---

# Linux Kernel Study

[Kernel Interest Group Homepage](https://confluence.int.net.nokia.com/display/~ryliu/KIG+--+Kernel+Interest+Group+Homepage)

## Structure of the Linux Kernel
[Linux Kernel Map](http://www.makelinux.net/kernel_map/)  

![structure](/uploads/images/linux-structure.png)


## Books

- [Linux Kernel Networking: Implementation and Theory](https://learning.oreilly.com/library/view/linux-kernel-networking/9781430261964/9781430261964_Ch04.xhtml)
- [Linux-insides](https://0xax.gitbooks.io/linux-insides/)
- [Professional Linux Kernel Architecture](https://learning.oreilly.com/library/view/professional-linux-kernel/9780470343432/)

- [x86 Assembly Guide](http://flint.cs.yale.edu/cs421/papers/x86-asm/asm.html)
- [x64 Cheat Sheet](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf)


## Build Kernel
> **OpenSUSE make config necessary packages:**
> for opensuse build with make gconfig should install `libglade2-devel`, for make xconfig, should install `libglade2-devel` or `libqt5-qtbase-devel`

> sudo zypper install p7zip-full 
> 7z e xx.zip

### Fedora
```bash
sudo dnf install flex bison elfutils-libelf-devel ncurses-devel openssl-devel
```

## UML
[HomePage](http://user-mode-linux.sourceforge.net/)
[Book: User Mode Linux](/uploads/pdf/User-Mode-Linux.pdf)
https://christine.website/blog/howto-usermode-linux-2019-07-07

### build UML
```
cd linux-kernel-dir
mkdir build
make ARCH=um O=build defconfig
make ARCH=um O=build linux -j12

# create rootfs, I tried 2 methods to creat rootfs, unpack the rootfs from alpine and use debootstrap, following is the procedure of create rootfs from alpine rootfs
dd if=/dev/zero of=root_fs seek=500 count=1 bs=1M
mkfs.ext4 ./root_fs
mkdir /mnt/rootfs
mount -o loop root_fs /mnt/rootfs/
wget http://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86_64/alpine-minirootfs-3.10.2-x86_64.tar.gz
cp alpine-minirootfs-3.10.2-x86_64.tar.gz /mnt/rootfs
cd /mnt/rootfs && tar –xf alpine-minirootfs-3.10.2-x86_64.tar.gz && cd -
# you can change the root password by midifying the etc/password and etc/shadow
umount /mnt/rootfs/

#Use debootstrap to create rootfs
felixdu:/data/linux# cat create_image.sh
IMG=qemu-image.img
DIR=mount-point.dir
qemu-img create $IMG 1g
mkfs.ext4 $IMG
mkdir $DIR
sudo mount -o loop $IMG $DIR
#sudo debootstrap --arch amd64 jessie $DIR
sudo debootstrap --arch amd64 buster $DIR

#sudo chroot $DIR
#passwd

root@felixdu:/etc/network/interfaces.d# cat enp0s3
auto enp0s3
iface enp0s3 inet dhcp

sudo umount $DIR
rmdir $DIR
# this used to use the image for qemu
#qemu-img convert -O qcow2 qemu-img.img qemu-img.qcow2
```

Run the UML with command:
```
./build/linux ubda=./root_fs rw mem=512m init=/bin/sh or ./build/linux ubda=./ qemu-img.img rw mem=512m
```

After the kernel booting and mounting the rootfs, it will print
```
Virtual console 5 assigned device '/dev/pts/7'
Virtual console 3 assigned device '/dev/pts/12'
Virtual console 4 assigned device '/dev/pts/13'
Virtual console 6 assigned device '/dev/pts/14'
Virtual console 1 assigned device '/dev/pts/15'
Virtual console 2 assigned device '/dev/pts/16'
```
Then login to the linux by `screen /dev/pts/7`

https://buildroot.org/

Debug UML with debug
`gdb --args build/linux ubda=./root_fs rw mem=512m`

### start the linux simulator with qemu
```
qemu-system-x86_64 -kernel /data/linux/build/arch/x86_64/boot/bzImage -hda ./qemu-image.img -nographic -append "earlyprintk=serial,ttyS0 console=ttyS0 root=/dev/sda rw kgdboc=ttyS1,115200 nokaslr" -m 1024m -s -S
mount -o remount,rw /

gdb /data/linux/build/vmlinux
target remote :1234
b ip_rcv
c
```
http://nickdesaulniers.github.io/blog/2018/10/24/booting-a-custom-linux-kernel-in-qemu-and-debugging-it-with-gdb/
https://01.org/linuxgraphics/gfx-docs/drm/dev-tools/gdb-kernel-debugging.html


## Generate linux kernel api doc
```sh
make mandocs
make installmandocs
```
```
zhaozc@zhaozc-VirtualBox:~$ man printk  
PRINTK(9)                                                 Driver Basics                                                PRINTK(9)

NAME
       printk - print a kernel message

SYNOPSIS
       int printk(const char * fmt, ...);

ARGUMENTS
       fmt
           format string

       ...
           variable arguments

DESCRIPTION
       This is printk. It can be called from any context. We want it to work.

       We try to grab the console_lock. If we succeed, it's easy - we log the output and call the console drivers. If we fail to
       get the semaphore, we place the output into the log buffer and return. The current holder of the console_sem will notice
       the new output in console_unlock; and will send it to the consoles before releasing the lock.

       One effect of this deferred printing is that code which calls printk and then changes console_loglevel may break. This is
       because console_loglevel is inspected when the actual printing occurs.

SEE ALSO
       printf(3)

       See the vsnprintf documentation for format string extensions over C99.

COPYRIGHT
```

## Tools

### Iproute2
The iproute2 package includes commands like the following:
ip: For management of network tables and network interfaces
tc: For traffic control management
ss: For dumping socket statistics
lnstat: For dumping linux network statistics
bridge: For management of bridge addresses and devices

### slabtop /proc/slabinfo

### SystemTap

####cross-instrumentation
Cross-instrumentation is the process of generating SystemTap instrumentation modules from a SystemTap script on one computer to be used on another computer. This process offers the following benefits:
- The kernel information packages for various machines can be installed on a single host machine.
- Each target machine only needs one package to be installed to use the generated SystemTap instrumentation module: systemtap-runtime.
Build Instrumentation Module on host system
```
stap -r kernel_version script -m module_name -p4
stap -r /data/linux/build /data/test/linux/stap_vfs -m stap_test.ko -p4
cat /data/test/linux/stap_vfs
probe vfs.read {printf("read performed\n"); exit()}
```

Load Instrumentation Module on target system
```
staprun module_name.ko
```

How systemap works
https://my.oschina.net/sherrywangzh/blog/1518223
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html-single/systemtap_beginners_guide/index

> Note:
> There will be compile error like this:
> ```
> $ stap -r /data/linux/build /data/test/linux/stap_vfs -m stap_mod.ko -p4
> Truncating module name to 'stap_mod'
> In file included from /usr/share/systemtap/runtime/linux/print.c:18,
>                  from /usr/share/systemtap/runtime/print.c:17,
>                  from /usr/share/systemtap/runtime/runtime_context.h:22,
>                  from /tmp/stap7KpE9c/stap_mod_src.c:115:
> /usr/share/systemtap/runtime/vsprintf.c: In function ‘_stp_vsnprintf’:
> /usr/share/systemtap/runtime/vsprintf.c:641:35: error: this statement may fall through [-Werror=implicit-fallthrough=]
>   641                               flags |= STP_LARGE;
>                                     ~~~~~~^~~~~~~~~~~~
> /usr/share/systemtap/runtime/vsprintf.c:642:21: note: here
>   642                       case 'x':
>                             ^~~~
> /usr/share/systemtap/runtime/vsprintf.c:828:10: error: this statement may fall through [-Werror=implicit-fallthrough=]
>   828      flags |= STP_LARGE;
>            ~~~~~~^~~~~~~~~~~~
> /usr/share/systemtap/runtime/vsprintf.c:829:3: note: here
>   829     case 'x':
>           ^~~~
> cc1: all warnings being treated as errors
> make[2]: *** [../scripts/Makefile.build:280: /tmp/stap7KpE9c/stap_mod_src.o] Error 1
> make[1]: *** [/data/linux/Makefile:1624: _module_/tmp/stap7KpE9c] Error 2
> make: *** [/data/linux/Makefile:179: sub-make] Error 2
> ```
> Fix this by adding `__attribute__ ((fallthrough));` after the error line.
> Detail about `implicit-fallthrough` please refer to https://developers.redhat.com/blog/2017/03/10/wimplicit-fallthrough-in-gcc-7/


## Knowledge

Implementation of the layer model in the kernel  

![network_implementaion_of_the_layer_model_in_kernel](/uploads/images/network_implementaion_of_the_layer_model_in_kernel.png)

**Network Namespace**

As usual, a central structure is used to keep track of all available namespaces. The definition is as follows:
```
> include/net/net_namespace.h

struct net {
        atomic_t count; /* To decided when the network
        * namespace should be freed.
        * get_net/put_net if count = 0, free this net
        */
        ...
        struct list_head list; /* list of network namespaces
                                * header is net_namespace_list
                                */
        ...
        struct proc_dir_entry *proc_net;
        struct proc_dir_entry *proc_net_stat;
        struct proc_dir_entry *proc_net_root;
        struct net_device *loopback_dev; /* The loopback */
        struct list_head dev_base_head;
        struct hlist_head *dev_name_head;
        struct hlist_head *dev_index_head;
};
```

Most computers will typically require only a single networking namespace. The global variable `init_net`
(and in this case, the variable is really global and not contained in another namespace!) contains the net
instance for this namespace.

**Socket Buffer**
```
<skbuff.h>
struct sk_buff {
        /* These two members must be first. */
        struct sk_buff *next;
        struct sk_buff *prev;
        struct sock *sk;
        ktime_t tstamp;
        struct net_device *dev;
        struct dst_entry *dst;
        char cb[48];
        unsigned int len,
        data_len;
        __u16 mac_len,
        hdr_len;
        ...
};
```
> next/prev have been replaced by an union which support standard list
>```
> union {
>     struct {
>         /* These two members must be first. */
>         struct sk_buff      *next;
>         struct sk_buff      *prev;
>         union {
>             struct net_device   *dev;
>             /* Some protocols might use this space to store information,
>              * while device pointer would be NULL.
>              * UDP receive path is one user.
>              */
>             unsigned long       dev_scratch;
>         };
>     };
>     struct rb_node      rbnode; /* used in netem, ip4 defrag, and tcp stack */
>     struct list_head    list;
> };
>```

A list head is used to implement wait queues with socket buffers. Its structure is defined as follows:
```
<skbuff.h>
struct sk_buff_head {
				/* These two members must be first. */
				struct sk_buff *next;
				struct sk_buff *prev;
				__u32 qlen;
				spinlock_t lock;
};
```
qlen specifies the length of the wait queue; that is, the number of elements in the queue. next and prev
of sk_buff_head and sk_buff are used to create a cyclic doubly linked list, and the list element of the
socket buffer points back to the list head,

**Network Access Layer**
net_device <netdevice.h>
net/core/dev.c register_netdev to register net device to kernel
create sysfs /sys/class/net/<device>

❑ open and stop initialize and terminate network cards. These actions are usually triggered from
outside the kernel by calling the ifconfig command. open is responsible for initializing the
hardware registers and registering system resources such as interrupts, DMA, IO ports, and so
on. close releases these resources and stops transmission.
❑ hard_start_xmit is called to remove finished packets from the wait queue and send them.
❑ header_ops contains a pointer to a structure that provides more function pointers to operations
on the hardware header.
Most important are header_ops->create, which creates a new, and header_ops->parse, to analyze
a given hardware header.
❑ get_stats queries statistical data that are returned in a structure of type net_device_stats.
This structure consists of more than 20 members, all of which are numeric values to indicate,
for example, the number of packets sent, received, with errors, discarded, and so on. (Lovers of
statistics can query these data using ifconfig and netstat -i.)
Because the net_device structure provides no specific field to store the net_device_stats
object, the individual device drivers must keep it in their private data area.
❑ tx_timeout is called to resolve the problem of packet transmission failure.
❑ do_ioctl forwards device-specific commands to the network card.
❑ nd_net is a pointer to the networking namespace (represented by an instance of struct net) to
which the device belongs.

> there is a typo in professional linux kernel architecture nd_det -> nd_net

Network devices work in two directions — they send and they receive (these directions are often referred
to as downstream and upstream). The kernel sources include two driver skeletons (`isa-skeleton.c` and
`pci-skeleton.c` in `drivers/net`) for use as network driver templates

**Registering Network Devices**
alloc_netdev
register_netdev/register_netdevice
alloc_etherdev(sizeof_priv)/ether_setup

**NAPI**

>What happens if more than one device is present on the system? This is accounted for by a round robin
>method employed to poll the devices

The key change in contrast to the old API is that a network device that supports NAPI must provide a
poll function. The device-specific method is specified when the network card is registered with
netif_napi_add. Calling this function also indicates that the devices can and must be handled with the
new methods.
```
<netdevice.h>
static inline void netif_napi_add(struct net_device *dev,
                                  struct napi_struct *napi,
                                  int (*poll)(struct napi_struct *, int),
                                  int weight);
```
`dev` points to the `net_device` instance for the device in question, `poll` specifies which function is used
to poll the device with IRQs disabled, and `weight` does what you expect it to do: It specifies a relative
weight for the interface. In principle, an arbitrary integer value can be specified. Usually 10- and 100-MBit
drivers specify 16, while 1,000- and 10,000-MBit drivers use 64. In any case, the weight must not exceed
the number of packets that can be stored by the device in the Rx buffer.



```
//struct sock - network layer representation of sockets
struct sock {
```






**IP header:**
Definition include/uapi/linux/ip.h struct iphdr 
RFC https://tools.ietf.org/html/rfc791 

![ip_rev](https://learning.oreilly.com/library/view/linux-kernel-networking/9781430261964/images/9781430261964_Fig04-02.jpg)  
```c
int ip_rcv(struct sk_buff *skb, struct net_device *dev, struct packet_type *pt,
	   struct net_device *orig_dev)
     |
  skb = ip_rcv_core(skb, net);
    if promisc --> drop
	  skb = skb_share_check(skb, GFP_ATOMIC); // check if buffer is shared and if so clone it
    /*
      pskb_may_pull确保skb->data指向的内存包含的数据至少为IP头部大小，由于每个IP数据包包括IP分片必须包含一个完整的IP头部。如果小于IP头部大小，则缺失的部分将从数据分片中拷贝。这些分片保存在skb_shinfo(skb)->frags[]中
    */
	  if (!pskb_may_pull(skb, sizeof(struct iphdr)))
    get ip_hdr check ip_hdr and set transport_header

	return NF_HOOK(NFPROTO_IPV4, NF_INET_PRE_ROUTING,
		       net, NULL, skb, dev, NULL,
		       ip_rcv_finish);

					    enum nf_inet_hooks {
 						          NF_INET_PRE_ROUTING,
 						          NF_INET_LOCAL_IN,
 						          NF_INET_FORWARD,
 						          NF_INET_LOCAL_OUT,
 						          NF_INET_POST_ROUTING,
 						          NF_INET_NUMHOOKS
 						  };

							static inline int
							NF_HOOK(uint8_t pf, unsigned int hook, struct net *net, struct sock *sk, struct sk_buff *skb,
								struct net_device *in, struct net_device *out,
								int (*okfn)(struct net *, struct sock *, struct sk_buff *))
							{
								int ret = nf_hook(pf, hook, net, sk, skb, in, out, okfn);
								if (ret == 1)
									ret = okfn(net, sk, skb);
								return ret;
							}

    static int ip_rcv_finish(struct net *net, struct sock *sk, struct sk_buff *skb)
      skb = l3mdev_ip_rcv(skb); // https://netdevconf.info/1.2/papers/ahern-what-is-l3mdev-paper.pdf VRF(Virtual Routing Forwarding)
      ret = ip_rcv_finish_core(net, sk, skb, dev); 
        if (net->ipv4.sysctl_ip_early_demux &&
            !skb_dst(skb) &&
            !skb->sk &&
            !ip_is_fragment(iph)) {
              /* use tcp/udp early demux, it will select established tcp socket and set skb's dst to socket's dst for local socket, it will benefit, but for forward packages, it will waste time to search local socket */
              int protocol = iph->protocol;

              ipprot = rcu_dereference(inet_protos[protocol]);
              if (ipprot && (edemux = READ_ONCE(ipprot->early_demux))) {
                err = INDIRECT_CALL_2(edemux, tcp_v4_early_demux,
                          udp_v4_early_demux, skb);

        /* if invalud dst, search the routeing table and set skb->dst */
        if (!skb_valid_dst(skb)) {
          err = ip_route_input_noref(skb, iph->daddr, iph->saddr,
                   iph->tos, dev);
	      rt = skb_rtable(skb);
	    if (ret != NET_RX_DROP)
		  ret = dst_input(skb);  --> skb_dst(skb)->input(skb) // for unicast the input is ip_local_deliver
```

```c
/*
 * 	Deliver IP Packets to the higher protocol layers.
 */
int ip_local_deliver(struct sk_buff *skb)
	if (ip_is_fragment(ip_hdr(skb))) {
    /*
    return (iph->frag_off & htons(IP_MF | IP_OFFSET)) != 0;
    frag_off 16bits, the first 3 bits(MF) is flag, last 13bits(OFFSET) are offset, if not fragment, both MF and OFFSET equal to 0, if the last fragemtn(MF == 0, OFFSET != 0), if it is a fragemnt but not lat one, call ip_defrag, if not fragment or last fragment, goto ip_local_deliver_finish.
    */
		if (ip_defrag(net, skb, IP_DEFRAG_LOCAL_DELIVER))
			return 0;
	}

	return NF_HOOK(NFPROTO_IPV4, NF_INET_LOCAL_IN,
		       net, NULL, skb, skb->dev, NULL,
		       ip_local_deliver_finish);

           tcp_v4_rcv/udp_rcv

static int ip_local_deliver_finish(struct net *net, struct sock *sk, struct sk_buff *skb)
  ip_protocol_deliver_rcu
	  raw = raw_local_deliver(skb, protocol);
      // 所有的raw socket都挂在全局数组raw_v4_hashinfo上每种协议都有一个列表
      raw_v4_hashinfo
      raw_v4_input(skb, ip_hdr(skb), hash)
        int raw_rcv(struct sock *sk, struct sk_buff *skb)
          static int raw_rcv_skb(struct sock *sk, struct sk_buff *skb)
	          sock_queue_rcv_skb(sk, skb)
		          sk->sk_data_ready(sk);
                                           void sock_init_data(struct socket *sock, struct sock *sk)
	                                           sk->sk_data_ready	=	sock_def_readable;

	  ipprot = rcu_dereference(inet_protos[protocol]);
		  ret = INDIRECT_CALL_2(ipprot->handler, tcp_v4_rcv, udp_rcv, skb);
```

### Function backtrace of ip_rcv
```
#0  ip_rcv (skb=0xffff88803d60cc00, dev=0xffff88803e556000, pt=0xffffffff8254ada0 <ip_packet_type>,
    orig_dev=0xffff88803e556000) at ../net/ipv4/ip_input.c:516
#1  0xffffffff818188eb in __netif_receive_skb_one_core (skb=0xffff88803d60cc00,
    pfmemalloc=<optimized out>) at ../net/core/dev.c:5004
#2  0xffffffff8181898a in netif_receive_skb_internal (skb=0xffff88803d60cc00) at ../net/core/dev.c:5208
#3  0xffffffff818197d1 in napi_skb_finish (skb=<optimized out>, ret=GRO_NORMAL)
    at ../net/core/dev.c:5671
#4  napi_gro_receive (napi=0xffff88803e556af0, skb=0xffff88803d60cc00) at ../net/core/dev.c:5704
#5  0xffffffff8167219b in e1000_receive_skb (skb=<optimized out>, vlan=<optimized out>,
    status=<optimized out>, adapter=<optimized out>)
    at ../drivers/net/ethernet/intel/e1000/e1000_main.c:3999
#6  e1000_clean_rx_irq (adapter=0xffff88803e556840, rx_ring=<optimized out>, work_done=<optimized out>,
    work_to_do=<optimized out>) at ../drivers/net/ethernet/intel/e1000/e1000_main.c:4455
#7  0xffffffff81672a23 in e1000_clean (napi=0xffff88803e556af0, budget=64)
    at ../drivers/net/ethernet/intel/e1000/e1000_main.c:3799
#8  0xffffffff81818e6c in napi_poll (repoll=<optimized out>, n=<optimized out>)
    at ../net/core/dev.c:6352
#9  net_rx_action (h=<optimized out>) at ../net/core/dev.c:6418
#10 0xffffffff81e000db in __do_softirq () at ../kernel/softirq.c:292
#11 0xffffffff8106a395 in run_ksoftirqd (cpu=<optimized out>) at ../kernel/softirq.c:603
#12 run_ksoftirqd (cpu=<optimized out>) at ../kernel/softirq.c:595
#13 0xffffffff81089780 in smpboot_thread_fn (data=0xffff88803e4021c0) at ../kernel/smpboot.c:165
#14 0xffffffff81085ef6 in kthread (_create=0xffff88803e43f600) at ../kernel/kthread.c:255
#15 0xffffffff81c00205 in ret_from_fork () at ../arch/x86/entry/entry_64.S:352
```
```
subsys_initcall(net_dev_init)
	static int __init net_dev_init(void)
		open_softirq(NET_TX_SOFTIRQ, net_tx_action); // register the callback for softirq
		open_softirq(NET_RX_SOFTIRQ, net_rx_action); // register the callback for softirq
```

net_rx_action

It must be made sure that not too much time is spent in the softIRQ handler. Processing is aborted on
two conditions:
1. More than one jiffie has been spent in the handler.
2. The total number of processed packets is larger than a total budget specified
   by netdev_budget. Usually, this is set to 300, but the value can be changed via
   /proc/sys/net/core/netdev_budget.
   This budget must not be confused with the local budget for each network device! After each
   poll step, the number of processed packets is subtracted from the global budget, and if the
   value drops below zero, the softIRQ handler is aborted.
```

static int __init inet_init(void)
```

![ip_send](https://learning.oreilly.com/library/view/linux-kernel-networking/9781430261964/images/9781430261964_Fig04-08.jpg)
```
const struct inet_connection_sock_af_ops ipv4_specific = {
	.queue_xmit	   = ip_queue_xmit,
	.send_check	   = tcp_v4_send_check,
	.rebuild_header	   = inet_sk_rebuild_header,
	.sk_rx_dst_set	   = inet_sk_rx_dst_set,
	.conn_request	   = tcp_v4_conn_request,
	.syn_recv_sock	   = tcp_v4_syn_recv_sock,
	.net_header_len	   = sizeof(struct iphdr),
	.setsockopt	   = ip_setsockopt,
	.getsockopt	   = ip_getsockopt,
	.addr2sockaddr	   = inet_csk_addr2sockaddr,
	.sockaddr_len	   = sizeof(struct sockaddr_in),
#ifdef CONFIG_COMPAT
	.compat_setsockopt = compat_ip_setsockopt,
	.compat_getsockopt = compat_ip_getsockopt,
#endif
	.mtu_reduced	   = tcp_v4_mtu_reduced,
};

static int tcp_v4_init_sock(struct sock *sk)
	tcp_sk(sk)->af_specific = &tcp_sock_ipv4_specific;
```

dev_queue_xmit from net/core/dev.c is used to place the packet on the queue for outgoing packets.

```c
int fib_lookup(struct net *net, const struct flowi4 *flp, struct fib_result *res)
                                                      |                        |
                                                      |                        | -> struct fib_result {
                                                      |                                 unsigned char    prefixlen;        // netmask 0 if using default route check_leaf() set it
                                                      |                                 unsigned char    nh_sel;           // nexthop number, 0 -> one nexthop only
                                                      |                                 unsigned char    type;             // RTN_UNICAST -> forward via a gw or direct route / RTN_LOCA local 
                                                      |                                 unsigned char    scope;        
                                                      |                                 u32              tclassid;        
                                                      |                                 struct fib_info  *fi;        
                                                      |                                 struct fib_table *table;        
                                                      |                                 struct list_head *fa_head;
                                                      |                             };
                                                      | -> destination address
                                                      | -> source address
                                                      | -> TOS

struct dst_entry {
    ...    
    int  (*input)(struct sk_buff *);    
    int  (*output)(struct sk_buff *);    
    ...
}

struct rtable {    
    struct dst_entry  dst;     
    int               rt_genid;    
    unsigned int      rt_flags;    
    __u16             rt_type;    
    __u8              rt_is_input;    
    __u8              rt_uses_gateway;     
    int               rt_iif;      
    /* Info on neighbour */   
    __be32            rt_gateway;     
    /* Miscellaneous cached information */    
    u32               rt_pmtu;     
    struct list_head  rt_uncached;
};

skb->dst = rtable

  local delivery (RTN_LOCAL),
  forwarding to a supplied next hop (RTN_UNICAST),
  silent discard (RTN_BLACKHOLE).
```

FIB Tables
created by `fib_create_info`
stored in `fib_info_hash`

added into `fib_info_laddrhash` if the route uses prefsrc

`fib_info_cnt` `fib_create_info` `free_fib_info` `fib_find_info`

```
struct fib_table {
        struct hlist_node
        tb_hlist;
        u32                     tb_id;           // RT_TABLE_MAIN 254/RT_TABLE_LOCAL 255 **when working without Policy Routing, only main table and local table created in boot.**
        int                     tb_default;      //
        int                     tb_num_default;  // The number of the default routes in the table
        unsigned long           tb_data[0];      // A placeholder for a routing entry (trie) object
};

struct fib_info {
    struct hlist_node    fib_hash;
    struct hlist_node    fib_lhash;
    struct net        *fib_net; // namespace
    int               fib_treeref; // A reference counter that represents the number of fib_alias objects which hold a reference to this fib_info object
    atomic_t          fib_clntref;
    unsigned int      fib_flags;
    unsigned char     fib_dead;
    unsigned char     fib_protocol; // ip route add proto static 192.168.5.3 via 192.168.2.1 RTPROT_UNSPEC/RTPROT_REDIRECT/RTPROT_KERNEL/RTPROT_BOOT/RTPROT_STATIC/
    unsigned char     fib_scope; // ip address show/ip route show  -- scope host  RT_SCOPE_HOST The node cannot communicate with the other network nodes
                                                                      scope global  RT_SCOPE_UNIVERSE The address can be used anywhere
                                                                      scope link  RT_SCOPE_LINK This address can be accessed only from directly attached hosts.
                                                                      scope site  RT_SCOPE_SITE ipv6
                                                                      scope nowhere  RT_SCOPE_NOWHERE Destination doesn't exist.
    unsigned char     fib_type;
    __be32            fib_prefsrc;
    u32               fib_priority; // The priority of the route, by default, is 0, which is the highest priority. The higher the value of the priority, the lower the priority is.
    u32               *fib_metrics;
#define fib_mtu fib_metrics[RTAX_MTU-1]
#define fib_window fib_metrics[RTAX_WINDOW-1]
#define fib_rtt fib_metrics[RTAX_RTT-1]
#define fib_advmss fib_metrics[RTAX_ADVMSS-1]    
    int               fib_nhs;
#ifdef CONFIG_IP_ROUTE_MULTIPATH    
    int               fib_power;
#endif    struct rcu_head   rcu;
    struct fib_nh     fib_nh[0];
#define fib_dev       fib_nh[0].nh_dev
};
```

Since 2.6.39, Linux stores routes in a compressed prefix tree

A lookup in the routing subsystem is done for each packet, both in the Rx path and in the Tx path





### ip send package
#### Trace Back
```
#0  __dev_queue_xmit (skb=0x0 <fixed_percpu_data>, sb_dev=0x0 <fixed_percpu_data>) at ../net/core/dev.c:3897
#1  0xffffffff81890b4d in neigh_output (skip_cache=<optimized out>, skb=<optimized out>, n=<optimized out>) at ../include/net/neighbour.h:511
#2  ip_finish_output2 (net=<optimized out>, sk=<optimized out>, skb=0xffff88803d5e7900) at ../net/ipv4/ip_output.c:228
#3  0xffffffff818927bc in ip_finish_output (skb=<optimized out>, sk=<optimized out>, net=<optimized out>) at ../net/ipv4/ip_output.c:318
#4  NF_HOOK_COND (pf=<optimized out>, hook=<optimized out>, in=<optimized out>, okfn=<optimized out>, cond=<optimized out>, out=<optimized out>,
    skb=<optimized out>, sk=<optimized out>, net=<optimized out>) at ../include/linux/netfilter.h:294
#5  ip_output (net=0xffffffff824ef880 <init_net>, sk=0xffff88803b3c1dc0, skb=0xffff88803d5e7900) at ../net/ipv4/ip_output.c:432
#6  0xffffffff818930b0 in ip_send_skb (net=0xffffffff824ef880 <init_net>, skb=<optimized out>) at ../net/ipv4/ip_output.c:1554
#7  0xffffffff818befe3 in udp_send_skb (skb=0xffff88803d5e7900, fl4=0xffff88803b3c2110, cork=<optimized out>, cork=<optimized out>)
    at ../include/net/net_namespace.h:311
#8  0xffffffff818bfbe4 in udp_sendmsg (sk=0xffff88803b3c1dc0, msg=<optimized out>, len=<optimized out>) at ../net/ipv4/udp.c:1174
#9  0xffffffff817f6002 in sock_sendmsg_nosec (msg=<optimized out>, sock=<optimized out>) at ../include/linux/uio.h:235
#10 sock_sendmsg (sock=0xffff88803be18000, msg=0xffffc900002abe18) at ../net/socket.c:657
#11 0xffffffff817f63d6 in ___sys_sendmsg (sock=0xffff88803be18000, msg=<optimized out>, msg_sys=0xffffc900002abe18, flags=<optimized out>,
    used_address=0xffffc900002abe70, allowed_msghdr_flags=<optimized out>) at ../net/socket.c:2311
#12 0xffffffff817f7c40 in __sys_sendmmsg (fd=<optimized out>, mmsg=0x7f6605861d40, vlen=<optimized out>, flags=<optimized out>,
    forbid_cmsg_compat=<optimized out>) at ../net/socket.c:2413
#13 0xffffffff817f7d6b in __do_sys_sendmmsg (flags=<optimized out>, vlen=<optimized out>, mmsg=<optimized out>, fd=<optimized out>) at ../net/socket.c:2442
#14 __se_sys_sendmmsg (flags=<optimized out>, vlen=<optimized out>, mmsg=<optimized out>, fd=<optimized out>) at ../net/socket.c:2439
#15 __x64_sys_sendmmsg (regs=<optimized out>) at ../net/socket.c:2439
#16 0xffffffff81002573 in do_syscall_64 (nr=<optimized out>, regs=0xffffc900002abf58) at ../arch/x86/entry/common.c:296
#17 0xffffffff81c0007c in entry_SYSCALL_64 () at ../arch/x86/entry/entry_64.S:175





#0  dev_hard_start_xmit (first=0xffff88803d5e7e00, dev=0xffff88803e556000, txq=0xffff88803db03a00, ret=0xffffc9000029fac4) at ../net/core/dev.c:3292
#1  0xffffffff81852d61 in sch_direct_xmit (skb=0xffff88803d5e7e00, q=0xffff88803d637400, dev=0xffff88803e556000, txq=0xffff88803db03a00,
    root_lock=0x0 <fixed_percpu_data>, validate=<optimized out>) at ../net/sched/sch_generic.c:308
#2  0xffffffff81817427 in __dev_xmit_skb (txq=<optimized out>, dev=<optimized out>, q=<optimized out>, skb=<optimized out>) at ../net/core/dev.c:3477
#3  __dev_queue_xmit (skb=0xffff88803d5e7e00, sb_dev=0x0 <fixed_percpu_data>) at ../net/core/dev.c:3838
#4  0xffffffff81817577 in dev_queue_xmit (skb=<optimized out>) at ../net/core/dev.c:3902
#5  0xffffffff81890c6b in neigh_hh_output (skb=<optimized out>, hh=<optimized out>) at ../include/net/neighbour.h:500
#6  neigh_output (skip_cache=<optimized out>, skb=<optimized out>, n=<optimized out>) at ../include/net/neighbour.h:509
#7  ip_finish_output2 (net=<optimized out>, sk=<optimized out>, skb=0xffff88803d5e7e00) at ../net/ipv4/ip_output.c:228
#8  0xffffffff818927bc in ip_finish_output (skb=<optimized out>, sk=<optimized out>, net=<optimized out>) at ../net/ipv4/ip_output.c:318
#9  NF_HOOK_COND (pf=<optimized out>, hook=<optimized out>, in=<optimized out>, okfn=<optimized out>, cond=<optimized out>, out=<optimized out>,
    skb=<optimized out>, sk=<optimized out>, net=<optimized out>) at ../include/linux/netfilter.h:294
#10 ip_output (net=0xffffffff824ef880 <init_net>, sk=0xffff88803b3c1dc0, skb=0xffff88803d5e7e00) at ../net/ipv4/ip_output.c:432
#11 0xffffffff818930b0 in ip_send_skb (net=0xffffffff824ef880 <init_net>, skb=<optimized out>) at ../net/ipv4/ip_output.c:1554
#12 0xffffffff818befe3 in udp_send_skb (skb=0xffff88803d5e7e00, fl4=0xffffc9000029fd28, cork=<optimized out>, cork=<optimized out>)
    at ../include/net/net_namespace.h:311
#13 0xffffffff818bfbe4 in udp_sendmsg (sk=0xffff88803b3c1dc0, msg=<optimized out>, len=<optimized out>) at ../net/ipv4/udp.c:1174
#14 0xffffffff817f6002 in sock_sendmsg_nosec (msg=<optimized out>, sock=<optimized out>) at ../include/linux/uio.h:235
#15 sock_sendmsg (sock=0xffff88803be18000, msg=0xffffc9000029fe28) at ../net/socket.c:657
#16 0xffffffff817f7699 in __sys_sendto (fd=<optimized out>, buff=<optimized out>, len=<optimized out>, flags=64, addr=0x559106c42e18, addr_len=16)
    at ../net/socket.c:1952
#17 0xffffffff817f7720 in __do_sys_sendto (addr_len=<optimized out>, addr=<optimized out>, flags=<optimized out>, len=<optimized out>, buff=<optimized out>,
    fd=<optimized out>) at ../net/socket.c:1964
#18 __se_sys_sendto (addr_len=<optimized out>, addr=<optimized out>, flags=<optimized out>, len=<optimized out>, buff=<optimized out>, fd=<optimized out>)
    at ../net/socket.c:1960
#19 __x64_sys_sendto (regs=<optimized out>) at ../net/socket.c:1960
#20 0xffffffff81002573 in do_syscall_64 (nr=<optimized out>, regs=0xffffc9000029ff58) at ../arch/x86/entry/common.c:296
#21 0xffffffff81c0007c in entry_SYSCALL_64 () at ../arch/x86/entry/entry_64.S:175
#22 0x0000559106c42e18 in ?? ()
#23 0x0000000000000040 in ?? ()
#24 0x0000000000000030 in fixed_percpu_data ()
#25 0x00007ffeb533e9a0 in ?? ()
#26 0x0000000000000000 in ?? ()
```
#### ip_queue_xmit

```c
const struct inet_connection_sock_af_ops ipv4_specific = {
	.queue_xmit	   = ip_queue_xmit,
	.send_check	   = tcp_v4_send_check,
	.rebuild_header	   = inet_sk_rebuild_header,
	.sk_rx_dst_set	   = inet_sk_rx_dst_set,
	.conn_request	   = tcp_v4_conn_request,
	.syn_recv_sock	   = tcp_v4_syn_recv_sock,
	.net_header_len	   = sizeof(struct iphdr),
	.setsockopt	   = ip_setsockopt,
	.getsockopt	   = ip_getsockopt,
	.addr2sockaddr	   = inet_csk_addr2sockaddr,
	.sockaddr_len	   = sizeof(struct sockaddr_in),
#ifdef CONFIG_COMPAT
	.compat_setsockopt = compat_ip_setsockopt,
	.compat_getsockopt = compat_ip_getsockopt,
#endif
	.mtu_reduced	   = tcp_v4_mtu_reduced,
};

__tcp_transmit_skb
        err = icsk->icsk_af_ops->queue_xmit(sk, skb, &inet->cork.fl);


static inline int ip_queue_xmit(struct sock *sk, struct sk_buff *skb,
	      return __ip_queue_xmit(sk, skb, fl, inet_sk(sk)->tos);
                rt = (struct rtable *)__sk_dst_check(sk, 0);
                If the strict routing option flag is set, the destination address is set to be the first address of the IP options:

                skb_push(skb, sizeof(struct iphdr) + (inet_opt ? inet_opt->opt.optlen : 0));
                skb_reset_network_header(skb);
                iph = ip_hdr(skb);
                *((__be16 *)iph) = htons((4 << 12) | (5 << 8) | (tos & 0xff));
                if (ip_dont_fragment(sk, &rt->dst) && !skb->ignore_df)
                  iph->frag_off = htons(IP_DF);
                else
                  iph->frag_off = 0;
                iph->ttl      = ip_select_ttl(inet, &rt->dst);
                iph->protocol = sk->sk_protocol;
                ip_copy_addrs(iph, fl4);

                /* Transport layer set skb->h.foo itself. */

                if (inet_opt && inet_opt->opt.optlen) {
                  iph->ihl += inet_opt->opt.optlen >> 2;
                  ip_options_build(skb, &inet_opt->opt, inet->inet_daddr, rt, 0);
                }

                ip_select_ident_segs(net, skb, sk,
                         skb_shinfo(skb)->gso_segs ?: 1);

                /* TODO : should we use skb->sk here instead of sk ? */
                skb->priority = sk->sk_priority;
                skb->mark = sk->sk_mark;

                res = ip_local_out(net, sk, skb);
```

The getfrag() method is a callback to **copy the actual data from userspace into the SKB**
Before discussing the ip_append_data() method, I want to mention a callback which is a parameter
to the ip_append_data() method: the getfrag() callback. The getfrag() method is a callback to
copy the actual data from userspace into the SKB. In UDPv4, the getfrag() callback is set to be
the generic method, ip_generic_getfrag(). In ICMPv4, the getfrag() callback is set to be a
protocol-specific method, icmp_glue_bits(). Another issue I should mention here is the UDPv4 corking
feature. The UDP_CORK socket option was added in kernel 2.5.44; when this option is enabled, all
data output on this socket is accumulated into a single datagram that is transmitted when the option
is disabled. You can enable and disable this socket option with the setsockopt() system call;
see man 7 udp. In kernel 2.6.39, a lockless transmit fast path was added to the UDPv4 implementation.
With this addition, when the corking feature is not used, the socket lock is not used. So when the
UDP_CORK socket option is set (with the setsockopt() system call), or the MSG_MORE flag is set,
the ip_append_data() method is invoked. And when the UDP_CORK socket option is not set, another
path in the udp_sendmsg() method is used, which does not hold the socket lock and is faster as a
result, and the ip_make_skb() method is invoked. Calling the ip_make_skb() method is similar to
the ip_append_data() and the ip_push_pending_frames() methods rolled into one, except that it
does not send the SKB produced. Sending the SKB is carried out by the ip_send_skb() method.

```c
int ip_local_out(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	int err;

	err = __ip_local_out(net, sk, skb);
	if (likely(err == 1))
		err = dst_output(net, sk, skb);

	return err;
}

int __ip_local_out(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	struct iphdr *iph = ip_hdr(skb);

	iph->tot_len = htons(skb->len);
	ip_send_check(iph);

	/* if egress device is enslaved to an L3 master device pass the
	 * skb to its handler for processing
	 */
	skb = l3mdev_ip_out(sk, skb);
	if (unlikely(!skb))
		return 0;

	skb->protocol = htons(ETH_P_IP);

	return nf_hook(NFPROTO_IPV4, NF_INET_LOCAL_OUT,
		       net, sk, skb, NULL, skb_dst(skb)->dev,
		       dst_output);
}

static inline int dst_output(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	return skb_dst(skb)->output(net, sk, skb);
}

struct rtable *rt_dst_alloc(struct net_device *dev,
			    unsigned int flags, u16 type,
			    bool nopolicy, bool noxfrm, bool will_cache)
{
	struct rtable *rt;

	rt = dst_alloc(&ipv4_dst_ops, dev, 1, DST_OBSOLETE_FORCE_CHK,
		       (will_cache ? 0 : DST_HOST) |
		       (nopolicy ? DST_NOPOLICY : 0) |
		       (noxfrm ? DST_NOXFRM : 0));

	if (rt) {
		rt->rt_genid = rt_genid_ipv4(dev_net(dev));
		rt->rt_flags = flags;
		rt->rt_type = type;
		rt->rt_is_input = 1;
		rt->rt_iif = 0;
		rt->rt_pmtu = 0;
		rt->rt_mtu_locked = 0;
		rt->rt_gw_family = 0;
		rt->rt_gw4 = 0;
		INIT_LIST_HEAD(&rt->rt_uncached);

		rt->dst.output = ip_output;
		if (flags & RTCF_LOCAL)
			rt->dst.input = ip_local_deliver;
	}

	return rt;
}

int ip_output(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	struct net_device *dev = skb_dst(skb)->dev;

	IP_UPD_PO_STATS(net, IPSTATS_MIB_OUT, skb->len);

	skb->dev = dev;
	skb->protocol = htons(ETH_P_IP);

	return NF_HOOK_COND(NFPROTO_IPV4, NF_INET_POST_ROUTING,
			    net, sk, skb, NULL, dev,
			    ip_finish_output,
			    !(IPCB(skb)->flags & IPSKB_REROUTED));
}

static int ip_finish_output(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	int ret;

	ret = BPF_CGROUP_RUN_PROG_INET_EGRESS(sk, skb);
	switch (ret) {
	case NET_XMIT_SUCCESS:
		return __ip_finish_output(net, sk, skb);
	case NET_XMIT_CN:
		return __ip_finish_output(net, sk, skb) ? : ret;
	default:
		kfree_skb(skb);
		return ret;
	}
}

static int __ip_finish_output(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	unsigned int mtu;

#if defined(CONFIG_NETFILTER) && defined(CONFIG_XFRM)
	/* Policy lookup after SNAT yielded a new policy */
	if (skb_dst(skb)->xfrm) {
		IPCB(skb)->flags |= IPSKB_REROUTED;
		return dst_output(net, sk, skb);
	}
#endif
	mtu = ip_skb_dst_mtu(sk, skb);
	if (skb_is_gso(skb))
		return ip_finish_output_gso(net, sk, skb, mtu);

	if (skb->len > mtu || (IPCB(skb)->flags & IPSKB_FRAG_PMTU))
		return ip_fragment(net, sk, skb, mtu, ip_finish_output2);

	return ip_finish_output2(net, sk, skb);
}

static inline int neigh_output(struct neighbour *n, struct sk_buff *skb,
			       bool skip_cache)
{
	const struct hh_cache *hh = &n->hh;

	if ((n->nud_state & NUD_CONNECTED) && hh->hh_len && !skip_cache)
		return neigh_hh_output(hh, skb);
	else
		return n->output(n, skb);
}

static inline int neigh_hh_output(const struct hh_cache *hh, struct sk_buff *skb)
{
	unsigned int hh_alen = 0;
	unsigned int seq;
	unsigned int hh_len;

	do {
		seq = read_seqbegin(&hh->hh_lock);
		hh_len = hh->hh_len;
		if (likely(hh_len <= HH_DATA_MOD)) {
			hh_alen = HH_DATA_MOD;

			/* skb_push() would proceed silently if we have room for
			 * the unaligned size but not for the aligned size:
			 * check headroom explicitly.
			 */
			if (likely(skb_headroom(skb) >= HH_DATA_MOD)) {
				/* this is inlined by gcc */
				memcpy(skb->data - HH_DATA_MOD, hh->hh_data,
				       HH_DATA_MOD);
			}
		} else {
			hh_alen = HH_DATA_ALIGN(hh_len);

			if (likely(skb_headroom(skb) >= hh_alen)) {
				memcpy(skb->data - hh_alen, hh->hh_data,
				       hh_alen);
			}
		}
	} while (read_seqretry(&hh->hh_lock, seq));

	if (WARN_ON_ONCE(skb_headroom(skb) < hh_alen)) {
		kfree_skb(skb);
		return NET_XMIT_DROP;
	}

	__skb_push(skb, hh_len);
	return dev_queue_xmit(skb);
}

return n->output(n, skb) = int neigh_resolve_output(struct neighbour *neigh, struct sk_buff *skb)
```

### IP fragment
```c
static int ip_fragment(struct net *net, struct sock *sk, struct sk_buff *skb,
		       unsigned int mtu,
		       int (*output)(struct net *, struct sock *, struct sk_buff *))
{
	struct iphdr *iph = ip_hdr(skb);

	if ((iph->frag_off & htons(IP_DF)) == 0) /*FD IP_DF not set, do fragemnt */
		return ip_do_fragment(net, sk, skb, output);

	if (unlikely(!skb->ignore_df ||
		     (IPCB(skb)->frag_max_size &&
		      IPCB(skb)->frag_max_size > mtu))) {
		IP_INC_STATS(net, IPSTATS_MIB_FRAGFAILS);
		icmp_send(skb, ICMP_DEST_UNREACH, ICMP_FRAG_NEEDED,
			  htonl(mtu));
		kfree_skb(skb);
		return -EMSGSIZE;
	}

	return ip_do_fragment(net, sk, skb, output);
}
```












lsinitrd initrd ls
switchover GRP update mac ARP

net.ipv4.udp_mem  
udp check sum error


ipv4/ipv6 mtu can be difference

[get_online_cpus()](https://lwn.net/Articles/569686/)

### KASLR
[reference](http://www.wowotech.net/memory_management/441.html)
KASLR is short for `kernel address space layout randomization`

#### enable KASLR
```
CONFIG_RANDOMIZE_BASE=y
```

#### disable KASLR
add `nokaslr` at kernel start attribute






net.ipv4.ip_early_demux = 1





FIB(Forward Information dataBase)






net_dev_init

ethtool -l

softirqd
/proc/interupts
/proc/irg
/etc/sys











## MISC
### sk_buff
skb->truesize=sizeof(struct sk_buff) + SKB_DATA_ALIGN(size)
skb->len是指数据长度（包括数据的包头），即data，tail指针所指的部分。当调用数据区的操作函数如：skb_put,skb_push,skb_pull,skb_trim来在数据区增加或去除协议的头部，尾部时，即在增加或减少len的长度。
skb->datalen是指数据长度（不包括数据的包头）
SKB_DATA_ALIGN(size)是在alloc_skb（size，gfp_mask）函数初始定义时的size的基础上调整后的，通常把调整后的值再赋给size，代替原来的size。
在用alloc_skb（）函数分配缓冲区时，alloc_skb（）建立了套接字缓冲区与struct skb_shared_info结构的关系。
用alloc_skb（）函数分配的缓冲区大小为：
struct sk_buff + SKB_DATA_ALIGN(size) + struct skb_shared_info
这三个的大小之和。



### Udp packet drops and packet receive error difference
If you write a program which receives very high amount of udp packets, it is good to know whether all the packets processed by your program or not.

To get the information about statictics of udp stack, you can use netstat with -anus parameter like that:
```
$ netstat -anus
...
Udp:
    531412134 packets received
    125 packets to unknown port received.
    38491 packet receive errors
    531247364 packets sent
...
```

In this example there are 125 unknown port errors. It can be normal because you can’t control the incoming traffic if no firewall exists and any host can send udp packets to any port, which your server doesn’t listen. In this scenario, you don’t much worry about it. But if this number is so high on average for a day, you may want to capture udp traffics and make further analysis on that.

Another reason for unknown port case can be very important issue. If your program crashes randomly, your system may be receiving packets but because of the crash there are no software to accept that packets, so unknown port error counter increments. You have to fix this at the software level.

There are 38491 packet receive errors and we have to take it seriously. Packet receive errors doesn’t include the problems occured on network card level, it shows only received packets on udp protocol stack. Main reasons for packet receive errors:

udp packet header corruption or checksum problems
packet receive buffer problems in application or kernel side
Unlike TCP, UDP protocol does not have built-in flow-control capabilities, so if you can’t process all of the received packets fast enough, kernel will start to drop new incoming packets because of the socket receive buffer is full. When you don’t make any tuning on udp stack, default udp receive buffer size is between 32-128 kilobytes per socket. You can set it to much higher value with setsockopt like below:
```
int size = 2 * 1024 * 1024;
setsockopt(socket, SOL_SOCKET, SO_RCVBUF, &size, (socklen_t)sizeof(int));
```
2 MB’s of receive buffer handle up to 1 Gbps of data if you can process fast, but you can also increase it to 16 or 32 MB if you need.

Please note that, you can’t set socket receive buffer to maximum value defined in kernel which you can see on /proc/sys/net/core/rmem_max. You have to change this value to use big socket receive buffer size in application:
```
$ sudo sysctl -w net.core.rmem_max=33554432
```

There is one more parameter: netdev_max_backlog. It controls the number of packets allowed to queue for network cards in kernel side. If you are receiving very high level of traffic, you may want to inrease packet backlog queue in kernel to 2000 (Default value is 1000):
```
$ sudo sysctl -w net.core.netdev_max_backlog=2000
```

After these settings, you must check udp packet statistics again. There is no way to get same statistics grouped by process, so they are always showing the whole udp protocol stack. It will be reset with system reboot.

> /proc/sys/net/core/rmem_default 
> /proc/sys/net/core/rmem_max
> When use UDP receive data:
> If haven't invoke setsockopt setting the recv buffer, then the recv buffer size = rmem_default
> If invoke setsockopt to set the recv buffer, the max number can't exceed rmem_max ??? setsockopt max size = rmem_max x 2








ipfrag_time effect 
ttl no performace 

LRO  offset  delete






2020-01-10
CBIS security group map to Linux Bridge
SIP less than 1200 bytes UDP, otherwise TCP



tap/virt-io-net/virt-io-driver


## Virtio
[Virtio: An I/O virtualization framework for Linux](https://developer.ibm.com/articles/l-virtio/)






















## E100 net device driver
```c
static int e100_probe(struct pci_dev *pdev, const struct pci_device_id *ent)
{
	struct net_device *netdev;
	struct nic *nic;
	int err;

        /* allocate struct net_device + struct nic, struct nic as the private data append after struct  net_device, also do init like init network namespace

        TX subqueue is 1, RX subqueue is 1
	dev->num_tx_queues = txqs;
	dev->real_num_tx_queues = txqs;
	dev->num_rx_queues = rxqs;
	dev->real_num_rx_queues = rxqs;
        */
	if (!(netdev = alloc_etherdev(sizeof(struct nic))))
		return -ENOMEM;

	netdev->hw_features |= NETIF_F_RXFCS;
	netdev->priv_flags |= IFF_SUPP_NOFCS;
	netdev->hw_features |= NETIF_F_RXALL;

	netdev->netdev_ops = &e100_netdev_ops;
        /*
                static const struct net_device_ops e100_netdev_ops = {
                  .ndo_open		= e100_open,
                  .ndo_stop		= e100_close,
                  .ndo_start_xmit		= e100_xmit_frame,
                  .ndo_validate_addr	= eth_validate_addr,
                  .ndo_set_rx_mode	= e100_set_multicast_list,
                  .ndo_set_mac_address	= e100_set_mac_address,
                  .ndo_do_ioctl		= e100_do_ioctl,
                  .ndo_tx_timeout		= e100_tx_timeout,
                #ifdef CONFIG_NET_POLL_CONTROLLER
                  .ndo_poll_controller	= e100_netpoll,
                #endif
                  .ndo_set_features	= e100_set_features,
                };
        */
	netdev->ethtool_ops = &e100_ethtool_ops;
        /*
                static const struct ethtool_ops e100_ethtool_ops = {
                        .get_drvinfo		= e100_get_drvinfo,
                        .get_regs_len		= e100_get_regs_len,
                        .get_regs		= e100_get_regs,
                        .get_wol		= e100_get_wol,
                        .set_wol		= e100_set_wol,
                        .get_msglevel		= e100_get_msglevel,
                        .set_msglevel		= e100_set_msglevel,
                        .nway_reset		= e100_nway_reset,
                        .get_link		= e100_get_link,
                        .get_eeprom_len		= e100_get_eeprom_len,
                        .get_eeprom		= e100_get_eeprom,
                        .set_eeprom		= e100_set_eeprom,
                        .get_ringparam		= e100_get_ringparam,
                        .set_ringparam		= e100_set_ringparam,
                        .self_test		= e100_diag_test,
                        .get_strings		= e100_get_strings,
                        .set_phys_id		= e100_set_phys_id,
                        .get_ethtool_stats	= e100_get_ethtool_stats,
                        .get_sset_count		= e100_get_sset_count,
                        .get_ts_info		= ethtool_op_get_ts_info,
                        .get_link_ksettings	= e100_get_link_ksettings,
                        .set_link_ksettings	= e100_set_link_ksettings,
                };
        */
	netdev->watchdog_timeo = E100_WATCHDOG_PERIOD;
	strncpy(netdev->name, pci_name(pdev), sizeof(netdev->name) - 1);

	nic = netdev_priv(netdev);

        /* add napi poll callback to RX data */
	netif_napi_add(netdev, &nic->napi, e100_poll, E100_NAPI_WEIGHT);
	nic->netdev = netdev;
	nic->pdev = pdev;
	nic->msg_enable = (1 << debug) - 1;
	nic->mdio_ctrl = mdio_ctrl_hw;
	pci_set_drvdata(pdev, netdev);

	if ((err = pci_enable_device(pdev))) {
		netif_err(nic, probe, nic->netdev, "Cannot enable PCI device, aborting\n");
		goto err_out_free_dev;
	}

	if (!(pci_resource_flags(pdev, 0) & IORESOURCE_MEM)) {
		netif_err(nic, probe, nic->netdev, "Cannot find proper PCI device base address, aborting\n");
		err = -ENODEV;
		goto err_out_disable_pdev;
	}

	if ((err = pci_request_regions(pdev, DRV_NAME))) {
		netif_err(nic, probe, nic->netdev, "Cannot obtain PCI resources, aborting\n");
		goto err_out_disable_pdev;
	}

	if ((err = pci_set_dma_mask(pdev, DMA_BIT_MASK(32)))) {
		netif_err(nic, probe, nic->netdev, "No usable DMA configuration, aborting\n");
		goto err_out_free_res;
	}

	SET_NETDEV_DEV(netdev, &pdev->dev);

	if (use_io)
		netif_info(nic, probe, nic->netdev, "using i/o access mode\n");

	nic->csr = pci_iomap(pdev, (use_io ? 1 : 0), sizeof(struct csr));
	if (!nic->csr) {
		netif_err(nic, probe, nic->netdev, "Cannot map device registers, aborting\n");
		err = -ENOMEM;
		goto err_out_free_res;
	}

	if (ent->driver_data)
		nic->flags |= ich;
	else
		nic->flags &= ~ich;

        /* init nic */
	e100_get_defaults(nic);

	/* D100 MAC doesn't allow rx of vlan packets with normal MTU */
	if (nic->mac < mac_82558_D101_A4)
		netdev->features |= NETIF_F_VLAN_CHALLENGED;

	/* locks must be initialized before calling hw_reset */
	spin_lock_init(&nic->cb_lock);
	spin_lock_init(&nic->cmd_lock);
	spin_lock_init(&nic->mdio_lock);

	/* Reset the device before pci_set_master() in case device is in some
	 * funky state and has an interrupt pending - hint: we don't have the
	 * interrupt handler registered yet. */
	e100_hw_reset(nic);

	pci_set_master(pdev);

	timer_setup(&nic->watchdog, e100_watchdog, 0);

	INIT_WORK(&nic->tx_timeout_task, e100_tx_timeout_task);

        /* alloc DMA area nic->mm */
	if ((err = e100_alloc(nic))) {
		netif_err(nic, probe, nic->netdev, "Cannot alloc driver memory, aborting\n");
		goto err_out_iounmap;
	}

	if ((err = e100_eeprom_load(nic)))
		goto err_out_free;

	e100_phy_init(nic);

	memcpy(netdev->dev_addr, nic->eeprom, ETH_ALEN);
	if (!is_valid_ether_addr(netdev->dev_addr)) {
		if (!eeprom_bad_csum_allow) {
			netif_err(nic, probe, nic->netdev, "Invalid MAC address from EEPROM, aborting\n");
			err = -EAGAIN;
			goto err_out_free;
		} else {
			netif_err(nic, probe, nic->netdev, "Invalid MAC address from EEPROM, you MUST configure one.\n");
		}
	}

	/* Wol magic packet can be enabled from eeprom */
	if ((nic->mac >= mac_82558_D101_A4) &&
	   (nic->eeprom[eeprom_id] & eeprom_id_wol)) {
		nic->flags |= wol_magic;
		device_set_wakeup_enable(&pdev->dev, true);
	}

	/* ack any pending wake events, disable PME */
	pci_pme_active(pdev, false);

	strcpy(netdev->name, "eth%d");
        /*
         *	Take a completed network device structure and add it to the kernel
         *	interfaces. A %NETDEV_REGISTER message is sent to the netdev notifier
         *	chain. 0 is returned on success. A negative errno code is returned
         *	on a failure to set up the device, or if the name is a duplicate.
         *
         *	This is a wrapper around register_netdevice that takes the rtnl semaphore
         *	and expands the device name if you passed a format string to
         *	alloc_netdev.
         *	alloc and init RX/TX queue, notifiler_call notification
         *	will analyze this later *************************************
        */
	if ((err = register_netdev(netdev))) { //////////////////////////////////////////////////////////
		netif_err(nic, probe, nic->netdev, "Cannot register net device, aborting\n");
		goto err_out_free;
	}

        /* Create DMA pool */
	nic->cbs_pool = dma_pool_create(netdev->name,
			   &nic->pdev->dev,
			   nic->params.cbs.max * sizeof(struct cb),
			   sizeof(u32),
			   0);
	if (!nic->cbs_pool) {
		netif_err(nic, probe, nic->netdev, "Cannot create DMA pool, aborting\n");
		err = -ENOMEM;
		goto err_out_pool;
	}
	netif_info(nic, probe, nic->netdev,
		   "addr 0x%llx, irq %d, MAC addr %pM\n",
		   (unsigned long long)pci_resource_start(pdev, use_io ? 1 : 0),
		   pdev->irq, netdev->dev_addr);

	return 0;
}

static void e100_remove(struct pci_dev *pdev)
{
	struct net_device *netdev = pci_get_drvdata(pdev);

	if (netdev) {
		struct nic *nic = netdev_priv(netdev);
		unregister_netdev(netdev);
		e100_free(nic);
		pci_iounmap(pdev, nic->csr);
		dma_pool_destroy(nic->cbs_pool);
		free_netdev(netdev);
		pci_release_regions(pdev);
		pci_disable_device(pdev);
	}
}


// Invoke when ip link set eth0 up
static int e100_open(struct net_device *netdev)
{
	struct nic *nic = netdev_priv(netdev);
	int err = 0;

        /*
        通常网络设备会定时地检测设备是否处于可传递状态。当状态发生变化时，会调用netif_carrier_on或者netif_carrier_off来通知内核；
        从网上设备插拔网线或者另一端的设备关闭或禁止，都会导致连接状态改变；

        netif_carrier_on—-设备驱动监测到设备传递信号时调用
        netif_carrier_off—-设备驱动监测到设备丢失信号时调用
        */
	netif_carrier_off(netdev);
	if ((err = e100_up(nic)))
		netif_err(nic, ifup, nic->netdev, "Cannot open interface, aborting\n");
	return err;
}

static int e100_up(struct nic *nic)
{
	int err;

	if ((err = e100_rx_alloc_list(nic)))
		return err;
	if ((err = e100_alloc_cbs(nic)))
		goto err_rx_clean_list;
	if ((err = e100_hw_init(nic)))
		goto err_clean_cbs;
	e100_set_multicast_list(nic->netdev);
	e100_start_receiver(nic, NULL);
	mod_timer(&nic->watchdog, jiffies);

        /* register irq for RX */
	if ((err = request_irq(nic->pdev->irq, e100_intr, IRQF_SHARED,
		nic->netdev->name, nic->netdev)))
		goto err_no_irq;
        /*
         *	Allow upper layers to call the device hard_start_xmit routine.
         *	Used for flow control when transmit resources are available.
         */
	netif_wake_queue(nic->netdev);
	napi_enable(&nic->napi);
	/* enable ints _after_ enabling poll, preventing a race between
	 * disable ints+schedule */
	e100_enable_irq(nic);
	return 0;
}

static int e100_close(struct net_device *netdev)
{
	e100_down(netdev_priv(netdev));
	return 0;
}

static irqreturn_t e100_intr(int irq, void *dev_id)
{
	if (likely(napi_schedule_prep(&nic->napi))) {
		e100_disable_irq(nic);
		__napi_schedule(&nic->napi);
	}

	return IRQ_HANDLED;
}

static inline void ____napi_schedule(struct softnet_data *sd,
				     struct napi_struct *napi)
{
	list_add_tail(&napi->poll_list, &sd->poll_list);
	__raise_softirq_irqoff(NET_RX_SOFTIRQ); // XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
}



static __latent_entropy void net_rx_action(struct softirq_action *h)
{
	struct softnet_data *sd = this_cpu_ptr(&softnet_data);
	unsigned long time_limit = jiffies +
		usecs_to_jiffies(netdev_budget_usecs);    // 2000
	int budget = netdev_budget;                       // 300
	LIST_HEAD(list);
	LIST_HEAD(repoll);

	local_irq_disable();
	list_splice_init(&sd->poll_list, &list);
	local_irq_enable();

	for (;;) {
		struct napi_struct *n;

		if (list_empty(&list)) {
			if (!sd_has_rps_ipi_waiting(sd) && list_empty(&repoll))
				goto out;
			break;
		}

		n = list_first_entry(&list, struct napi_struct, poll_list);
		budget -= napi_poll(n, &repoll);

		/* If softirq window is exhausted then punt.
		 * Allow this to run for 2 jiffies since which will allow
		 * an average latency of 1.5/HZ.
		 */
		if (unlikely(budget <= 0 ||
			     time_after_eq(jiffies, time_limit))) {
			sd->time_squeeze++;
			break;
		}
	}

	local_irq_disable();

	list_splice_tail_init(&sd->poll_list, &list);
	list_splice_tail(&repoll, &list);
	list_splice(&list, &sd->poll_list);
	if (!list_empty(&sd->poll_list))
		__raise_softirq_irqoff(NET_RX_SOFTIRQ);

	net_rps_action_and_irq_enable(sd);
out:
	__kfree_skb_flush();
}


/*
 *	Initialize the DEV module. At boot time this walks the device list and
 *	unhooks any devices that fail to initialise (normally hardware not
 *	present) and leaves us with a valid list of present and active devices.
 *
 */

/*
 *       This is called single threaded during boot, so no need
 *       to take the rtnl semaphore.
 */
static int __init net_dev_init(void)
{
	int i, rc = -ENOMEM;

	BUG_ON(!dev_boot_phase);

        // proc/net/dev  可以显示网络接口的一些收发包信息
        // proc/net/softnet_stat 显示每个CPU处理接收包的统计信息
        // 四行表示4个CPU
        // 第一列为该CPU所接收到的所有数据包, 第二列为该CPU缺省queue满的时候, 所删除的包的个数,(没有统计对于使用NAPI的adapter, 由于ring 满而导致删除的包),第三列表示time_squeeze, 就是说,一次的软中断的触发还不能处理完目前已经接收的数据,因而要设置下轮软中断,time_squeeze 就表示设置的次数.  第四--第八暂时没用, 最后一个为CPU_collision,cpu 碰撞ci数.(net/sched.c) 这些信息对分析cpu队列性能会有帮助
        // proc/net/type 输出当前内核注册网络处理协议
	if (dev_proc_init())
		goto out;

        // sys/class/net/xxx/*** 显示了各种设备的一些属性信息，比如IP、掩码、是否有载波等
	if (netdev_kobject_init())
		goto out;

        // 注册网络协议，包类型
	INIT_LIST_HEAD(&ptype_all);
	for (i = 0; i < PTYPE_HASH_SIZE; i++)
		INIT_LIST_HEAD(&ptype_base[i]);

	INIT_LIST_HEAD(&offload_base);

	if (register_pernet_subsys(&netdev_net_ops))
		goto out;

	/*
	 *	Initialise the packet receive queues.
	 */

	for_each_possible_cpu(i) {
		struct work_struct *flush = per_cpu_ptr(&flush_works, i);
		struct softnet_data *sd = &per_cpu(softnet_data, i);

		INIT_WORK(flush, flush_backlog);

		skb_queue_head_init(&sd->input_pkt_queue);
		skb_queue_head_init(&sd->process_queue);
#ifdef CONFIG_XFRM_OFFLOAD
		skb_queue_head_init(&sd->xfrm_backlog);
#endif
		INIT_LIST_HEAD(&sd->poll_list);
		sd->output_queue_tailp = &sd->output_queue;
#ifdef CONFIG_RPS
		sd->csd.func = rps_trigger_softirq;
		sd->csd.info = sd;
		sd->cpu = i;
#endif

		init_gro_hash(&sd->backlog);
		sd->backlog.poll = process_backlog;
		sd->backlog.weight = weight_p;
	}

	dev_boot_phase = 0;

	/* The loopback device is special if any other network devices
	 * is present in a network namespace the loopback device must
	 * be present. Since we now dynamically allocate and free the
	 * loopback device ensure this invariant is maintained by
	 * keeping the loopback device as the first device on the
	 * list of network devices.  Ensuring the loopback devices
	 * is the first device that appears and the last network device
	 * that disappears.
	 */
	if (register_pernet_device(&loopback_net_ops))
		goto out;

	if (register_pernet_device(&default_device_ops))
		goto out;

	open_softirq(NET_TX_SOFTIRQ, net_tx_action);
	open_softirq(NET_RX_SOFTIRQ, net_rx_action);

	rc = cpuhp_setup_state_nocalls(CPUHP_NET_DEV_DEAD, "net/dev:dead",
				       NULL, dev_cpu_dead);
	WARN_ON(rc < 0);
	rc = 0;
out:
	return rc;
}
```






## 2020-10-19
network message tracking, from docker layer to host layer

ftrace perf
