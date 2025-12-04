# Photopia Capture Tower - Mini PC Setup Guide

Complete setup guide for NiPoGi Pinova P1 Mini PC (AMD Ryzen 4300U, Windows 11 Pro)

## Overview

This guide will walk you through setting up the mini PC as a capture tower, including:
- WSL2 (Windows Subsystem for Linux) installation
- Python environment setup
- Camera (gphoto2) configuration
- Application deployment
- Auto-start configuration

**Estimated Time**: 60-90 minutes

---

## Table of Contents

1. [Windows 11 Initial Setup](#1-windows-11-initial-setup)
2. [WSL2 Installation](#2-wsl2-installation)
3. [Ubuntu Setup in WSL2](#3-ubuntu-setup-in-wsl2)
4. [System Dependencies Installation](#4-system-dependencies-installation)
5. [Python Environment Setup](#5-python-environment-setup)
6. [USB Camera Configuration](#6-usb-camera-configuration)
7. [Photopia Application Setup](#7-photopia-application-setup)
8. [Machine Registration](#8-machine-registration)
9. [Testing the System](#9-testing-the-system)
10. [Auto-start Configuration](#10-auto-start-configuration)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Windows 11 Initial Setup

### 1.1 Basic Configuration

1. **Complete Windows 11 setup wizard**
   - Choose language, region, and keyboard
   - Sign in with Microsoft account (or create local account)
   - Skip unnecessary setup options

2. **Update Windows**
   ```
   Settings → Windows Update → Check for updates
   ```
   - Install all available updates
   - Restart if required

3. **Disable Sleep Mode (Important for continuous operation)**
   ```
   Settings → System → Power → Screen and sleep
   - When plugged in, turn off after: Never
   - When plugged in, put my device to sleep after: Never
   ```

4. **Install Windows Terminal (Recommended)**
   - Open Microsoft Store
   - Search for "Windows Terminal"
   - Install (provides better terminal experience)

---

## 2. WSL2 Installation

### 2.1 Enable WSL2

1. **Open PowerShell as Administrator**
   - Right-click Start menu → Terminal (Admin) or PowerShell (Admin)

2. **Install WSL2**
   ```powershell
   wsl --install
   ```

   This command will:
   - Enable WSL feature
   - Enable Virtual Machine Platform
   - Download and install Ubuntu (default distribution)
   - Set WSL2 as default version

3. **Restart your computer**
   ```powershell
   Restart-Computer
   ```

### 2.2 Verify WSL2 Installation

After restart:

1. **Open PowerShell and verify WSL version**
   ```powershell
   wsl --list --verbose
   ```

   Expected output:
   ```
   NAME      STATE           VERSION
   * Ubuntu    Running         2
   ```

2. **If Ubuntu is not installed, install it manually**
   ```powershell
   wsl --install -d Ubuntu
   ```

---

## 3. Ubuntu Setup in WSL2

### 3.1 First Launch

1. **Launch Ubuntu from Start Menu** or run `wsl` in PowerShell

2. **Create Unix username and password**
   - Username: `photopia` (or your choice)
   - Password: (choose a secure password - you'll need this for sudo commands)

### 3.2 Update Ubuntu System

```bash
# Update package lists
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install essential build tools
sudo apt install -y build-essential
```

---

## 4. System Dependencies Installation

### 4.1 Install gphoto2 (Critical for Camera Control)

```bash
# Install gphoto2 and libgphoto2
sudo apt install -y gphoto2 libgphoto2-dev

# Verify installation
gphoto2 --version
```

Expected output:
```
gphoto2 2.5.x
...
```

### 4.2 Install Python 3 and Development Tools

```bash
# Install Python 3.10+ and pip
sudo apt install -y python3 python3-pip python3-venv python3-dev

# Verify Python installation
python3 --version

# Upgrade pip
python3 -m pip install --upgrade pip
```

### 4.3 Install Image Processing Libraries

```bash
# Install system libraries required by PIL, OpenCV, etc.
sudo apt install -y \
    libjpeg-dev \
    libpng-dev \
    libtiff-dev \
    libavcodec-dev \
    libavformat-dev \
    libswscale-dev \
    libv4l-dev \
    libxvidcore-dev \
    libx264-dev \
    libgtk-3-dev \
    libatlas-base-dev \
    gfortran \
    libopenblas-dev \
    liblapack-dev
```

### 4.4 Install Git

```bash
sudo apt install -y git
```

---

## 5. Python Environment Setup

### 5.1 Create Project Directory

```bash
# Create workspace directory
mkdir -p ~/photopia
cd ~/photopia
```

### 5.2 Clone/Copy Photopia Repository

**Option A: If using Git**
```bash
git clone https://github.com/Auxf/Photopia.git
cd Photopia/capture-tower
```

**Option B: If copying files manually**
- Copy the `capture-tower` folder from the repository to `~/photopia/capture-tower`

```bash
cd ~/photopia/capture-tower
```

### 5.3 Create Virtual Environment (Recommended)

```bash
# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Your prompt should now show (venv)
```

### 5.4 Install Python Dependencies

```bash
# Install all required packages
pip install -r requirements.txt
```

This will install approximately 99 packages including:
- Flask (web framework)
- Pillow (image processing)
- OpenCV (computer vision)
- Supabase client
- And many others

**Note**: Installation may take 10-15 minutes depending on internet speed.

### 5.5 Verify Installation

```bash
# Test import of critical packages
python3 -c "import flask; import PIL; import cv2; import supabase; print('✅ All packages imported successfully')"
```

---

## 6. USB Camera Configuration

### 6.1 Install USB/IP Support for WSL2

WSL2 requires USB/IP to access USB devices like cameras.

**On Windows (PowerShell as Admin):**

1. **Install usbipd-win**
   ```powershell
   winget install --interactive --exact dorssel.usbipd-win
   ```

   **Or download manually from**: https://github.com/dorssel/usbipd-win/releases

2. **Restart your computer** after installation

**In WSL2 (Ubuntu):**

```bash
# Install USB/IP tools
sudo apt install -y linux-tools-generic hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*-generic/usbip 20
```

### 6.2 Connect and Bind Camera

1. **Connect your camera to the mini PC via USB**

2. **List USB devices (PowerShell as Admin)**
   ```powershell
   usbipd list
   ```

   Look for your camera (e.g., "Canon EOS 400D" or similar)
   Note the BUSID (e.g., 2-1)

3. **Bind the camera device**
   ```powershell
   usbipd bind --busid 2-1
   ```

   Replace `2-1` with your camera's BUSID

4. **Attach camera to WSL2**
   ```powershell
   usbipd attach --wsl --busid 2-1
   ```

5. **Verify camera in WSL2**

   In Ubuntu terminal:
   ```bash
   # Check if camera is detected
   lsusb

   # Test with gphoto2
   gphoto2 --auto-detect
   ```

   Expected output:
   ```
   Model                          Port
   ----------------------------------------------------------
   Canon EOS 400D                 usb:002,003
   ```

### 6.3 Configure Camera Permissions

```bash
# Add your user to plugdev group
sudo usermod -a -G plugdev $USER

# Create udev rules for camera
sudo tee /etc/udev/rules.d/51-gphoto.rules > /dev/null <<EOF
# Canon cameras
SUBSYSTEM=="usb", ATTRS{idVendor}=="04a9", MODE="0666", GROUP="plugdev"
# Sony cameras
SUBSYSTEM=="usb", ATTRS{idVendor}=="054c", MODE="0666", GROUP="plugdev"
EOF

# Reload udev rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### 6.4 Test Camera Capture

```bash
# Test capturing a photo
gphoto2 --capture-image-and-download --filename=test.jpg

# Verify file was created
ls -lh test.jpg
```

If successful, you should see a new `test.jpg` file downloaded from the camera.

---

## 7. Photopia Application Setup

### 7.1 Create Required Directories

```bash
cd ~/photopia/capture-tower

# Create storage directories
mkdir -p photobooth_photos/originals
mkdir -p photobooth_photos/processed
mkdir -p photobooth_photos/transparent
mkdir -p photobooth_photos/background1
mkdir -p photobooth_photos/background2
mkdir -p backgrounds
mkdir -p backgrounds/cache
```

### 7.2 Configure Environment

The application uses Railway API for configuration, but you need a machine token.

```bash
# Ensure you're in the capture-tower directory
cd ~/photopia/capture-tower

# Activate virtual environment if not already active
source venv/bin/activate
```

---

## 8. Machine Registration

### 8.1 Register Machine with Railway Server

You need to register this machine with the Photopia Railway server to get a machine token.

```bash
# Run registration script
python3 register_machine.py
```

**Follow the prompts:**

1. **Machine ID**: Will be auto-generated from hardware
2. **Machine type**: Enter `capture_tower`
3. **Location**: Enter a descriptive name (e.g., "Mini PC Canon 400D")
4. **Admin key**: Contact your Photopia administrator for the admin key

**Example:**
```
==============================================================
PHOTOPIA MACHINE REGISTRATION
==============================================================

Machine ID: A3B5C7D9E1F3A5B7

Config Server: https://web-production-a8558.up.railway.app

------------------------------------------------------------
Machine type (online/capture_tower/order_station): capture_tower
Location (e.g., Store 1, Office): Mini PC Canon 400D
Admin key: [enter admin key here]
------------------------------------------------------------
```

After successful registration:
- A `.machine_token` file will be created
- A `.machine_id` file will be created
- These files authenticate your machine with the Railway server

### 8.2 Verify Registration

```bash
# Check that token file exists
ls -la .machine_token

# Test connection
python3 -c "from services.database_v3 import DatabaseService; db = DatabaseService(); print('✅ Connection successful')"
```

---

## 9. Testing the System

### 9.1 Start the Application

```bash
# Ensure virtual environment is activated
source venv/bin/activate

# Start the Flask application
python3 app.py
```

Expected output:
```
==================================================
PHOTOPIA - CAPTURE TOWER MODE
==================================================
📱 Machine: A3B5C7D9E1F3A5B7 (Mini PC Canon 400D)
📷 Camera: Connected
🌐 Access: http://localhost:5000
==================================================
```

### 9.2 Access Web Interface

1. **Open browser in Windows**
   - Navigate to: `http://localhost:5000`

2. **Test camera connection**
   - The interface should show camera status
   - Click to start a capture session

3. **Test photo capture**
   - Enter an email address
   - Capture 3 photos
   - Verify photos are processed and uploaded

### 9.3 Stop the Application

Press `Ctrl+C` in the terminal to stop the Flask application.

---

## 10. Auto-start Configuration

For production use, you'll want the capture tower to start automatically when the mini PC boots.

### 10.1 Create Startup Script

```bash
# Create startup script
cat > ~/photopia/start_capture_tower.sh <<'EOF'
#!/bin/bash

# Navigate to project directory
cd ~/photopia/capture-tower

# Activate virtual environment
source venv/bin/activate

# Ensure camera is available (wait for USB device)
sleep 5

# Re-attach camera if needed (requires BUSID - adjust as needed)
# This step may need to be run from Windows PowerShell instead

# Start the application
python3 app.py
EOF

# Make script executable
chmod +x ~/photopia/start_capture_tower.sh
```

### 10.2 Windows Auto-start (Option 1: Task Scheduler)

1. **Open Task Scheduler** (`taskschd.msc`)

2. **Create Basic Task**
   - Name: "Photopia Capture Tower"
   - Trigger: "When the computer starts"
   - Action: "Start a program"

3. **Program/script:**
   ```
   wsl.exe
   ```

4. **Arguments:**
   ```
   -d Ubuntu -e bash -c "cd ~/photopia/capture-tower && source venv/bin/activate && python3 app.py"
   ```

5. **Run with highest privileges**: Check this option

### 10.3 Windows Auto-start (Option 2: Startup Folder)

Create a batch file in the Windows Startup folder:

1. **Create batch file**
   - Press `Win + R`, type `shell:startup`, press Enter
   - Create new file: `PhotopiaCaptureTower.bat`

2. **File contents:**
   ```batch
   @echo off
   REM Attach USB camera (adjust BUSID as needed)
   usbipd attach --wsl --busid 2-1

   REM Start Photopia in WSL2
   wsl -d Ubuntu -e bash -c "cd ~/photopia/capture-tower && source venv/bin/activate && python3 app.py"
   ```

3. **Save and test**
   - Restart computer
   - Application should start automatically

### 10.4 Keep Terminal Open (Optional)

If you want to see logs, modify the batch file to keep the window open:

```batch
@echo off
start "Photopia Capture Tower" wsl -d Ubuntu -e bash -c "cd ~/photopia/capture-tower && source venv/bin/activate && python3 app.py; exec bash"
```

---

## 11. Troubleshooting

### Camera Not Detected

**Problem**: `gphoto2 --auto-detect` shows no camera

**Solutions:**
1. Ensure camera is powered on
2. Check USB cable connection
3. Re-attach USB device:
   ```powershell
   # In PowerShell (Admin)
   usbipd detach --busid 2-1
   usbipd attach --wsl --busid 2-1
   ```
4. Check camera mode (should be in PTP/PC mode, not mass storage)
5. Restart WSL2:
   ```powershell
   wsl --shutdown
   wsl
   ```

### Python Package Installation Fails

**Problem**: Error during `pip install -r requirements.txt`

**Solutions:**
1. Ensure all system libraries are installed (Section 4.3)
2. Update pip:
   ```bash
   python3 -m pip install --upgrade pip
   ```
3. Install packages one by one to identify the problematic package:
   ```bash
   pip install Flask Pillow opencv-python supabase
   ```

### WSL2 Network Issues

**Problem**: Cannot reach Railway server / no internet in WSL2

**Solutions:**
1. Check Windows firewall settings
2. Restart WSL2:
   ```powershell
   wsl --shutdown
   wsl
   ```
3. Check DNS configuration in WSL2:
   ```bash
   cat /etc/resolv.conf
   # Should contain nameserver (e.g., 8.8.8.8)
   ```

### Application Won't Start

**Problem**: Errors when running `python3 app.py`

**Solutions:**
1. Check machine token exists:
   ```bash
   ls -la .machine_token
   ```
2. Verify virtual environment is activated:
   ```bash
   which python3
   # Should show path inside venv directory
   ```
3. Check logs for specific error messages
4. Verify Railway server connection:
   ```bash
   curl https://web-production-a8558.up.railway.app/health
   ```

### USB Device Persistence

**Problem**: Camera disconnects after WSL2 restart

**Solution:**
- USB devices must be re-attached after each WSL2 restart
- Consider creating a PowerShell script to automate this:

```powershell
# auto-attach-camera.ps1
$BUSID = "2-1"  # Adjust as needed
usbipd attach --wsl --busid $BUSID
```

Run this script in Task Scheduler on startup.

### Performance Issues

**Problem**: Application runs slowly

**Solutions:**
1. Ensure adequate RAM (8GB should be sufficient)
2. Close unnecessary applications
3. Check WSL2 memory allocation:

   Create `%USERPROFILE%\.wslconfig`:
   ```
   [wsl2]
   memory=4GB
   processors=2
   ```

   Then restart WSL2:
   ```powershell
   wsl --shutdown
   ```

---

## Quick Reference Commands

### Daily Operations

```bash
# Start capture tower
cd ~/photopia/capture-tower
source venv/bin/activate
python3 app.py

# Check camera connection
gphoto2 --auto-detect

# Check application status
curl http://localhost:5000/health

# View logs (if running in background)
tail -f ~/photopia/capture-tower/logs/app.log
```

### Windows (PowerShell Admin)

```powershell
# List USB devices
usbipd list

# Attach camera to WSL2
usbipd attach --wsl --busid 2-1

# Detach camera
usbipd detach --busid 2-1

# Restart WSL2
wsl --shutdown
wsl
```

### Maintenance

```bash
# Update Python packages
cd ~/photopia/capture-tower
source venv/bin/activate
pip install --upgrade -r requirements.txt

# Update Ubuntu packages
sudo apt update && sudo apt upgrade -y

# Clean up old photos (if needed)
cd ~/photopia/capture-tower/photobooth_photos
rm -rf originals/*.jpg processed/*.jpg
```

---

## Network Configuration

The capture tower needs network access to:
- **Railway Server**: `https://web-production-a8558.up.railway.app`
- **Supabase**: Photo storage and database
- **Modal**: Background removal API

Ensure the mini PC has:
- Stable WiFi or Ethernet connection
- No firewall blocking outbound HTTPS (ports 443, 80)
- DNS resolution working properly

---

## Security Recommendations

1. **Machine Token**: Keep `.machine_token` file secure (already set to 0600 permissions)
2. **Admin Key**: Don't share the admin registration key
3. **Windows Updates**: Enable automatic security updates
4. **User Account**: Use a strong password for Windows and WSL2
5. **Network**: Use WPA2/WPA3 for WiFi, or prefer wired Ethernet

---

## Hardware Specifications Reference

**NiPoGi Pinova P1 Mini PC:**
- CPU: AMD Ryzen 4300U (4C/4T, Max 3.7GHz)
- RAM: 8GB DDR4
- Storage: 256GB SSD
- OS: Windows 11 Pro
- Display: 4K Triple Display (HDMI 2.0 + Type-C + DP1.4)
- Connectivity: WiFi 5, Bluetooth 4.2, LAN

**Recommended Accessories:**
- USB Mouse & Keyboard
- Monitor (HDMI)
- Ethernet cable (for stable connection)
- UPS (for power protection)

---

## Support

For technical support or questions:
- Check application logs in `~/photopia/capture-tower/`
- Review Railway server status
- Contact system administrator

---

## Appendix: Alternative Python Installation (pyenv)

If you need a specific Python version, consider using pyenv:

```bash
# Install pyenv
curl https://pyenv.run | bash

# Add to ~/.bashrc
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc

# Reload shell
source ~/.bashrc

# Install Python 3.10
pyenv install 3.10.12
pyenv global 3.10.12

# Verify
python --version
```

---

## Changelog

- **2024-12-03**: Initial setup guide created for NiPoGi Pinova P1 Mini PC
