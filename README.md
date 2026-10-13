# omar-little — my Proxmox learning server

My old gaming rig, rebuilt into a Proxmox VE server where I learn by building, breaking and rebuilding.

> **Motto: zero trust, scalable, reliable, and documented** — documented so clearly that someone who has never seen the system can follow it without guessing.

---

## Why this server matters to me

The first time Proxmox actually worked, it was a pure "it's alive!" Frankenstein moment. This is **my** machine: I built it, I control every layer of it, and nobody else decides how it runs. It felt like getting a third eye, suddenly seeing everything that happens underneath the operating systems I had only used before.

**Where DBA fits in:** I'm studying to become a DBA, and this server is where the two worlds meet. A database is only as reliable as the server, storage, network and security underneath it, and that's exactly the layer I practice here. My Oracle Linux template will be the base for my database lab, running Oracle Database in containers the same way we work in class. I want to be the DBA who understands the whole stack, not just the SQL.

**And honestly:** the more I learn, the more I see how much there is. It feels like I've taken a glass of water from the fountain of knowledge, then turned around and seen the ocean. That didn't scare me away. It put me in awe. I have a lot to learn, and I want to learn it from experienced professionals, with the same enthusiasm that got this server running in the first place.

---

## Why Proxmox

I wanted to run VMs the way production environments do: a bare-metal hypervisor (type 1) instead of a desktop app, built to be deployable, scalable and reliable. Proxmox gave me that with open-source tooling, KVM, ZFS built in, and templates and clustering included, with no license costs.

---

## What I've done so far

### GPU passthrough, done the real way

My NVMe drive shared an IOMMU group with the GPU, which blocks passthrough. Instead of the common ACS override hack, which only makes Linux *pretend* the devices are isolated, I **moved the NVMe to a chipset-connected slot** for real isolation at the hardware level. Then I verified the groups by hand and bound all four of the GPU's functions to `vfio-pci`.

### Templates: VMs I can throw away

I built minimal, right-sized templates for **Debian 13** and **Oracle Linux 10** (2 vCPU, 2 GB RAM, 10 GB disk, headless). New VMs are a clone away, and when one breaks or has done its job, I throw it away and spin up a fresh one. Cattle, not pets.

### Linux experience

I have early, hands-on experience with **Debian** and **Oracle Linux**. On the Oracle side I fell in love with **SELinux** and `semanage`: the idea that every process gets exactly the access it needs, and nothing else. Looking for the equivalent on Debian, I found **AppArmor**, and realized it has a whole set of tools of its own for building and managing profiles. Two different approaches to the same principle: least privilege, enforced by the kernel.

On Oracle Linux I also discovered **Podman**, and I was impressed by how it works together with SELinux. Containers run **rootless**, with no background daemon holding root rights, and SELinux labels keep each container inside its own boundaries. Less running as root means less to attack, the same thinking behind my firewall.

### Hardening baseline (work in progress)

My first Bash script, written without help, automates the basics on a fresh Debian VM: system updates, the ufw firewall and fail2ban. Next comes auditd, unattended upgrades and regular Lynis audits, and then turning it all into an Ansible role.

On my Debian VMs I use **etckeeper**, which tracks every change to `/etc` in Git. A small hook exports only the install log, never file contents, so I keep a record of what I've installed without leaking configuration or keys.

---

## Design philosophy: build once, deploy anywhere

I don't want one hand-tuned machine per job. My goal is a **portable toolkit**:

- **Templates** for headless Debian and Oracle Linux, ready in minutes.
- **Ansible playbooks** that tune them, kept in Git and small enough to carry on a USB stick.
- From there: **deploy 50 VMs** from a template and configure all of them with one playbook, the same way on any rack.

The same recipe, every time, with no human error in between. That's my motto in practice: scalable, reliable and documented.

---

## Next steps

1. **Database lab:** an Oracle Linux clone running Oracle Database Free in rootless Podman, sized up from the template as the database needs it.
2. **Learn Ansible:** start by turning my Debian hardening script into a reusable role.
3. **Cloud-Init**, so every clone sets its own hostname, user and SSH keys automatically.
4. **GPU VM, my "golden jewel":** a Debian clone with the passed-through GPU, the NVIDIA driver and XFCE, driving my 60-inch TV as a daily desktop.
5. **Backups** to a separate machine.
6. **Isolate lab VMs** on their own VLAN once the network supports it.

---

## Hardware

| Part | Choice |
|---|---|
| CPU | Intel i7-10700K (integrated graphics for the host) |
| Motherboard | ASUS ROG Strix Z590-E |
| RAM | 32 GB DDR4 |
| Storage | 500 GB NVMe, ZFS |
| GPU | NVIDIA GTX 1660 Super (passthrough) |
| Network | 2 × Intel 2.5 GbE |

---

## How I learned

Getting Proxmox to run properly took me a while. I reinstalled it several times, because I didn't yet understand some of the settings, or the real power of ZFS and snapshots. Every reinstall taught me something, and that trial and error is a big part of how I learn.

Alongside testing, I learned from:

- **[Barmine Tech](https://www.youtube.com/@BarmineTech):** calm, hype-free Proxmox guides, from full setups to short how-tos
- **[Altsito](https://www.youtube.com/@noclipalt):** short, no-AI security basics (tmux, nmap) with clear ethics: only test your own gear
- **[The official Proxmox VE documentation](https://pve.proxmox.com/pve-docs/)**
- **[The Arch Wiki](https://wiki.archlinux.org/):** explains how Linux works, not just which buttons to press
- Reading, searching and testing until it made sense

**My personal graduation exam:** installing Arch Linux by hand, with no installer babysitting me. Building a system from the ground up is the best way I know to prove to myself that I understand every layer. Not as a daily driver, but as a test of what I've learned.
