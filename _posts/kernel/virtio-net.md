---
title: Virtio-net
---

# virtio-net

> drivers/net/virtio_net.c

## Reference
https://www.redhat.com/en/blog/virtio-networking-first-series-finale-and-plans-2020?source=bloglisting&f%5B0%5D=post_tags%3AVirtualization&search=virtio
https://www.redhat.com/en/blog/introduction-virtio-networking-and-vhost-net
https://www.redhat.com/en/blog/deep-dive-virtio-networking-and-vhost-net
https://lettieri.iet.unipi.it/virtualization/2018/20161124-io-paravirtualization-tour.pdf
https://docs.oasis-open.org/virtio/virtio/v1.1/virtio-v1.1.html

![image](https://learning.oreilly.com/library/view/mastering-kvm-virtualization/9781784399054/graphics/3990_13_37.jpg)

![image](https://www.redhat.com/cms/managed-files/2019-09-12-virtio-networking-fig1.png)

![image](https://www.redhat.com/cms/managed-files/2019-09-12-virtio-networking-fig2.png.jpg)

The virtio network device is a virtual ethernet card, and it supports multiqueue for TX/RX. Empty buffers are placed in N virtqueues for receiving packets, and outgoing packets are enqueued into another N virtqueues for transmission. Another virtqueue is used for driver-device communication outside of the data plane, like to control advanced filtering features, settings like the mac address, or the number of active queues. As a physical NIC, the virtio device supports features such as many offloadings, and can let the real host’s device do them.

To send a packet, the driver sends to the device a buffer that includes metadata information such as desired offloadings for the packet, followed by the packet frame to transmit. The driver can also split the buffer into multiple gather entries, e.g. it can split the metadata header from the packet frame.

These buffers are managed by the driver and mapped by the device. In this case the device is "inside" the hypervisor. Since the hypervisor (qemu) has access to all the guests’ memory it is capable of locating the buffers and reading or writing them.

The following flow diagram shows the virtio-net device configuration and the sending of a packet using virtio-net driver, that communicates with the virtio-net device over PCI. After filling the packet to be sent, it triggers an "available buffer notification", returning the control to QEMU so it can send the packet through the TAP device.

Qemu then notifies the guest that the buffer operation (reading or writing) is done, and it does that by placing the data in the virtqueue and sending a used notification event, triggering an interruption in the guest vCPU.

The process of receiving a packet is similar to that of sending it. The only difference is that, in this case, empty buffers are pre-allocated by the guest and made available to the device so it can write the incoming data to them.

## module params
- napi_weight
- csum
- gso
- napi_tx

## virtnet_netdev
```c
static const struct net_device_ops virtnet_netdev = {
	.ndo_open            = virtnet_open,
	.ndo_stop   	     = virtnet_close,
	.ndo_start_xmit      = start_xmit,
	.ndo_validate_addr   = eth_validate_addr,
	.ndo_set_mac_address = virtnet_set_mac_address,
	.ndo_set_rx_mode     = virtnet_set_rx_mode,
	.ndo_get_stats64     = virtnet_stats,
	.ndo_vlan_rx_add_vid = virtnet_vlan_rx_add_vid,
	.ndo_vlan_rx_kill_vid = virtnet_vlan_rx_kill_vid,
	.ndo_bpf		= virtnet_xdp,
	.ndo_xdp_xmit		= virtnet_xdp_xmit,
	.ndo_features_check	= passthru_features_check,
	.ndo_get_phys_port_name	= virtnet_get_phys_port_name,
	.ndo_set_features	= virtnet_set_features,
};
```

## module_init
```c
static enum cpuhp_state virtionet_online;
static __init int virtio_net_driver_init(void)
{
	ret = cpuhp_setup_state_multi(CPUHP_AP_ONLINE_DYN, "virtio/net:online",
				      virtnet_cpu_online,
				      virtnet_cpu_down_prep);
	virtionet_online = ret;
	ret = cpuhp_setup_state_multi(CPUHP_VIRT_NET_DEAD, "virtio/net:dead",
				      NULL, virtnet_cpu_dead);

        ret = register_virtio_driver(&virtio_net_driver); //////////////////////
        // similiar to pci_register
}
module_init(virtio_net_driver_init);

static __exit void virtio_net_driver_exit(void)
{
	unregister_virtio_driver(&virtio_net_driver);
	cpuhp_remove_multi_state(CPUHP_VIRT_NET_DEAD);
	cpuhp_remove_multi_state(virtionet_online);
}
module_exit(virtio_net_driver_exit);

static struct virtio_driver virtio_net_driver = {
	.feature_table = features,
	.feature_table_size = ARRAY_SIZE(features),
	.feature_table_legacy = features_legacy,
	.feature_table_size_legacy = ARRAY_SIZE(features_legacy),
	.driver.name =	KBUILD_MODNAME,
	.driver.owner =	THIS_MODULE,
	.id_table =	id_table,
	.validate =	virtnet_validate,
	.probe =	virtnet_probe, //////////////////////////////////
	.remove =	virtnet_remove,
	.config_changed = virtnet_config_changed,
#ifdef CONFIG_PM_SLEEP
	.freeze =	virtnet_freeze,
	.restore =	virtnet_restore,
#endif
};


static int virtnet_probe(struct virtio_device *vdev)
{
	int i, err = -ENOMEM;
	struct net_device *dev;
	struct virtnet_info *vi;
	u16 max_queue_pairs;
	int mtu;

	/* Allocate ourselves a network device with room for our info */
	dev = alloc_etherdev_mq(sizeof(struct virtnet_info), max_queue_pairs);

	/* Set up network device as normal. */
	dev->priv_flags |= IFF_UNICAST_FLT | IFF_LIVE_ADDR_CHANGE;
	dev->netdev_ops = &virtnet_netdev;
	dev->features = NETIF_F_HIGHDMA;

	dev->ethtool_ops = &virtnet_ethtool_ops;
	SET_NETDEV_DEV(dev, &vdev->dev);

	dev->vlan_features = dev->features;

	/* MTU range: 68 - 65535 */
	dev->min_mtu = MIN_MTU;
	dev->max_mtu = MAX_MTU;

	/* Set up our device-specific information */
	vi = netdev_priv(dev);
	vi->dev = dev;
	vi->vdev = vdev;
	vdev->priv = vi;

	INIT_WORK(&vi->config_work, virtnet_config_changed_work);

	/* Enable multiqueue by default */
	if (num_online_cpus() >= max_queue_pairs)
		vi->curr_queue_pairs = max_queue_pairs;
	else
		vi->curr_queue_pairs = num_online_cpus();
	vi->max_queue_pairs = max_queue_pairs;

	/* Allocate/initialize the rx/tx queues, and invoke find_vqs */
	err = init_vqs(vi); //////////////// alloc max_queue_pairs rx/tx queue

	netif_set_real_num_tx_queues(dev, vi->curr_queue_pairs);
	netif_set_real_num_rx_queues(dev, vi->curr_queue_pairs);

	virtnet_init_settings(dev);

	err = register_netdev(dev);
	virtio_device_ready(vdev);

	err = virtnet_cpu_notif_add(vi);
	virtnet_set_queues(vi, vi->curr_queue_pairs);

	/* Assume link up if device can't report link status,
	   otherwise get link status from config. */
	netif_carrier_off(dev);
	if (virtio_has_feature(vi->vdev, VIRTIO_NET_F_STATUS)) {
		schedule_work(&vi->config_work);
	} else {
		vi->status = VIRTIO_NET_S_LINK_UP;
		virtnet_update_settings(vi);
		netif_carrier_on(dev);
	}

	return 0;
}


static const struct net_device_ops virtnet_netdev = {
	.ndo_open            = virtnet_open,
	.ndo_stop   	     = virtnet_close,
	.ndo_start_xmit      = start_xmit,
	.ndo_validate_addr   = eth_validate_addr,
	.ndo_set_mac_address = virtnet_set_mac_address,
	.ndo_set_rx_mode     = virtnet_set_rx_mode,
	.ndo_get_stats64     = virtnet_stats,
	.ndo_vlan_rx_add_vid = virtnet_vlan_rx_add_vid,
	.ndo_vlan_rx_kill_vid = virtnet_vlan_rx_kill_vid,
	.ndo_bpf		= virtnet_xdp,
	.ndo_xdp_xmit		= virtnet_xdp_xmit,
	.ndo_features_check	= passthru_features_check,
	.ndo_get_phys_port_name	= virtnet_get_phys_port_name,
	.ndo_set_features	= virtnet_set_features,
};


static int virtnet_open(struct net_device *dev)
{
	struct virtnet_info *vi = netdev_priv(dev);
	int i, err;

	for (i = 0; i < vi->max_queue_pairs; i++) {
		if (i < vi->curr_queue_pairs)
			/* Make sure we have some buffers: if oom use wq. */
			if (!try_fill_recv(vi, &vi->rq[i], GFP_KERNEL))
				schedule_delayed_work(&vi->refill, 0);

		err = xdp_rxq_info_reg(&vi->rq[i].xdp_rxq, dev, i);
		if (err < 0)
			return err;

		err = xdp_rxq_info_reg_mem_model(&vi->rq[i].xdp_rxq,
						 MEM_TYPE_PAGE_SHARED, NULL);
		if (err < 0) {
			xdp_rxq_info_unreg(&vi->rq[i].xdp_rxq);
			return err;
		}

		virtnet_napi_enable(vi->rq[i].vq, &vi->rq[i].napi);
		virtnet_napi_tx_enable(vi, vi->sq[i].vq, &vi->sq[i].napi);
	}

	return 0;
}
```

## send packet
```c
static netdev_tx_t start_xmit(struct sk_buff *skb, struct net_device *dev)
{
	struct virtnet_info *vi = netdev_priv(dev);
	int qnum = skb_get_queue_mapping(skb); // select tx queue
	struct send_queue *sq = &vi->sq[qnum];
	struct netdev_queue *txq = netdev_get_tx_queue(dev, qnum);
	bool kick = !netdev_xmit_more();
	bool use_napi = sq->napi.weight;

	/* Free up any pending old buffers before queueing new ones. */
	free_old_xmit_skbs(sq, false);

	if (use_napi && kick)
		virtqueue_enable_cb_delayed(sq->vq);

	/* timestamp packet in software */
	skb_tx_timestamp(skb);

	/* Try to transmit */
	err = xmit_skb(sq, skb);

	/* This should not happen! */
	if (unlikely(err)) {
		dev->stats.tx_fifo_errors++;
		if (net_ratelimit())
			dev_warn(&dev->dev,
				 "Unexpected TXQ (%d) queue failure: %d\n",
				 qnum, err);
		dev->stats.tx_dropped++;
		dev_kfree_skb_any(skb);
		return NETDEV_TX_OK;
	}

	/* Don't wait up for transmitted skbs to be freed. */
	if (!use_napi) {
		skb_orphan(skb);
		nf_reset_ct(skb);
	}

	/* If running out of space, stop queue to avoid getting packets that we
	 * are then unable to transmit.
	 * An alternative would be to force queuing layer to requeue the skb by
	 * returning NETDEV_TX_BUSY. However, NETDEV_TX_BUSY should not be
	 * returned in a normal path of operation: it means that driver is not
	 * maintaining the TX queue stop/start state properly, and causes
	 * the stack to do a non-trivial amount of useless work.
	 * Since most packets only take 1 or 2 ring slots, stopping the queue
	 * early means 16 slots are typically wasted.
	 */
	if (sq->vq->num_free < 2+MAX_SKB_FRAGS) {
		netif_stop_subqueue(dev, qnum);
		if (!use_napi &&
		    unlikely(!virtqueue_enable_cb_delayed(sq->vq))) {
			/* More just got used, free them then recheck. */
			free_old_xmit_skbs(sq, false);
			if (sq->vq->num_free >= 2+MAX_SKB_FRAGS) {
				netif_start_subqueue(dev, qnum);
				virtqueue_disable_cb(sq->vq);
			}
		}
	}

	if (kick || netif_xmit_stopped(txq)) {
		if (virtqueue_kick_prepare(sq->vq) && virtqueue_notify(sq->vq)) {
			u64_stats_update_begin(&sq->stats.syncp);
			sq->stats.kicks++;
			u64_stats_update_end(&sq->stats.syncp);
		}
	}

	return NETDEV_TX_OK;
}
```

```sh
#0  virtqueue_add_split (gfp=<optimized out>, ctx=<optimized out>, data=<optimized out>, in_sgs=<optimized out>, out_sgs=<optimized out>, total_sg=<optimized out>, sgs=<optimized out>, _vq=<optimized out>)
    at ../drivers/virtio/virtio_ring.c:434
#1  virtqueue_add (_vq=0xffff88803b8c1600, sgs=<optimized out>, total_sg=1, out_sgs=<optimized out>, in_sgs=<optimized out>, data=0xffff88803d2ec200, ctx=0x0 <fixed_percpu_data>, gfp=2592)
    at ../drivers/virtio/virtio_ring.c:1706
#2  0xffffffffc0006b96 in virtqueue_add_outbuf (vq=<optimized out>, sg=0xffff88803ba4e008, num=<optimized out>, data=<optimized out>, gfp=<optimized out>) at ../drivers/virtio/virtio_ring.c:1763
#3  0xffffffffc00209e9 in xmit_skb (skb=<optimized out>, sq=<optimized out>) at ../drivers/net/virtio_net.c:1548
#4  start_xmit (skb=0xffff88803d2ec200, dev=0xffff88803d1c0000) at ../drivers/net/virtio_net.c:1571
#5  0xffffffff8188651d in __netdev_start_xmit (more=<optimized out>, dev=<optimized out>, skb=<optimized out>, ops=<optimized out>) at ../include/linux/netdevice.h:4418
#6  netdev_start_xmit (more=<optimized out>, txq=<optimized out>, dev=<optimized out>, skb=<optimized out>) at ../include/linux/netdevice.h:4432
#7  xmit_one (more=<optimized out>, txq=<optimized out>, dev=<optimized out>, skb=<optimized out>) at ../net/core/dev.c:3199
#8  dev_hard_start_xmit (first=0xffff88803d2ec200, dev=0xffff88803d1c0000, txq=<optimized out>, ret=<optimized out>) at ../net/core/dev.c:3215
#9  0xffffffff818c4926 in sch_direct_xmit (skb=0xffff88803d2ec200, q=0xffff88803d1e9400, dev=0xffff88803d1c0000, txq=0xffff88803d31bc00, root_lock=0x0 <fixed_percpu_data>, validate=<optimized out>)
    at ../net/sched/sch_generic.c:313
#10 0xffffffff81886d7e in __dev_xmit_skb (txq=<optimized out>, dev=<optimized out>, q=<optimized out>, skb=<optimized out>) at ../net/core/dev.c:3400
#11 __dev_queue_xmit (skb=<optimized out>, sb_dev=0x0 <fixed_percpu_data>) at ../net/core/dev.c:3761
#12 0xffffffff81904b05 in neigh_output (skip_cache=<optimized out>, skb=<optimized out>, n=<optimized out>) at ../include/net/neighbour.h:511
#13 ip_finish_output2 (net=<optimized out>, sk=<optimized out>, skb=0xffff88803d2ec200) at ../net/ipv4/ip_output.c:228
#14 0xffffffff81906801 in ip_finish_output (skb=<optimized out>, sk=<optimized out>, net=<optimized out>) at ../net/ipv4/ip_output.c:318
#15 NF_HOOK_COND (pf=<optimized out>, hook=<optimized out>, in=<optimized out>, okfn=<optimized out>, cond=<optimized out>, out=<optimized out>, skb=<optimized out>, sk=<optimized out>, net=<optimized out>)
    at ../include/linux/netfilter.h:294
#16 ip_output (net=0xffffffff8251c940 <init_net>, sk=0xffff88803e685540, skb=0xffff88803d2ec200) at ../net/ipv4/ip_output.c:432
#17 0xffffffff81907115 in ip_send_skb (net=0xffffffff8251c940 <init_net>, skb=<optimized out>) at ../net/ipv4/ip_output.c:1562
#18 0xffffffff81933cfe in udp_send_skb (skb=0xffff88803d2ec200, fl4=0xffff88803e6858a0, cork=<optimized out>, cork=<optimized out>) at ../include/net/net_namespace.h:320
#19 0xffffffff81934916 in udp_sendmsg (sk=0xffff88803e685540, msg=<optimized out>, len=<optimized out>) at ../net/ipv4/udp.c:1178
#20 0xffffffff81864647 in sock_sendmsg_nosec (msg=<optimized out>, sock=<optimized out>) at ../include/linux/uio.h:235
#21 sock_sendmsg (sock=0xffff88803b021200, msg=0xffffc900001c3e18) at ../net/socket.c:657
#22 0xffffffff81864880 in ____sys_sendmsg (sock=0xffff88803b021200, msg_sys=0xffffc900001c3e18, flags=<optimized out>, used_address=0xffffc900001c3e70, allowed_msghdr_flags=<optimized out>)
    at ../net/socket.c:2297
#23 0xffffffff81864a28 in ___sys_sendmsg (sock=<optimized out>, msg=<optimized out>, msg_sys=0xffffc900001c3e18, flags=16384, used_address=0xffffc900001c3e70, allowed_msghdr_flags=128) at ../net/socket.c:2351
#24 0xffffffff81865fb5 in __sys_sendmmsg (fd=<optimized out>, mmsg=0x7f5f43c5bd20, vlen=<optimized out>, flags=<optimized out>, forbid_cmsg_compat=<optimized out>) at ../net/socket.c:2454
#25 0xffffffff81866100 in __do_sys_sendmmsg (flags=<optimized out>, vlen=<optimized out>, mmsg=<optimized out>, fd=<optimized out>) at ../net/socket.c:2483
#26 __se_sys_sendmmsg (flags=<optimized out>, vlen=<optimized out>, mmsg=<optimized out>, fd=<optimized out>) at ../net/socket.c:2480
#27 __x64_sys_sendmmsg (regs=<optimized out>) at ../net/socket.c:2480
#28 0xffffffff81002678 in do_syscall_64 (nr=<optimized out>, regs=0xffffc900001c3f58) at ../arch/x86/entry/common.c:290
#29 0xffffffff81c0007c in entry_SYSCALL_64 () at ../arch/x86/entry/entry_64.S:175
#30 0x0000000000000000 in ?? ()
```

## Receive packaet
virtnet_probe
init_vqs
virtnet_alloc_queues
	for (i = 0; i < vi->max_queue_pairs; i++) {
		netif_napi_add(vi->dev, &vi->rq[i].napi, virtnet_poll,napi_weight); 
		netif_tx_napi_add(vi->dev, &vi->sq[i].napi, virtnet_poll_tx,napi_tx ? napi_weight : 0);

 static int virtnet_poll(struct napi_struct *napi, int budget)
```c
  static void receive_buf(struct virtnet_info *vi, struct receive_queue *rq,
                          void *buf, unsigned int len, void **ctx,
                          unsigned int *xdp_xmit,
                          struct virtnet_rq_stats *stats)
  {
          struct net_device *dev = vi->dev;
          struct sk_buff *skb;
          struct virtio_net_hdr_mrg_rxbuf *hdr;

          if (unlikely(len < vi->hdr_len + ETH_HLEN)) {
                  pr_debug("%s: short packet %i\n", dev->name, len);
                  dev->stats.rx_length_errors++;
                  if (vi->mergeable_rx_bufs) {
                          put_page(virt_to_head_page(buf));
                  } else if (vi->big_packets) {
                          give_pages(rq, buf);
                  } else {
                          put_page(virt_to_head_page(buf));
                  }
                  return;
          }

          if (vi->mergeable_rx_bufs)
                  skb = receive_mergeable(dev, vi, rq, buf, ctx, len, xdp_xmit,
                                          stats);
          else if (vi->big_packets)
                  skb = receive_big(dev, vi, rq, buf, len, stats);
          else
                  skb = receive_small(dev, vi, rq, buf, ctx, len, xdp_xmit, stats);

          if (unlikely(!skb))
                  return;

          hdr = skb_vnet_hdr(skb);

          if (hdr->hdr.flags & VIRTIO_NET_HDR_F_DATA_VALID)
                  skb->ip_summed = CHECKSUM_UNNECESSARY;

          if (virtio_net_hdr_to_skb(skb, &hdr->hdr,
                                  ¦ virtio_is_little_endian(vi->vdev))) {
                  net_warn_ratelimited("%s: bad gso: type: %u, size: %u\n",
                                  ¦    dev->name, hdr->hdr.gso_type,
                                  ¦    hdr->hdr.gso_size);
                  goto frame_err;
          }

          skb_record_rx_queue(skb, vq2rxq(rq->vq));
          skb->protocol = eth_type_trans(skb, dev);
          pr_debug("Receiving skb proto 0x%04x len %i type %i\n",
                  ¦ntohs(skb->protocol), skb->len, skb->pkt_type);

          napi_gro_receive(&rq->napi, skb);
          return;

  frame_err:
          dev->stats.rx_frame_errors++;
          dev_kfree_skb(skb);
  }

```

 ```sh
#0  mergeable_ctx_to_truesize (mrg_ctx=<optimized out>) at ../drivers/net/virtio_net.c:916
#1  receive_mergeable (stats=<optimized out>, xdp_xmit=<optimized out>, len=54, ctx=0x600, buf=<optimized out>, rq=<optimized out>, vi=<optimized out>, dev=<optimized out>) at ../drivers/net/virtio_net.c:916
#2  receive_buf (vi=0xffff88803d388840, rq=0xffff88803b95e000, buf=0xffff88803ba23600, len=<optimized out>, ctx=0x600, xdp_xmit=<optimized out>, stats=0xffffc90000003e68) at ../drivers/net/virtio_net.c:1033
#3  0xffffffffc00230eb in virtnet_receive (xdp_xmit=<optimized out>, budget=<optimized out>, rq=<optimized out>) at ../drivers/net/virtio_net.c:1323
#4  virtnet_poll (napi=0xffff88803b95e008, budget=64) at ../drivers/net/virtio_net.c:1428
#5  0xffffffff81889cd1 in napi_poll (repoll=<optimized out>, n=<optimized out>) at ../net/core/dev.c:6311
#6  net_rx_action (h=<optimized out>) at ../net/core/dev.c:6379
#7  0xffffffff81e000e0 in __do_softirq () at ../kernel/softirq.c:292
#8  0xffffffff81070220 in invoke_softirq () at ../kernel/softirq.c:373
#9  irq_exit () at ../kernel/softirq.c:413
#10 0xffffffff81c01b0d in exiting_irq () at ../arch/x86/include/asm/apic.h:536
#11 do_IRQ (regs=<optimized out>) at ../arch/x86/kernel/irq.c:263
#12 0xffffffff81c00a0f in common_interrupt () at ../arch/x86/entry/entry_64.S:607
#13 0xffffffff82403df8 in init_thread_union ()

 ```


## multi queue features
```c
#define VIRTNET_FEATURES \
	VIRTIO_NET_F_CSUM, VIRTIO_NET_F_GUEST_CSUM, \
	VIRTIO_NET_F_MAC, \
	VIRTIO_NET_F_HOST_TSO4, VIRTIO_NET_F_HOST_UFO, VIRTIO_NET_F_HOST_TSO6, \
	VIRTIO_NET_F_HOST_ECN, VIRTIO_NET_F_GUEST_TSO4, VIRTIO_NET_F_GUEST_TSO6, \
	VIRTIO_NET_F_GUEST_ECN, VIRTIO_NET_F_GUEST_UFO, \
	VIRTIO_NET_F_MRG_RXBUF, VIRTIO_NET_F_STATUS, VIRTIO_NET_F_CTRL_VQ, \
	VIRTIO_NET_F_CTRL_RX, VIRTIO_NET_F_CTRL_VLAN, \
	VIRTIO_NET_F_GUEST_ANNOUNCE, **VIRTIO_NET_F_MQ**, \
	VIRTIO_NET_F_CTRL_MAC_ADDR, \
	VIRTIO_NET_F_MTU, VIRTIO_NET_F_CTRL_GUEST_OFFLOADS, \
	VIRTIO_NET_F_SPEED_DUPLEX, VIRTIO_NET_F_STANDBY

static unsigned int features[] = {
	VIRTNET_FEATURES,
}

static int virtnet_probe(struct virtio_device *vdev)
{
        /* Find if host supports multiqueue virtio_net device */
        err = virtio_cread_feature(vdev, **VIRTIO_NET_F_MQ**,
                                   struct virtio_net_config,
                                   max_virtqueue_pairs, &max_queue_pairs);

        /* We need at least 2 queue's */
        if (err || max_queue_pairs < VIRTIO_NET_CTRL_MQ_VQ_PAIRS_MIN ||
        ¦   max_queue_pairs > VIRTIO_NET_CTRL_MQ_VQ_PAIRS_MAX ||
        ¦   !virtio_has_feature(vdev, VIRTIO_NET_F_CTRL_VQ))
                **max_queue_pairs = 1;**

        /* Allocate ourselves a network device with room for our info */
        dev = alloc_etherdev_mq(sizeof(struct virtnet_info), max_queue_pairs);
```

the feature should both supported by driver and device

so for the device part
```c
static int virtio_dev_probe(struct device *_d)
	device_features = dev->config->get_features(dev);

get_features = vp_get_features

  /* virtio config->get_features() implementation */
  static u64 vp_get_features(struct virtio_device *vdev)
  {
          struct virtio_pci_device *vp_dev = to_vp_device(vdev);
          u64 features;

          vp_iowrite32(0, &vp_dev->common->device_feature_select);
          features = vp_ioread32(&vp_dev->common->device_feature);
          vp_iowrite32(1, &vp_dev->common->device_feature_select);
          features |= ((u64)vp_ioread32(&vp_dev->common->device_feature) << 32);

          return features;
  }
```
so device's feature list is read from device registers, for virtio devices, this is set by the emmulate like qemu.


the struct device is constructed by pci drivers when with probe.
