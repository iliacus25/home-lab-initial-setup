# Home Lab Setup

## Overview

In order to practice IT tech support concepts such as networking, directory services, and security, I decided to set up a lab environment where I could explore these concepts using virtual machines.
An old laptop was employed for this purpose. I installed Ubuntu OS  in order to also have an environment for learning to use Linux, and used KVM/QEMU virtualisation due to it's effective integration with the Linux kernel.

---

## Objectives

* Build a reliable virtualisation environment using open-source tools
* Deploy multiple operating systems for different lab scenarios
* Develop practical skills in system administration and networking
* Create a reusable and documented lab environment

## Base System

* Host OS: Ubuntu (desktop)
* Hardware: Older laptop (repurposed)
* Virtualisation: KVM + QEMU
* Management Tool: virt-manager

---

## Setup Stages

### 1. Install KVM/QEMU

Set up the host machine with virtualisation support:

* Installed KVM and QEMU
* Configured user permissions
* Verified virtualisation support
* Installed and configured `virt-manager`

---

### 2. Kali Linux VM

Deployed a Kali Linux virtual machine for cybersecurity tooling and penetration testing labs.

* Installed from ISO
* Configured basic resources (CPU, RAM, disk)
* Verified network connectivity

---

### 3. Ubuntu Server VM

Deployed an Ubuntu Server instance to simulate a production-style Linux environment.

* Installed from ISO
* Configured networking
* Used for services and administration practice

---

### 4. Windows 11 VM

Set up a Windows 11 virtual machine.

This required additional configuration using VirtIO drivers for storage and performance optimisation, which introduced some complexity.

To keep the main setup flow clean, this process is documented separately.

---

## Use Cases

This lab environment will be used for:

* Networking practice (routing, DNS, services)
* Linux system administration
* Windows administration
* Cybersecurity labs and tooling
* General experimentation and troubleshooting

---

## Future Improvements

The host laptop is unfortunately old and with limited hardware resources, making running multiple VMs into a slow process and placing a cap on the number of VMs that can be run simultaneously. Currently in the process of working towards building a dedicated desktop PC for this purpose, until then, there will be a limit on how extensive the virtual environment can be. 

* Add internal virtual networking between VMs
* Introduce firewall and segmentation
* Deploy additional services (e.g., Samba, web servers)

---

## Notes

This repository focuses on clarity and reproducibility rather than optimisation. Some steps may not be the most efficient, but reflect real-world troubleshooting and learning.

---

## Disclaimer

This lab is for educational purposes only. Any tools or techniques used within this environment should be applied ethically and within legal boundaries.

---

## Getting Started

Clone the repository and follow the documentation in order:

1. KVM/QEMU setup
2. Linux VMs (Kali, Ubuntu Server)
3. Windows VM (advanced setup)

---

## Author

Documented as part of ongoing learning in IT, networking, and cybersecurity.
