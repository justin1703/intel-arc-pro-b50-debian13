# Intel Arc Pro B50 on Debian 13
This guide explains how I got my **Intel Arc Pro B50** graphics card working on **Debian 13 (Ventana)**. 

<p align="center">
  <img src="ressources/homeserver.jpg" width="500" alt="Homeserver">
  <br>
  <sub><em>My Homeserver (December 2025)</em></sub>
</p>

---

# Table of Contents

1. [Hardware](#hardware)
2. [Installation](#installation)
   1. [Move NVMe to chipset M.2 slot](#1-move-nvme-to-chipset-m2-slot)
   2. [BIOS Settings](#2-bios-settings)
   3. [Install the Firmware](#3-install-the-firmware)
   4. [Upgrade to Kernel 6.16+](#4-upgrade-to-kernel-616)
   5. [Install oneAPI](#5-install-oneapi)
3. [Ollama (inside a Ubuntu 24.04 Docker-Container)](#ollama-inside-a-ubuntu-2404-docker-container)
   1. [Installation inside a default Ubuntu 24.04 Container](#installation-inside-a-default-ubuntu-2404-container)
   2. [Models that I tried with Ollama](#models-that-i-tried-with-ollama)
4. [Setup a prepared Compose file running Ollama on 0.0.0.0](#setup-a-prepared-compose-file-running-ollama-on-0000)

---
## Hardware

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
## Installation

### 1. Move NVMe to chipset M.2 slot

In my case the NVMe drive needed to be installed in the **chipset M.2 slot** instead of the CPU M.2 slot.  

> ⚠️ **Warning:** The Intel Arc Pro B50 uses **PCI Express 5.0 x8**, so using it in a PCIe slot below version 3 may cause issues.

---

### 2. BIOS Settings

- Enable **4G Decoding**  
- Enable **Resizable BAR**  

After reboot, check if the GPU is detected:

```bash
lspci -nnk | grep -iA3 vga
```

### 3. Install the Firmware

```bash
apt update
apt install firmware-intel-graphics
```

### 4. Upgrade to Kernel 6.16+
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

### 5. Install oneAPI

```bash
wget https://registrationcenter-download.intel.com/akdlm/IRC_NAS/bd1d0273-a931-4f7e-ab76-6a2a67d646c7/intel-oneapi-base-toolkit-2025.2.0.592_offline.sh
sudo sh ./intel-oneapi-base-toolkit-2025.2.0.592_offline.sh -a --silent --eula accept
```

> ⚠️ After installing the new kernel or oneAPI toolkit, a system reboot is recommended.

---
## Ollama (inside a Ubuntu 24.04 Docker-Container)
I was not able to run Ollama on Debian 13 with the Intel Arc Pro B50 due to no drivers from Intel. 

### Installation inside a default Ubuntu 24.04 Container

Run a Ubuntu 24.04 Container with the following command after installing docker:
```bash
docker run -it --name intel-gpu   --device=/dev/dri:/dev/dri   ubuntu:24.04 bash
```

After you are inside the Container run the following in order to setup Ollama:
```bash
ls /dev/dri/
apt-get update
apt-get install -y software-properties-common
add-apt-repository -y ppa:kobuk-team/intel-graphics
apt-get install -y libze-intel-gpu1 libze1 intel-metrics-discovery intel-opencl-icd clinfo intel-gsc
apt-get install -y libze-dev intel-ocloc
export OLLAMA_INTEL_GPU=true
cd /tmp/
apt install wget
wget https://github.com/ipex-llm/ipex-llm/releases/download/v2.3.0-nightly/ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz
tar -xvf  ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz 
cd ollama-ipex-llm-2.3.0b20250630-ubuntu
./start-ollama.sh 
apt install screen
screen
./ollama run <llm-model>
```
---

<p align="center">
  <img src="ressources/nvtop.jpg" width="500" alt="Intel Arc Pro B50 in nvtop">
  <br>
  <sub><em>Intel Arc Pro B50 in nvtop when running deepseek-r1:14b</em></sub>
</p>

---

### Models that I tried with Ollama

| Model | Working? |
|------------|-------------|
| **llama2:latest** | ❌ |
| **deepseek-r1:14b** | ✅ |

---

### Setup a prepared Compose file which is running Ollama on 0.0.0.0
This guide explains how to run **Ollama IPEX-LLM** with an **Intel Arc Pro B50** GPU inside a Docker container on Debian 13.

Required files:
- [Dockerfile](Docker/Dockerfile) – Basis-Image und Installation von Ollama IPEX-LLM
- [docker-compose.yml](Docker/docker-compose.yml) – Startet den Container mit GPU-Zugriff

Clone the repository:
```bash
git clone git@github.com:justin1703/intel-arc-pro-b50-debian13.git
cd intel-arc-pro-b50-debian13/Docker
```

Build the Dockerfile:
```bash
docker compose build
```

Run the Container:
```bash
docker compose up -d
```

You can try if its working by either checking http://<HOST_IP>::11434 or you can run this command:
```bash
docker exec -it ollama ./ollama list
```
