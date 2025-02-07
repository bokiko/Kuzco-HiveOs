
# Kuzco Worker Setup for HiveOS Mining Rigs

<div align="center">
  <img src="https://avatars.githubusercontent.com/u/125929854?s=200&v=4" alt="Kuzco Logo" width="200"/>

  ### A Step-by-Step Guide for Running Kuzco Workers on HiveOS-based Rigs with NVIDIA GPUs

  [![Documentation](https://img.shields.io/badge/docs-kuzco-blue)](https://docs.kuzco.xyz/)
  [![GitHub](https://img.shields.io/badge/github-context--labs-black)](https://github.com/context-labs)
</div>

---

## 📑 Table of Contents
- [Introduction](#introduction)
- [Requirements](#requirements)
- [Installation Guide](#installation-guide)
  - [1. Update System & Install Basic Packages](#1-update-system--install-basic-packages)
  - [2. Install Docker](#2-install-docker)
  - [3. Install NVIDIA Container Toolkit](#3-install-nvidia-container-toolkit)
  - [4. Configure iptables (if needed)](#4-configure-iptables-if-needed)
  - [5. Create Kuzco Account](#5-create-kuzco-account)
  - [6. Start Your First Worker](#6-start-your-first-worker)
- [Multiple GPU Setup Using a Script](#multiple-gpu-setup-using-a-script)
  - [Staggered Startup Script](#staggered-startup-script)
- [Monitoring & Troubleshooting](#monitoring--troubleshooting)
- [Upgrading / Updating Your Worker](#upgrading--updating-your-worker)
- [Mining Compatibility](#mining-compatibility)
- [FAQ](#faq)
- [Support Resources](#support-resources)
- [Disclaimer](#disclaimer)

---

## Introduction

### What is Kuzco?

Kuzco Testnet is a decentralized network where GPU owners can run AI inference tasks for large language models (e.g., Llama3, Mistral). As an operator, you earn **$KZO points** for each GPU you contribute, with earnings increasing as you add more GPUs.

### ⚠️ Important Note

Kuzco is in **early development**. Occasional bugs or downtime may occur. Join the official Discord for real-time updates. If you encounter issues, please open a ticket with detailed information and stay tuned for maintenance announcements or new releases.

---

## Requirements

### System Requirements
- **HiveOS** updated to the latest version.
- **NVIDIA drivers** up-to-date.
- **Minimum 16GB RAM** (especially for multiple GPUs).
- **No active mining** – stop all mining operations before proceeding.
- **Stable internet connection**.
- **Docker** installed and running.
- **NVIDIA Container Toolkit** installed.

### Supported GPUs
- NVIDIA RTX 4090
- NVIDIA RTX 4080
- NVIDIA RTX 3090 Ti
- NVIDIA RTX 3090
- NVIDIA RTX 3080 Ti
- NVIDIA RTX 3080
- NVIDIA A100

For more details, refer to the [Kuzco Hardware Requirements](https://docs.kuzco.xyz/hardware).

---

## Installation Guide

### 1. Update System & Install Basic Packages
Run the following commands to update your system and install essential utilities including **nano** (a simple text editor):
```bash
sudo apt update
sudo apt install -y tmux curl wget nano
```

### 2. Install Docker
To ensure a clean installation, remove any old versions of Docker before installing:
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### 3. Install NVIDIA Container Toolkit

> **Note**:  
> For Ubuntu 18.04 use `ubuntu18.04`, and for Ubuntu 22.04 use `ubuntu22.04` as the distribution value.

1. **Remove Old or Conflicting Repositories**:
   ```bash
   grep -r "nvidia.github.io" /etc/apt/sources.list* 2>/dev/null
   sudo rm /etc/apt/sources.list.d/nvidia-container-toolkit.list 2>/dev/null
   sudo rm /etc/apt/sources.list.d/libnvidia-container.list 2>/dev/null
   ```
2. **Add the NVIDIA Repository and GPG Key**:
   ```bash
   distribution=$(. /etc/os-release; echo $ID$VERSION_ID)
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
       | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

   curl -s -L https://nvidia.github.io/libnvidia-container/stable/$distribution/nvidia-container-toolkit.list \
       | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
       | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```
3. **Install the Toolkit**:
   ```bash
   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit
   ```
4. **Configure Docker to Use the NVIDIA Runtime**:
   ```bash
   sudo nvidia-ctk runtime configure --runtime=docker
   sudo systemctl restart docker
   ```
5. **Test the Setup**:
   ```bash
   docker run --rm --gpus all nvidia/cuda:11.0.3-base nvidia-smi
   ```
   If your GPU information is displayed, the configuration is successful.

### 4. Configure iptables (if needed)
If you encounter Docker networking issues, configure iptables to use the legacy version:
```bash
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo systemctl restart docker
```

### 5. Create Kuzco Account
1. Visit [kuzco.xyz/register](https://kuzco.xyz/register).  
2. Create and verify your account.  
3. Connect your Discord account.  
4. Click **Create Worker** in the **Workers** tab.  
5. Save your **Worker ID** and **Code** for later use.

### 6. Start Your First Worker
1. **Open a tmux Session** (optional – this is for a single GPU; however, for multiple GPUs, we recommend using the script method explained below):
   ```bash
   tmux new -s kuzco0
   ```
2. **Run the Worker Container** (replace `YOUR_WORKER_ID` and `YOUR_CODE` with your credentials):
   ```bash
   docker run --restart=always --runtime=nvidia --gpus "device=0" \
       -e CACHE_DIRECTORY=/root/models \
       -v ~/.kuzco/models:/root/models \
       kuzcoxyz/amd64-ollama-nvidia-worker \
       --worker YOUR_WORKER_ID \
       --code YOUR_CODE
   ```
3. **Detach from tmux** by pressing **Ctrl+B** then **D**.

---

## Multiple GPU Setup Using a Script

For systems with multiple GPUs, it is simpler and more maintainable to use a startup script. This method allows for easier future upgrades and ensures consistency across all GPU containers. In this approach, you define a container for each GPU you have (for example, name them `gpu0`, `gpu1`, etc.). Simply adjust the script to include as many containers as needed for your system.

### Staggered Startup Script

Using a script helps manage startup delays (to avoid power spikes) and makes upgrades easier. You can edit the script with **nano** if needed. To open the script for editing, use:
```bash
nano run_kuzco.sh
```

Below is an example script. **Ensure you add a block for each GPU you have (e.g., gpu0 for the first GPU, gpu1 for the second, etc.).**

```bash
#!/bin/bash

# Start Worker for GPU 0 (rename as gpu0)
docker run -d --name gpu0 \
  --restart=always --runtime=nvidia --gpus "device=0" \
  -e CACHE_DIRECTORY=/root/models \
  -v ~/.kuzco/models:/root/models \
  kuzcoxyz/amd64-ollama-nvidia-worker \
  --worker YOUR_WORKER_ID \
  --code YOUR_CODE

# Wait 3 minutes before starting the next container
sleep 180

# Start Worker for GPU 1 (rename as gpu1)
docker run -d --name gpu1 \
  --restart=always --runtime=nvidia --gpus "device=1" \
  -e CACHE_DIRECTORY=/root/models \
  -v ~/.kuzco/models:/root/models \
  kuzcoxyz/amd64-ollama-nvidia-worker \
  --worker YOUR_WORKER_ID \
  --code YOUR_CODE

# Repeat similar blocks for additional GPUs (e.g., gpu2, gpu3, etc.) as per the number of GPUs in your rig.
```

Make the script executable and then run it:
```bash
chmod +x run_kuzco.sh
./run_kuzco.sh
```

---

## Monitoring & Troubleshooting

**Basic Commands:**
```bash
# List running Docker containers
docker ps

# Check GPU status
nvidia-smi

# Follow logs for a specific container (replace <container_name_or_ID> with the actual name or ID)
docker logs -f <container_name_or_ID>
```

**Common Issues & Fixes:**
1. **Docker Fails to Start**  
   Check the Docker service status:
   ```bash
   sudo systemctl status docker
   journalctl -xeu docker.service
   ```
2. **"Unknown runtime: nvidia" Error**  
   - Verify the NVIDIA Container Toolkit installation.
   - Test GPU visibility inside a Docker container:
     ```bash
     docker run --rm --gpus all nvidia/cuda:11.0.3-base nvidia-smi
     ```
3. **Container Fails to Run or Exits Unexpectedly**  
   - Review the container logs:
     ```bash
     docker logs -f <container_name_or_ID>
     ```
   - Ensure that your Worker ID and Code are correctly specified.
4. **Network Issues or IP Conflicts**  
   - Confirm that iptables are set to legacy if you experience Docker networking issues:
     ```bash
     sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
     sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
     sudo systemctl restart docker
     ```

---

## Upgrading / Updating Your Worker

Before upgrading, check which containers are running:
```bash
docker ps
```
Then, follow these steps:

1. **Stop and Remove Existing Containers:**
   ```bash
   docker stop gpu0 gpu1   # Include all container names as listed by docker ps
   docker rm gpu0 gpu1     # Remove them after stopping
   ```
2. **Pull the Latest Worker Image:**
   ```bash
   docker pull kuzcoxyz/amd64-ollama-nvidia-worker
   ```
3. **Restart Your Containers Using the Script:**
   ```bash
   ./run_kuzco.sh
   ```
*Note: Only upgrade Docker if there’s a Docker-specific issue or version mismatch.*

---

## Mining Compatibility

- Allow up to **10 minutes** for the worker initialization.
- Monitor GPU **temperatures** closely.
- Initially, avoid heavy mining workloads (e.g., KawPoW) to ensure system stability.
- Check power consumption and overall system stability before scaling up.

---

## FAQ

**Q: What if my GPU isn’t recognized in Docker?**  
*A:* Verify the NVIDIA Container Toolkit installation and run:
```bash
docker run --rm --gpus all nvidia/cuda:11.0.3-base nvidia-smi
```

**Q: Can I run multiple workers with the same Worker ID?**  
*A:* Yes. You can use the same Worker ID/Code for multiple GPUs. To check your running containers, run:
```bash
docker ps
```

**Q: How do I view logs if something goes wrong?**  
*A:* Use the following command (replace `<container_name_or_ID>` with the actual value):
```bash
docker logs -f <container_name_or_ID>
```

---

## Support Resources

- [Discord Community](https://discord.gg/kuzco)  
- [Kuzco Documentation](https://docs.kuzco.xyz)  
- [GitHub: context-labs](https://github.com/context-labs)

---

## Disclaimer

Use this guide at your own risk. Always monitor your hardware (temperatures, power consumption, and stability) during operations.

<div align="center">
  <i>Made with ❤️ by <a href="https://github.com/bokiko">bokiko</a></i>
</div>
```



