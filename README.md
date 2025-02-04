# Kuzco Worker Setup for Mining Rigs using HiveOs

<div align="center">

<img src="https://avatars.githubusercontent.com/u/125929854?s=200&v=4" alt="Kuzco Logo" width="200"/>

Setup guide for running Kuzco workers on mining rigs with NVIDIA GPUs.

[![Discord](https://img.shields.io/discord/YOUR_DISCORD_ID)](https://discord.gg/kuzco)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/docs-kuzco-blue)](https://docs.kuzco.xyz/)
[![GitHub](https://img.shields.io/badge/github-context--labs-black)](https://github.com/context-labs)

</div>

## ⚠️ Safety Requirements

Before proceeding with installation, ensure:

- [ ] All mining operations are stopped
- [ ] GPUs are at idle state
- [ ] System is stable and updated
- [ ] Adequate cooling is available
- [ ] Power supply can handle peak loads

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Installation](#installation)
- [Worker Setup](#worker-setup)
- [Multi-GPU Configuration](#multi-gpu-configuration)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)
- [Mining Compatibility](#mining-compatibility)

## Hardware Requirements

### Supported GPUs:
- NVIDIA RTX 4090
- NVIDIA RTX 4080
- NVIDIA RTX 3090 Ti
- NVIDIA RTX 3090
- NVIDIA RTX 3080 Ti
- NVIDIA RTX 3080
- NVIDIA A100

For detailed hardware specifications and requirements, visit the [official hardware documentation](https://docs.kuzco.xyz/hardware).

Additional Requirements:
- HiveOS or Linux-based mining OS
- Docker support
- SSH access
- Stable internet connection
- At least 16GB system RAM
- Sufficient power supply for full GPU utilization
- Update both HiveOs and Nvidia Drivers before begining

## Installation

### 1. Docker Setup
```bash
# Update system
sudo apt update

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### 2. NVIDIA Container Toolkit
```bash
# Add NVIDIA repository
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Install toolkit
sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

## Worker Setup

### 1. Account Creation
1. Register at [kuzco.xyz/register](https://kuzco.xyz/register)
2. Verify email
3. Connect Discord account
4. Create worker in dashboard

### 2. Single GPU Setup
```bash
# Create tmux session
tmux new -s kuzco0

# Run worker (replace credentials)
docker run --restart=always --runtime=nvidia --gpus "device=0" \
    -e CACHE_DIRECTORY=/root/models \
    -v ~/.kuzco/models:/root/models \
    kuzcoxyz/amd64-ollama-nvidia-worker \
    --worker YOUR_WORKER_ID \
    --code YOUR_CODE

# Detach from tmux: Ctrl+B, then D
```

## Multi-GPU Configuration

For additional GPUs:

1. Create new worker on dashboard
2. Launch new tmux session:
```bash
tmux new -s kuzco1  # Increment for each GPU
```

3. Run worker with modified device number:
```bash
docker run --restart=always --runtime=nvidia --gpus "device=1" \
    -e CACHE_DIRECTORY=/root/models \
    -v ~/.kuzco/models:/root/models \
    kuzcoxyz/amd64-ollama-nvidia-worker \
    --worker YOUR_WORKER_ID \
    --code YOUR_CODE
```

## Monitoring

### Commands
```bash
# List tmux sessions
tmux ls

# Check containers
docker ps

# GPU status
nvidia-smi

# Attach to session
tmux attach -t kuzco0
```

### Health Checks
- Monitor temperatures
- Watch power consumption
- Check worker status in dashboard
- Verify system stability

## Mining Compatibility

### ⚠️ Important Notes
- Not recommended with KawPoW or intensive algorithms
- Monitor power consumption closely
- Ensure adequate cooling
- Start with lighter algorithms if mining
- Consider reducing mining intensity
- Test stability before running alongside mining operations

## Troubleshooting

### Common Issues

1. Docker fails to start:
```bash
sudo systemctl status docker
```

2. GPU not detected:
```bash
nvidia-smi
```

3. Container crashes:
```bash
docker logs $(docker ps -q --filter ancestor=kuzcoxyz/amd64-ollama-nvidia-worker)
```

## Support

- Join [Discord](https://discord.gg/kuzco) for support
- Check [Official Documentation](https://docs.kuzco.xyz)
- Visit [GitHub Repository](https://github.com/context-labs)
- Report issues in [GitHub Issues](issues)

## Disclaimer

Use at your own risk. Monitor hardware closely. Authors not responsible for damage.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">
Made with ❤️ by the Mining Community
</div>
