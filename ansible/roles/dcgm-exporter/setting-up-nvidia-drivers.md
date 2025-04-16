
# Setting Up NVIDIA GPU with Docker on Bare Metal

This guide walks through setting up an **NVIDIA GPU** on a **bare-metal** server and configuring Docker to run GPU-based containers, such as `dcgm-exporter`.

---

## 1. **Install NVIDIA Driver**

### 1.1. **Remove any existing NVIDIA drivers**
If any NVIDIA drivers are already installed, remove them first:
```bash
sudo apt purge nvidia-driver*
sudo apt autoremove
```

### 1.2. **Install recommended NVIDIA drivers**
Install the NVIDIA driver recommended for your GPU:
```bash
sudo apt update
sudo ubuntu-drivers autoinstall
```
This will automatically detect your GPU and install the appropriate driver.

### 1.3. **Reboot the system**
After installing the drivers, reboot your system to apply the changes:
```bash
sudo reboot
```

### 1.4. **Verify the installation**
After reboot, verify the driver is installed correctly:
```bash
nvidia-smi
```
You should see the GPU information in a table format. If you see any errors, refer to the troubleshooting section.

---

## 2. **Install NVIDIA Docker Toolkit**

### 2.1. **Install the NVIDIA container toolkit**
Install the NVIDIA container toolkit to enable GPU support for Docker containers:
```bash
sudo apt update
sudo apt install -y nvidia-container-toolkit
```

### 2.2. **Configure Docker to use NVIDIA runtime**
Configure Docker to use the `nvidia` runtime by running the following:
```bash
sudo nvidia-ctk runtime configure --runtime=docker
```

This command automatically adds the necessary configuration to Docker's settings.

### 2.3. **Restart Docker service**
Restart Docker for the changes to take effect:
```bash
sudo systemctl restart docker
```

---

## 3. **Test the NVIDIA Docker Setup**

### 3.1. **Test GPU access in Docker**
Run the following test to check if Docker can access the GPU:
```bash
docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu20.04 nvidia-smi
```
This should return GPU information similar to the `nvidia-smi` command.

---

## 4. **Configure Docker Compose with GPU Support**

### 4.1. **Set up Docker Compose with `dcgm-exporter`**

Here’s a Docker Compose configuration example to run `dcgm-exporter` using the NVIDIA GPU:
```yaml
version: "3.8"

services:
  dcgm-exporter:
    image: nvcr.io/nvidia/k8s/dcgm-exporter:2.1.4-2.3.1-ubuntu20.04
    restart: unless-stopped
    runtime: nvidia
    network_mode: host
    cap_add:
      - SYS_ADMIN
    command: ["-f", "/etc/dcgm-exporter/dcp-metrics-included.csv"]
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
```

### 4.2. **Start the service with Docker Compose**
Once the `docker-compose.yml` file is set up, start the service with:
```bash
docker-compose up -d
```

---

## Troubleshooting

If you encounter the error:
```
Error response from daemon: unknown or invalid runtime name: nvidia
```

It means that Docker is not aware of the NVIDIA runtime. To resolve this:

1. Ensure the NVIDIA container toolkit is installed.
2. Configure Docker to use the NVIDIA runtime:
   ```bash
   sudo nvidia-ctk runtime configure --runtime=docker
   ```
3. Restart the Docker service:
   ```bash
   sudo systemctl restart docker
   ```

---

## Additional Notes

- **Secure Boot**: If you are running with Secure Boot enabled, the NVIDIA drivers may fail to load. Consider disabling Secure Boot in your BIOS/UEFI.
- **VMs**: If running on a virtual machine, ensure that GPU passthrough is properly configured.
- **Ubuntu Versions**: These steps are for Ubuntu-based systems. Ensure you have the appropriate versions for your specific system.
