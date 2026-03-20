# Setting up Windows 11 VM and VirtIO Drivers

**Objective:** Improve Windows 11 VM performance by migrating from emulated SATA to paravirtual VirtIO storage drivers.

**Background:** By default KVM presents Windows with emulated legacy hardware (SATA controller, e1000e network card etc). VirtIO replaces this with a paravirtual interface designed specifically for virtualisation, significantly reducing CPU overhead and improving disk I/O speed. Given the extremely limited hardware resources of the old laptop I am using for this project, improving the speed and lowering the resource consumption of this VM was essential.

---

**Step 1 — Download VirtIO drivers**
Downloaded `virtio-win.iso` from fedorapeople.org, the official Fedora Project maintained driver package for Windows guests on KVM.

**Step 2 — Attach ISO to VM**
In virt-manager → VM details → Add Hardware → Storage → CDROM → pointed at virtio-win.iso.

**Step 3 — Install VirtIO guest tools**
Booted VM, navigated to the VirtIO ISO in Windows File Explorer, ran `virtio-win-guest-tools.exe` as administrator. This installed all available VirtIO drivers including display (QXL), network, memory balloon, and serial drivers.

**Step 4 — Create dummy VirtIO disk**
On the Ubuntu host:
```bash
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/virtio-dummy.qcow2 1G
```

**Step 5 — Attach dummy disk as VirtIO**
In virt-manager → Add Hardware → Storage → selected existing image (virtio-dummy.qcow2) → bus type VirtIO → Finish.

**Step 6 — Boot and confirm VirtIO storage driver**
Booted VM. Opened Device Manager → Storage Controllers → confirmed **Red Hat VirtIO SCSI controller** present. This confirmed Windows had loaded the VirtIO storage driver.

**Step 7 — Shut down VM and edit XML**
Shut down VM cleanly from within Windows. On Ubuntu host:
```bash
sudo EDITOR=nano virsh edit Win11
```
Located the main disk block (Win11.qcow2) and made two changes:
- Changed `bus='sata'` to `bus='virtio'` in the `<target>` line
- Replaced the `<address type='drive'.../>` line with a PCI address:
```xml
<address type='pci' domain='0x0000' bus='0x08' slot='0x00' function='0x0'/>
```
Saved with Ctrl+X, Y, Enter.

**Step 8 — Remove dummy disk from XML**
In the same XML editor, located and deleted the entire dummy disk block (virtio-dummy.qcow2) to avoid a duplicate VirtIO device ID conflict.

**Step 9 — Boot and verify**
Booted VM — significantly faster boot time, Windows desktop responsive, host CPU fan noticeably quieter. Confirmed both VMs (Windows 11 and Ubuntu Server) could ping each other successfully.

---