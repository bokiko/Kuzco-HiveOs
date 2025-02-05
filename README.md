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
- 16GB RAM minimum
- All mining operations stopped
- Stable internet connection

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

# Install tmux if not already installed
sudo apt install -y tmux
```

### 2. Install Docker
```bash
# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### 3. Create Kuzco Account
1. Visit [kuzco.xyz/register](https://kuzco.xyz/register)
2. Create your account and verify email
3. Connect your Discord account
4. Navigate to "Workers" tab
5. Click "Create Worker" and save your worker ID and code

### 4. Start Your Worker

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

---

## 📊 Monitoring Commands

### Basic Commands
```bash
# List tmux sessions
tmux ls

# List running containers
docker ps

# Check GPU status
nvidia-smi

# Reattach to tmux session
tmux attach -t kuzco0
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
        <i>Made with ❤️ by [bokiko](https://github.com/bokiko) </i>
</div>
