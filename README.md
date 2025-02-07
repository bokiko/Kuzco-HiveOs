
# Kuzco Docker Setup for HiveOS

Welcome to the Kuzco Docker Setup guide for HiveOS. This guide explains how to deploy your Kuzco Worker on HiveOS using Docker. If you need further help, please join our Discord community.

---

## Requirements

- **HiveOS:** Updated to the latest version
- **GPU:** An NVIDIA GPU from our [Supported Hardware List](https://docs.kuzco.xyz/hardware)
- **Docker:** Installed and running (HiveOS includes Docker by default)
- **NVIDIA Container Toolkit:** Installed
- **Kuzco Account:** Registered at [kuzco.xyz/register](https://kuzco.xyz/register) with your Worker ID and Code

---

## Setting Up Your Environment on HiveOS

### 1. Update Your System and Install Basic Packages

Even though HiveOS comes with many utilities, you may want to ensure your system is updated and that you have a simple text editor like **nano** available:
```bash
sudo apt update
sudo apt install -y nano curl wget
```

### 2. Verify Docker Installation

HiveOS typically includes Docker. Confirm it’s running with:
```bash
docker ps
```
If Docker isn’t running, start it with:
```bash
sudo systemctl start docker
```

### 3. Install or Verify the NVIDIA Container Toolkit

Ensure your system can access your NVIDIA GPU through Docker:

1. **Remove any old or conflicting repositories:**
   ```bash
   grep -r "nvidia.github.io" /etc/apt/sources.list* 2>/dev/null
   sudo rm /etc/apt/sources.list.d/nvidia-container-toolkit.list 2>/dev/null
   sudo rm /etc/apt/sources.list.d/libnvidia-container.list 2>/dev/null
   ```
2. **Add the NVIDIA Repository and GPG Key:**
   ```bash
   distribution=$(. /etc/os-release; echo $ID$VERSION_ID)
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
       | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

   curl -s -L https://nvidia.github.io/libnvidia-container/stable/$distribution/nvidia-container-toolkit.list \
       | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
       | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```
3. **Install the NVIDIA Container Toolkit:**
   ```bash
   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit
   ```
4. **Configure Docker to use the NVIDIA runtime:**
   ```bash
   sudo nvidia-ctk runtime configure --runtime=docker
   sudo systemctl restart docker
   ```
5. **Test GPU visibility:**
   ```bash
   docker run --rm --gpus all nvidia/cuda:11.0.3-base nvidia-smi
   ```
   If your GPU details appear, the setup is successful.

### 4. Create Your Kuzco Account and Get Your Credentials

1. Visit [kuzco.xyz/register](https://kuzco.xyz/register) to create and verify your account.
2. Connect your Discord account.
3. In the "Workers" tab on your dashboard, click **Create Worker**.
4. Save your **Worker ID** and **Code** for later use.

---

## Launching Your Kuzco Worker with Docker

### Single GPU Setup

To run your worker on a single GPU, use the following command (replace `YOUR_WORKER_ID` and `YOUR_CODE` with your actual credentials):
```bash
docker run --restart=always --rm --runtime=nvidia --gpus all \
  -e CACHE_DIRECTORY=/root/models \
  -v ~/.kuzco/models:/root/models \
  kuzcoxyz/amd64-ollama-nvidia-worker \
  --worker YOUR_WORKER_ID --code YOUR_CODE
```
The `--restart=always` flag ensures your container will automatically restart if it crashes.

### Multiple GPU Setup Using a Script

For systems with multiple GPUs, a startup script makes management and future upgrades much easier. Create a script named `run_kuzco.sh`:
```bash
nano run_kuzco.sh
```
Paste the following example into the file—update the GPU device numbers and add a block for each GPU (e.g., `gpu0`, `gpu1`, etc.) that your rig has:
```bash
#!/bin/bash

# Start Worker for GPU 0 (named gpu0)
docker run -d --name gpu0 \
  --restart=always --runtime=nvidia --gpus "device=0" \
  -e CACHE_DIRECTORY=/root/models \
  -v ~/.kuzco/models:/root/models \
  kuzcoxyz/amd64-ollama-nvidia-worker \
  --worker YOUR_WORKER_ID --code YOUR_CODE

# Wait 3 minutes before starting the next container
sleep 180

# Start Worker for GPU 1 (named gpu1)
docker run -d --name gpu1 \
  --restart=always --runtime=nvidia --gpus "device=1" \
  -e CACHE_DIRECTORY=/root/models \
  -v ~/.kuzco/models:/root/models \
  kuzcoxyz/amd64-ollama-nvidia-worker \
  --worker YOUR_WORKER_ID --code YOUR_CODE

# Repeat similar blocks for additional GPUs (e.g., gpu2, gpu3, etc.) as needed.
```
Make the script executable and run it:
```bash
chmod +x run_kuzco.sh
./run_kuzco.sh
```

---

## Useful Docker Commands

- **List running containers:**
  ```bash
  docker ps
  ```
- **View logs of a container:**
  ```bash
  docker logs -f <container_name_or_ID>
  ```
- **Stop a container:**
  ```bash
  docker stop <container_name_or_ID>
  ```
- **Remove a container:**
  ```bash
  docker rm <container_name_or_ID>
  ```

---

## Upgrading Your Worker

Before upgrading, see which containers are running:
```bash
docker ps
```
Then, stop and remove the existing containers:
```bash
docker stop gpu0 gpu1   # Include all container names as listed by docker ps
docker rm gpu0 gpu1
```
Pull the latest worker image:
```bash
docker pull kuzcoxyz/amd64-ollama-nvidia-worker
```
Finally, restart your containers using your script:
```bash
./run_kuzco.sh
```
*Note: Upgrade Docker itself only if you encounter Docker-specific issues.*

---

## Deprecation Notice

As of December 2024, the old `kuzcoxyz/worker` image is deprecated. Always use the `kuzcoxyz/amd64-ollama-nvidia-worker` image.

---

This guide provides a simple, step-by-step approach to setting up your Kuzco Worker on HiveOS using Docker. If you run into any issues, refer to the Useful Docker Commands section or join our Discord community for assistance.

<div align="center">
  <i>Made with ❤️ by <a href="https://github.com/bokiko">bokiko</a></i>
</div>
```

