# Home Lab Build: KVM/QEMU Virtualisation & Kali Linux Setup

## Overview

This document covers the setup of a KVM/QEMU virtualisation environment on an Ubuntu host machine, including the installation and configuration of a Kali Linux virtual machine as part of an ongoing home lab build for IT and cybersecurity skill development.

**Host machine:** Lenovo running 24.04.4 LTS 
**CPU:** AMD A12 7th Gen — 4 cores with hardware virtualisation support  
**RAM:** 7.2GB  
**Storage:** ~191GB available  

---

## 1. Choosing a Hypervisor

Rather than following the course recommendation of VirtualBox, I opted for KVM/QEMU for the following reasons:

- KVM (Kernel-based Virtual Machine) is native to the Linux kernel — a Type 1 hypervisor rather than a Type 2 application layer solution
- Widely used in enterprise Linux environments (Red Hat, etc.), making it more relevant to professional development
- Fully FOSS with no proprietary dependencies
- Better performance due to kernel-level integration
- `virt-manager` provides an accessible GUI while keeping CLI skills transferable

---

## 2. Pre-Installation Checks

### Verifying Hardware Virtualisation Support

Before installing KVM, confirmed that the CPU supports hardware virtualisation:

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

**Result:** `4` — all four CPU cores report virtualisation support (SVM for AMD).

### System Update

```bash
sudo apt update && sudo apt upgrade
```

---

## 3. Installing KVM/QEMU

### Package Installation

```bash
sudo apt install qemu-kvm libvirt-daemon-system virt-manager bridge-utils
```

**Note:** apt substituted `qemu-system-x86` for `qemu-kvm` — this is expected behaviour, not an error.

### Adding User to Required Groups

To allow virtualisation access without requiring root privileges:

```bash
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```

- `libvirt` group — grants access to the virtualisation daemon
- `kvm` group — grants access to the `/dev/kvm` device interface

Group membership requires a reboot to take effect.

---

## 4. Troubleshooting

### Issue 1: KVM Not Available Warning in virt-manager

After rebooting and launching virt-manager, a warning indicated KVM was unavailable. Investigation:

```bash
ls -la /dev/kvm
# Result: No such file or directory

lsmod | grep kvm
# kvm module present but kvm_amd missing

sudo modprobe kvm_amd
# Result: "could not insert 'kvm_amd': Operation not supported"
```

**Root cause:** Hardware virtualisation (AMD SVM) was disabled in the system BIOS/UEFI despite the CPU supporting it. The `cpuinfo` check confirms CPU capability but not firmware enablement.

**Resolution:** Rebooted into BIOS/UEFI, located the **SVM Mode** setting under CPU/Advanced settings, and enabled it.

**Verification after reboot:**

```bash
ls -la /dev/kvm
# crw-rw----+ 1 root kvm 10, 232 [date] /dev/kvm
```

The `kvm` group having read/write access confirms the earlier `usermod` commands worked correctly.

### Issue 2: Package Name Truncation

During initial installation, `libvirt-daemon-system` failed to install due to the package name being visually truncated in the terminal. Resolved by running the install command again with the full package name confirmed.

**Lesson:** Always verify package names when install commands fail — terminal line wrapping can make truncated names appear complete.

---

## 5. ISO Acquisition and Verification

### Downloading Kali Linux

Downloaded the **Installer ISO** (64-bit) from [kali.org/get-kali](https://kali.org/get-kali).

Image type selection rationale:
- **Installer** chosen over pre-built VM images — pre-built images are packaged for VirtualBox/VMware (.ova, .vmdk) and are not native to KVM's qcow2 format
- **Weekly** builds are untested — not suitable for a stable lab environment
- **NetInstaller** requires internet throughout installation — no advantage here
- **Everything** image is for air-gapped networks — unnecessary

### Checksum Verification

SHA256 checksum obtained from the Kali downloads page and verified using Kleopatra (Gpg4win) on a Windows machine before transferring the ISO via Samba share.

Checksum file format required by sha256sum:

```
<sha256hash>  kali-linux-2025.4-installer-amd64.iso
```

**Note:** Two spaces between hash and filename; trailing newline required — sha256sum will error without it.

**Result:** Verification passed (green — no errors).

ISO transferred to Ubuntu host via Samba share prior to VM creation.

---

## 6. Creating the Kali Linux VM

### VM Configuration in virt-manager

| Setting | Value |
|---|---|
| Installation media | Local ISO |
| OS type | Generic Linux 2024 |
| RAM | 2048MB |
| CPUs | 2 |
| Disk | 25GB (qcow2) |

**Note:** Kali Linux is not in virt-manager's OS database by name — "Generic Linux 2024" provides appropriate defaults for a modern Linux install.

### Installer Issues and Resolutions

**Disk partitioning loop:** The guided partitioning option looped without proceeding. Resolved by navigating back in the installer menu and running **"Detect disks"** explicitly before returning to partitioning. The virtual disk (`/dev/vda`, 26.8GB, VirtIO block device) was then visible and partitioning proceeded normally.

**Final partition layout:**
- Partition 1: ext4 (root filesystem)
- Partition 5: swap

**Keyboard layout:** Installer did not correctly apply UK keyboard layout despite selection during setup. Resolved post-install by editing `/etc/default/keyboard`:

```bash
sudo nano /etc/default/keyboard
# Set: XKBLAYOUT="gb"
```

Applied to the Wayland graphical session with:

```bash
gsettings set org.gnome.desktop.input-sources sources "[('xkb', 'gb')]"
```

### Software Selection

Selected the default live system toolset — standard Kali tooling without unnecessary additions. Additional tools can be installed via apt as needed.

---

## 7. Post-Installation

### System Update

```bash
sudo apt update && sudo apt upgrade
```

### Verifying Installed Tools

```bash
ls /usr/bin | wc -l
# Result: 3593
```

3,593 commands and tools available — the majority CLI-only, reflecting Kali's nature as a comprehensive security platform.

---

## 8. Set up Ubuntu Server VM

Downloaded Ubuntu Server ISO from [ubuntu.com/download/server](https://ubuntu.com/download/server). 
Downloading from the site, verification via PGP key authentication and checksum, and installing on a virtual machine via virt manager followed broadly similar steps as used for Kali, as documented in stages 5 and 6, however this installation was simpler and quicker and proceeded with no unexpected problems to solve.

## 9. Set up Windows 11 VM

Downloaded Windows 11 ISO from [microsoft.com](https://www.microsoft.com/en-gb/software-download/windows11?msockid=0882ae2a3a0b6d5d1b8fb84f3beb6cb3)
As with Ubuntu Server, download, verification and installation as a VM through virt manager was simple and straightfoward, however, the virtual machine was extremely slow and resource intensive given the weak host machine. Further investigation online revealed that dedicated virtual drivers are available from the Fedora Project to speed up virtualisation of Windows 11. This process is covered separately in [Windows 11 VirtIO setup](./win11-virtio.md)

---

## 10. Testing Connection Between Ubuntu Server and Windows 11

On the Ubuntu server, type `ip a`, and on Windows 11 PowerShell, type `ipconfig`. 

This reveals both VMs' IP addresses, assigned by DHCP from libvirt.

On PowerShell, typed `ping 192.168.122.220` to test connection with Ubuntu server. Was succesful. 

On the Ubuntu server, typed `ping 192.168.122.136` to test connection back.  This was unsuccessful. Required adjusting Windows firewall settings to solve.
Opened PowerShell as administrator, and ran this:

```powershell
netsh advfirewall firewall add rule name="Allow ICMPv4" protocol=icmpv4:8,any dir=in action=allow
```

`netsh` - network shell, Windows' command line network configuration tool.
`advfirewall` - specify firewall
`firewall add rule` - adding a new rule
`name="Allow ICMPv4"` - creating a name for the rule for future reference.
`protocol=icmpv4:8, any` - ICMP refers to the protocol used by ping with IPv4. `:8` is an echo request, which is what ping is. `any` means from any source.
`dir=in` - refers to inbound traffic specifically.
`action=allow` - specifically, allow inbound pings.

Pinged again from the Ubuntu server and it was succesful this time.

---

## Lab Environment — Next Steps

With KVM/QEMU configured and a working Kali VM in place, the planned lab environment is:

| VM | Purpose |
|---|---|
| Kali Linux | Attack platform / penetration testing |
| Ubuntu Server | Linux server administration, Nginx |
| Windows Server (eval) | Active Directory domain controller, file server |
| Windows 10/11 (eval) | Domain-joined client machine |

Resource note: With 7.2GB host RAM, VMs will be run selectively rather than simultaneously. Shut down VMs consume disk space only — RAM and CPU are fully returned to the host.

---

## Key Concepts Covered

- Type 1 vs Type 2 hypervisors
- Hardware virtualisation and firmware enablement (AMD SVM / Intel VT-x)
- Linux group-based permissions model
- Cryptographic checksum verification of downloaded images
- Virtual disk formats (qcow2 vs vmdk/ova)
- Linux disk partitioning (ext4, swap, guided vs manual)
- Wayland vs X11 keyboard configuration

---
