# Proxmox

Proxmox VE 9.2 runs on the HP EliteDesk 800 G5 DM. All my VMs will run on it.

## Why

I want to run many VMs on one machine.

## Install

1. Made a USB installer from the Proxmox VE ISO.
2. Booted from USB.
3. Chose **Install Proxmox VE (Graphical)**.
4. Disk: the SSD, filesystem `ext4`.
5. Set country, timezone (`Europe/Amsterdam`), keyboard, root password and email.
6. Set the network.
7. Removed the USB and rebooted.

## Network

| Setting | Value |
|---|---|
| Hostname | `pve.home.arpa` |
| IP | `192.168.1.200/24` (static, reserved on the router) |
| Gateway | `192.168.1.1` (home router) |
| DNS | `192.168.1.1` |
| Web UI | `https://192.168.1.200:8006` |

### Why the name `pve.home.arpa`



## Known issues

- Network drops, I can't connect to Proxmox → [incident](incidents/2026-11-13-e1000e-network-drop.md)

## VMs

not yet
