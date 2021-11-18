---
title: PCI
---

```sh
#0  pci_scan_device (devfn=<optimized out>, bus=<optimized out>) at ../drivers/pci/probe.c:2439
#1  pci_scan_single_device (devfn=<optimized out>, bus=<optimized out>) at ../drivers/pci/probe.c:2439
#2  pci_scan_single_device (bus=0xffff88803ea2f800, devfn=0) at ../drivers/pci/probe.c:2429
#3  0xffffffff813e505d in pci_scan_slot (bus=0xffff88803ea2f800, devfn=0) at ../drivers/pci/probe.c:2518
#4  0xffffffff813e6110 in pci_scan_child_bus_extend (bus=0xffff88803ea2f800,
    available_buses=<optimized out>) at ../drivers/pci/probe.c:2735
#5  0xffffffff813e62f7 in pci_scan_child_bus (bus=<optimized out>) at ../drivers/pci/probe.c:2865
#6  0xffffffff8142e083 in acpi_pci_root_create (root=<optimized out>, ops=<optimized out>,
    info=0xffff88803e976a80, sysdata=0xffff88803e976ab8) at ../drivers/acpi/pci_root.c:931
#7  0xffffffff81837a3d in pci_acpi_scan_root (root=0xffff88803e96c900) at ../arch/x86/pci/acpi.c:368
#8  0xffffffff8142e2a0 in acpi_pci_root_add (device=0xffff88803e967800, not_used=<optimized out>)
    at ../drivers/acpi/pci_root.c:603
#9  0xffffffff814264e1 in acpi_scan_attach_handler (device=<optimized out>)
    at ../drivers/acpi/scan.c:1941
#10 acpi_bus_attach (device=0xffff88803e967800) at ../drivers/acpi/scan.c:1985
#11 0xffffffff81426456 in acpi_bus_attach (device=0xffff88803e967000) at ../drivers/acpi/scan.c:2006
#12 0xffffffff81426456 in acpi_bus_attach (device=0xffff88803e966800) at ../drivers/acpi/scan.c:2006
#13 0xffffffff8142828e in acpi_bus_scan (handle=0xffffffffffffffff) at ../drivers/acpi/scan.c:2058
#14 0xffffffff82a1011c in acpi_scan_init () at ../drivers/acpi/scan.c:2218
#15 0xffffffff82a0fe5c in acpi_init () at ../drivers/acpi/bus.c:1249
#16 0xffffffff81000d71 in do_one_initcall (fn=0xffffffff82a0fbb1 <acpi_init>) at ../init/main.c:1151
#17 0xffffffff829e20fa in do_initcall_level (command_line=<optimized out>, level=<optimized out>)
    at ../include/linux/compiler.h:310
#18 do_initcalls () at ../init/main.c:1240
#19 do_basic_setup () at ../init/main.c:1260
#20 kernel_init_freeable () at ../init/main.c:1444
#21 0xffffffff81abbd2f in kernel_init (unused=<optimized out>) at ../init/main.c:1351
#22 0xffffffff81c001f2 in ret_from_fork () at ../arch/x86/entry/entry_64.S:352
#23 0x0000000000000000 in ?? ()
```

