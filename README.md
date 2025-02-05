# Kuzco Worker Setup for HiveOS Mining Rigs

<div align="center">
<img src="https://avatars.githubusercontent.com/u/125929854?s=200&v=4" alt="Kuzco Logo" width="200"/>

### Setup guide for running Kuzco workers on HiveOS mining rigs with NVIDIA GPUs

[![Documentation](https://img.shields.io/badge/docs-kuzco-blue)](https://docs.kuzco.xyz/)
[![GitHub](https://img.shields.io/badge/github-context--labs-black)](https://github.com/context-labs)

</div>

---

## 📖 Introduction

### What is Kuzco?

Kuzco Testnet is a distributed network where GPU owners can contribute their computing power to run AI inference tasks for large language models like Llama3, Mistral, and others. As a testnet operator, you can earn $KZO points by connecting your GPUs to the network. The earning potential scales with the number of GPUs you contribute.

### ⚠️ Important Note

Kuzco Testnet is in early development. Expect occasional bugs and downtime periods. The team actively communicates updates through the Operator Discord. While not yet production-ready, the network is continuously being improved and enhanced.

For bug reports or issues, please open a ticket in Discord with detailed information. Stay updated about network status through the Discord announcements channel.

---

## 🔍 Requirements

### System Requirements
- HiveOS updated to latest version
- NVIDIA drivers updated
- 16GB RAM minimum (required for multiple GPUs)
- All mining operations stopped
- Stable internet connection
- Docker installed and running
- NVIDIA Container Toolkit

### Supported GPUs
- NVIDIA RTX 4090
- NVIDIA RTX 4080
- NVIDIA RTX 3090 Ti
- NVIDIA RTX 3090
- NVIDIA RTX 3080 Ti
- NVIDIA RTX 3080
- NVIDIA A100

For detailed specifications, visit [hardware requirements](https://docs.kuzco.xyz/hardware).

---

## 🛠️ Installation Guide

### 1. Update System & Install Required Packages
```bash
# Update system
sudo apt update

# Install tmux and required packages
sudo apt install -y tmux curl wget
```

### 2. Install Docker
```bash
# Remove old Docker installations if present
sudo apt-get remove docker docker-engine docker.io containerd runc

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### 3. Install NVIDIA Container Toolkit
```bash
# Add NVIDIA repository and GPG key
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Install toolkit
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# Configure Docker to use NVIDIA runtime
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Test NVIDIA Docker setup
docker run --rm --gpus all nvidia/cuda:11.0.3-base nvidia-smi
```

### 4. Configure iptables (if needed)
If you encounter Docker network errors:
```bash
# Switch to iptables-legacy
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo systemctl restart docker
```

### 5. Create Kuzco Account
1. Visit [kuzco.xyz/register](https://kuzco.xyz/register)
2. Create your account and verify email
3. Connect your Discord account
4. Navigate to "Workers" tab
5. Click "Create Worker" and save your worker ID and code

### 6. Start Your Worker

Create a new tmux session:
```bash
tmux new -s kuzco0
```

Inside the tmux session, run your worker:
```bash
docker run --restart=always --runtime=nvidia --gpus "device=0" \
    -e CACHE_DIRECTORY=/root/models \
    -v ~/.kuzco/models:/root/models \
    kuzcoxyz/amd64-ollama-nvidia-worker \
    --worker YOUR_WORKER_ID \
    --code YOUR_CODE
```
Replace `YOUR_WORKER_ID` and `YOUR_CODE` with your credentials.

To detach from tmux: Press `Ctrl+B`, then `D`

---

## 🔄 Multiple GPU Setup

For each additional GPU:

1. Create new worker on Kuzco website
2. Create new tmux session:
```bash
tmux new -s kuzco1  # Use kuzco2 for third GPU, etc.
```
3. Run worker with different device number:
```bash
docker run --restart=always --runtime=nvidia --gpus "device=1" \
    -e CACHE_DIRECTORY=/root/models \
    -v ~/.kuzco/models:/root/models \
    kuzcoxyz/amd64-ollama-nvidia-worker \
    --worker YOUR_WORKER_ID \
    --code YOUR_CODE
```
Note: Change `device=1` to `device=2` for third GPU, etc.

---

## 📊 Monitoring and Troubleshooting

### Basic Commands
```bash
# List tmux sessions
tmux ls

# List running containers
docker ps

# Check GPU status
nvidia-smi

# Check container logs
docker logs $(docker ps | grep kuzco | awk '{print $1}')

# Reattach to tmux session
tmux attach -t kuzco0
```

### Common Issues and Solutions

1. Docker service fails to start:
```bash
sudo systemctl status docker
journalctl -xeu docker.service
```

2. "Unknown runtime: nvidia" error:
- Follow NVIDIA Container Toolkit installation steps
- Verify installation with test command

3. Duplicate tmux session:
```bash
# Kill existing session
tmux kill-session -t kuzco0
# Create new session
tmux new -s kuzco0
```

---

## ⚡ Mining Compatibility

### Important Guidelines
- Wait for worker initialization (up to 10 minutes)
- Monitor temperatures closely
- Not recommended with KawPoW or intensive algorithms
- Test stability before mining alongside workers
- Monitor power consumption carefully

### Best Practices
- Initialize one GPU at a time
- Monitor your rig's stability
- Keep track of power consumption
- Ensure proper cooling
- Back up your worker credentials

---

## 🔗 Support Resources
- Join [Discord](https://discord.gg/kuzco)
- Check [Documentation](https://docs.kuzco.xyz)
- Visit [GitHub](https://github.com/context-labs)

---

## ⚠️ Disclaimer
Use at your own risk. Monitor hardware closely.

---

<div align="center">
<i>Made with ❤️ by [bokiko](https://github.com/bokiko)</i>
</div>
