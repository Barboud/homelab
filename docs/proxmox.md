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

I can use any hostname, like `homelab.company.com`. I think that is what companies do in real life: they use their own domain, but with network traffic restrictions of course. The problem is, if the hostname matches an existing domain name, I can't open my local Proxmox server. That is why I use `.arpa`: it is reserved, and no one can register it, so it will never match a real domain. It works locally even if someone else has the same hostname, because every server runs locally, not in public.

## Known issues

- Server freezes, I can't connect to Proxmox → [incident](incidents/2026-11-13-server-freezes.md) (fixed)

## VMs

| VM | OS | CPU | RAM | Disk | IP |
|---|---|---|---|---|---|
| `server-production` | Ubuntu 26.04.1 LTS | 2 cores | 4GB | 32GB | `192.168.1.201` |

### server-production

Runs my websites with Docker. Hostname: `web-host.home.arpa`.

- Network: `vmbr0`, VirtIO
- IP reserved on the router + DNS entry
- Login: `ssh <user>@web-host.home.arpa`
