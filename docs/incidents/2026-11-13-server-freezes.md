# 2026-11-13: Server freezes

Status: fixed

## What happened
- After the install, Proxmox worked for some minutes, then the connection was gone. No ping, no SSH.
- The network light was still on, so the server froze, it did not turn off.
- Sometimes the boot also stops at `Loading initial ramdisk ...`.
- The logs show no error. They just stop.

## What I checked
- Temperature: normal.
- RAM test (F2): passed.
- I re-seated the RAM, because it just arrived and maybe it moved during transport.
- I updated the BIOS from 02.13.00 to 02.27.00 via Ethernet.

It still froze after each of these.

## What I changed
I suspected idle mode. Many devices use it to save power. I had the same problem when I turned my old laptop into a server, so I followed the same idea with this HP mini PC.
I changed one thing, tested, and if it still froze I tried the next thing.

### 1. Network card
- **Why:** The first problem was the connection. I read on the Proxmox forum that the Intel network card in this kind of PC (`e1000e`) can hang. The fix there was to turn off TSO and GSO.
  - TSO = TCP Segmentation Offload
  - GSO = Generic Segmentation Offload
  - Turning them off means the CPU splits the packets, not the NIC.

- **What I did:** I added this line under `iface nic0 inet manual` in `/etc/network/interfaces`:

```text
post-up /usr/sbin/ethtool -K nic0 tso off gso off
```

- **Result:** Still froze.

### 2. Graphics driver
This is not clearly related to the freezes, but it could be.
- **Why:** The boot sometimes stopped at `Loading initial ramdisk ...`. At this moment Linux loads the graphics driver. The log showed an error from it (`i915`).
- **What I did:** I told Linux to not load the graphics driver. I don't need the screen, I use the browser and SSH.

Open the file with the boot settings:

```bash
nano /etc/default/grub
```

GRUB = GRand Unified Bootloader. This file has the settings the kernel uses at boot.

Find this line:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
```

Add `nomodeset`:

nomodeset = no mode set. The kernel does not change the screen mode. It keeps the simple mode from the BIOS.

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet nomodeset"
```

Save, then apply and reboot:

```bash
update-grub
reboot
```

Check it works (the output must have `nomodeset`):

```bash
cat /proc/cmdline
```

- **Result:** Still froze.

### 3. Power saving
- **Why:** It always froze when the server was doing nothing, and the logs showed no error. I think a part goes to sleep to save power and does not wake up. People on the Proxmox forum fixed the same problem by turning power saving off.
- **What I did:** In the same file, I added 3 more settings to the same line:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet nomodeset nvme_core.default_ps_max_latency_us=0 pcie_aspm=off intel_idle.max_cstate=1"
```

| Setting | What it does |
|---|---|
| `nvme_core.default_ps_max_latency_us=0` | The SSD never goes to sleep. |
| `pcie_aspm=off` | The PCIe slots (SSD, network card) never go to sleep. |
| `intel_idle.max_cstate=1` | The CPU never goes to deep sleep. It uses a bit more power. |

Then the same steps: `update-grub`, `reboot`, `cat /proc/cmdline`.

- **Result:** Fixed. It ran more than 24 hours with no freeze (`uptime`: up 1 day), and it boots fine after reboot.

## Fix
After step 3 the server stopped freezing. I keep all 3 changes.


## Words
- **Driver**: a small program that lets Linux talk to a part of the PC.
- **e1000e**: the driver for the Intel network card in this PC.
- **TSO and GSO** (TCP Segmentation Offload, Generic Segmentation Offload): the network card cuts big data into small packets, so the CPU does less work. On this card it can make the card hang. When I turn it off, the CPU does this job.
- **Kernel**: the core of Linux. It starts first and controls the hardware.
- **GRUB**: the program that starts Linux when the PC boots.
- **Kernel options**: settings I give the kernel when it starts. They are in `/etc/default/grub`.
- **update-grub**: saves my changes into GRUB. They work after reboot.
- **nomodeset**: tells the kernel to not load the graphics driver. The screen still works, but simple.

## Sources
- [Proxmox forum: e1000e Hardware Unit Hang](https://forum.proxmox.com/threads/detected-hardware-unit-hang.178800/)
- [Proxmox forum: e1000e on EliteDesk 800 G6 Mini, Proxmox 9.2](https://forum.proxmox.com/posts/856567/)
- [Proxmox forum: freeze with no errors in logs, C-states and ASPM](https://forum.proxmox.com/goto/post?id=831815)
- [Proxmox forum: NVMe power saving fix](https://forum.proxmox.com/threads/workaround-nvme-nvme0-frozen-state-error-detected-samsung-980-m-2-nvme.108162/)
- [HP: update BIOS from F10](https://support.hp.com/au-en/document/ish_4208192-2358829-16)
