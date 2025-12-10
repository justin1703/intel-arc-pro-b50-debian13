# Intel Arc Pro B50 on Debian 13


This guide explains how I got my **Intel Arc Pro B50** graphics card working on **Debian 13 (Ventana)**. It is based on my hardware:

| Component | Description |
|------------|-------------|
| **CPU** | AMD Ryzen 5 5600G – APU with integrated graphics and excellent energy efficiency |
| **RAM** | 128 GB DDR4 @ 3200 MT/s – originally for many VMs under Proxmox, now provides plenty of headroom for containers and caching |
| **System Drive** | 2 TB NVMe SSD – high I/O performance for system, Docker containers, and databases |
| **Data Storage** | 2 × 4 TB HDD – plenty of space for media, backups, and persistent data |
| **GPU** | Intel Arc Pro B50 with 16 GB VRAM – powerful for LLM workloads and Plex transcoding |
| **Mainboard** | Gigabyte B550M AORUS ELITE AX – stable, well-equipped, and future-proof |
| **Cooler** | NZXT low-profile CPU cooler – quiet and space-saving |
| **Case** | Jonsbo N4 NAS case – compact design with room for multiple 3.5" HDDs |

---

## 1. Move NVMe to chipset M.2 slot

In my case the NVMe drive needed to be installed in the **chipset M.2 slot** instead of the CPU M.2 slot.  

> ⚠️ **Warning:** The Intel Arc Pro B50 uses **PCI Express 5.0 x8**, so using it in a PCIe slot below version 3 may cause issues.

---

## 2. BIOS Settings

- Enable **4G Decoding**  
- Enable **Resizable BAR**  

After reboot, check if the GPU is detected:

```bash
lspci -nnk | grep -iA3 vga
```

## 3. Install the Firmware

```bash
apt update
apt install firmware-intel-graphics
```

## 4. Upgrade to Kernel 6.16+
With Kernel 6.16+, you can monitor GPU parameters such as temperature, fan speed, and utilization using `nvtop`. Run with `sudo nvtop`. This kernel version also provides improved driver compatibility with Intel BattleMage GPUs.

```bash
echo "deb http://deb.debian.org/debian/ trixie-backports main contrib non-free" | sudo tee /etc/apt/sources.list.d/backports.list
apt update
apt search linux-image-6
apt -t stable-backports install linux-image-6.16.12+deb13-amd64 linux-headers-6.16.12+deb13-amd64
update-grub
reboot
```

You can check for the currently installed kernel with this command:
```bash
ls /boot/vmlinuz-*
```

## 5. Install oneAPI

```bash
wget https://registrationcenter-download.intel.com/akdlm/IRC_NAS/bd1d0273-a931-4f7e-ab76-6a2a67d646c7/intel-oneapi-base-toolkit-2025.2.0.592_offline.sh
sudo sh ./intel-oneapi-base-toolkit-2025.2.0.592_offline.sh -a --silent --eula accept
```

> ⚠️ After installing the new kernel or oneAPI toolkit, a system reboot is recommended.
