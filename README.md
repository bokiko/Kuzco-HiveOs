

# Kuzco Worker Setup for HiveOS Mining Rigs

<div align="center">
  <img src="https://avatars.githubusercontent.com/u/125929854?s=200&v=4" alt="Kuzco Logo" width="200"/>

  ### A step-by-step guide for running Kuzco workers on HiveOS-based rigs with NVIDIA GPUs

  [![Documentation](https://img.shields.io/badge/docs-kuzco-blue)](https://docs.kuzco.xyz/)
  [![GitHub](https://img.shields.io/badge/github-context--labs-black)](https://github.com/context-labs)
</div>

---

## 📖 Introduction

### What is Kuzco?

Kuzco Testnet is a decentralized network enabling GPU owners to run AI inference tasks for large language models (e.g., Llama3, Mistral). As an operator, you earn **$KZO points** for each GPU you contribute, with earnings scaling up per GPU.

### ⚠️ Important Note

Kuzco is in early development and may experience occasional bugs or downtime. The team provides real-time updates via the Operator Discord. Please submit detailed bug reports or issues through Discord’s ticket system and follow announcements there for any changes in network status.

---

## 🔍 Requirements

### System Requirements
- **HiveOS** updated to the latest version
- **NVIDIA drivers** updated
- **At least 16GB RAM** (necessary for multiple GPUs)
- **No active mining** (stop all mining operations)
- **Stable internet connection**
- **Docker** installed and running
- **NVIDIA Container Toolkit** installed

### Supported GPUs
- NVIDIA RTX 4090
- NVIDIA RTX 4080
- NVIDIA RTX 3090 Ti
- NVIDIA RTX 3090
- NVIDIA RTX 3080 Ti
- NVIDIA RTX 3080
- NVIDIA A100

Refer to [Kuzco hardware requirements](https://docs.kuzco.xyz/hardware) for detailed specs.

---

## 🛠️ Installation Guide

### 1. Update System & Install Basic Packages
```bash
# Update packages
sudo apt update

# Install tmux, curl, wget
sudo apt install -y tmux curl wget
```

### 2. Install Docker
```bash
# Remove older Docker versions if they exist
sudo apt-get remove docker docker-engine docker.io containerd runc

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### 3. Install NVIDIA Container Toolkit

**Note**: These steps assume HiveOS is based on Ubuntu 18.04 or similar. If you have a different base version, adjust `$distribution` accordingly.

1. **Remove old or conflicting NVIDIA repository entries:**
   ```bash
   grep -r "nvidia.github.io" /etc/apt/sources.list* 2>/dev/null

   # Remove any found entries (filenames may vary)
   sudo rm /etc/apt/sources.list.d/nvidia-container-toolkit.list 2>/dev/null
   sudo rm /etc/apt/sources.list.d/libnvidia-container.list 2>/dev/null
   ```

2. **Add the official NVIDIA repository and GPG key:**
   ```bash
   distribution=$(. /etc/os-release; echo $ID$VERSION_ID)

   # Add NVIDIA's GPG key
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
       | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

   # Add the stable repository for your distribution
   curl -s -L https://nvidia.github.io/libnvidia-container/stable/$distribution/nvidia-container-toolkit.list \
       | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
       | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```

3. **Install the toolkit:**
   ```bash
   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit
   ```

4. **Configure Docker to use NVIDIA runtime:**
   ```bash
   sudo nvidia-ctk runtime configure --runtime=docker
   sudo systemctl restart docker
   ```

5. **Test NVIDIA Docker:**
   ```bash
   docker run --rm --gpus all nvidia/cuda:11.0.3-base nvidia-smi
   ```
   If you see your GPU info, the setup was successful.

### 4. Configure iptables (if needed)

If you encounter Docker network issues on HiveOS, switch to iptables-legacy:
```bash
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo systemctl restart docker
```

### 5. Create Kuzco Account
1. Go to [kuzco.xyz/register](https://kuzco.xyz/register)
2. Sign up and verify your email
3. Connect your Discord account
4. In the "Workers" tab, click **Create Worker**
5. Save your **Worker ID** and **Code** (you’ll need them to start the worker)

### 6. Start Your Worker

Open a **tmux** session (so the worker keeps running even if you disconnect):
```bash
tmux new -s kuzco0
```

Run the Docker container within tmux:
```bash
docker run --restart=always --runtime=nvidia --gpus "device=0" \
    -e CACHE_DIRECTORY=/root/models \
    -v ~/.kuzco/models:/root/models \
    kuzcoxyz/amd64-ollama-nvidia-worker \
    --worker YOUR_WORKER_ID \
    --code YOUR_CODE
```
Replace `YOUR_WORKER_ID` and `YOUR_CODE` with your actual credentials.

To detach from the tmux session, press **Ctrl+B** then **D**.

---

## 🔄 Multiple GPU Setup

Each GPU is run by a separate worker. Repeat for every GPU:

1. **Create a new worker** in Kuzco’s dashboard.
2. **Open a new tmux session**:
   ```bash
   tmux new -s kuzco1
   ```
3. **Run the container** with the new worker ID and code, assigning `device=1`:
   ```bash
   docker run --restart=always --runtime=nvidia --gpus "device=1" \
       -e CACHE_DIRECTORY=/root/models \
       -v ~/.kuzco/models:/root/models \
       kuzcoxyz/amd64-ollama-nvidia-worker \
       --worker YOUR_WORKER_ID \
       --code YOUR_CODE
   ```
   - For a third GPU, use `device=2`, and so on.

---

## 📊 Monitoring & Troubleshooting

### Basic Commands
```bash
# List tmux sessions
tmux ls

# List running containers
docker ps

# Check GPU status
nvidia-smi

# View container logs (first running kuzco container)
docker logs $(docker ps | grep kuzco | awk '{print $1}')

# Reattach to a tmux session
tmux attach -t kuzco0
```

### Common Issues & Solutions

1. **Docker service fails to start**  
   ```bash
   sudo systemctl status docker
   journalctl -xeu docker.service
   ```
   Resolve any reported errors.

2. **"Unknown runtime: nvidia" error**  
   - Verify you followed the **NVIDIA Container Toolkit** install and config steps.
   - Confirm `docker run --rm --gpus all nvidia/cuda:11.0.3-base nvidia-smi` works.

3. **Duplicate tmux session**  
   ```bash
   # Kill the existing session
   tmux kill-session -t kuzco0
   
   # Create a new session
   tmux new -s kuzco0
   ```

---

## ⚡ Mining Compatibility

### Guidelines
- Wait up to **10 minutes** for worker initialization
- Monitor your GPU temperatures
- Avoid combining this with heavy mining algorithms (e.g., KawPoW) at first
- Test stability before running multiple workloads
- Keep an eye on power consumption

### Best Practices
- Start one GPU at a time
- Maintain strong cooling
- Back up your worker credentials
- Continuously monitor system stability

---

## 🔗 Support Resources
- [Discord Community](https://discord.gg/kuzco)
- [Kuzco Documentation](https://docs.kuzco.xyz)
- [GitHub: context-labs](https://github.com/context-labs)

---

## ⚠️ Disclaimer
Use this guide at your own risk. Monitor hardware temperatures and stability throughout the process.

<div align="center">
  <i>Made with ❤️ by <a href="https://github.com/bokiko">bokiko</a></i>
</div>
