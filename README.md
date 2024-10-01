# Jetson Installation Guide from Flashing

## Table of Contents
- [System Update and Upgrade](#system-update-and-upgrade)
- [Install Python and Virtual Environment](#install-python-and-virtual-environment)
- [Set Up Jetson Clocks and Power Mode on Startup](#3-set-up-jetson-clocks-and-power-mode-on-startup)
- [Enable SSH Access](#4-enable-ssh-access)
- [Install jtop for Monitoring](#5-install-jtop-for-monitoring)
- [Installation of NoMachine](#6-installation-of-nomachine)
- [Cuda and Pytorch installation](#7-cuda-and-pytorch-installation)



This guide provides step-by-step instructions to set up a Jetson device from flashing, including system updates, Python setup, clock services, and additional utilities.

## System Update and Upgrade

After flashing the Jetson device, start by updating and upgrading the system:

```bash
sudo apt update
sudo apt upgrade -y
```

If you encounter any issues related to the `nvidia-l4t-bootloader` during the upgrade process, refer to the solution provided on the NVIDIA forums: [Solution Link](https://forums.developer.nvidia.com/t/solution-dpkg-error-processing-package-nvidia-l4t-bootloader-configure/208627).

In case of dpkg error while upgraging:
```
 nvidia-l4t-bootloader
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

Apparently it is related to the information that dpkg saves and it conflicts in the installation
So wee needed move /var/lib/info/ and create new /var/lib/dpkg/info

```
sudo mv /var/lib/dpkg/info/ /var/lib/dpkg/backup/
sudo mkdir /var/lib/dpkg/info/
```

Next update repos and force install 
```
sudo apt-get update
sudo apt-get -f install
```
Install Nano Text Editor
```
sudo apt-get install nano
```
Ensure power mode is set to max first then reboot the jetson
```
sudo nvpmodel -m 0
```
Move the new structure dpkg/info to old info

```
sudo mv /var/lib/dpkg/info/* /var/lib/dpkg/backup/
```
Remove the new dpkg structure folder and back the old
```
sudo rm -rf /var/lib/dpkg/info
sudo mv /var/lib/dpkg/backup/ /var/lib/dpkg/info/
```

## Install Python and Virtual Environment

Install pip and venv for Python3:
```
sudo apt install python3-pip
sudo apt install python3-venv

```
## 3. Set Up Jetson Clocks and Power Mode on Startup

### 3.1 Create the Startup Script
Create a script that sets the Jetson to maximum power mode and activates Jetson clocks:
```
sudo nano /usr/local/bin/jetson_startup.sh
```
Add the following content to the script:

```
#!/bin/bash

# Activate Jetson clocks
sudo jetson_clocks

# Show current clock settings for verification
sudo jetson_clocks --show

```

Save and exit (CTRL + X, then Y, and Enter).


### 3.2 Make the Script Executable
```
sudo chmod +x /usr/local/bin/jetson_startup.sh
```

### 3.3 Create a Systemd Service for the Startup Script
Create a systemd service to run the script on startup
```
sudo nano /etc/systemd/system/jetson_startup.service
```
Add the following content to the service file:

```
[Unit]
Description=Run Jetson Clock and Max Power on Startup
After=network.target

[Service]
ExecStart=/usr/local/bin/jetson_startup.sh
User=root
Type=simple

[Install]
WantedBy=multi-user.target
```


### 3.4 Enable and Start the Service
Reload the systemd daemon and enable the service
```
sudo systemctl daemon-reload
sudo systemctl enable jetson_startup.service
sudo systemctl start jetson_startup.service
```


## 4. Enable SSH Access
To enable SSH on the Jetson device
```
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh

```

## 5. Install jtop for Monitoring
```
sudo pip3 install -U jetson-stats
```
## 6. Installation of NoMachine
Remote control program which i use for jetsons, I used it lots of jetson device and it worked well

Download this file [Download / NoMachine for ARM/ NoMachine for ARM ARMv8 DEB](https://downloads.nomachine.com/linux/?distro=Arm&id=30)

After installation, use dpkg and that's it
```
sudo dpkg -i nomachine_8.13.1_1_arm64.deb
```



## 7-Cuda and Pytorch installation

![Uygulama Ekran Görüntüsü](https://github.com/Erzn3522/Jetson_Orin_Setup/blob/main/jtop.png)
This is the most hated part. I sacrifed my days to figure out how can i get out the Cuda hell then finally got it, here :

### 1- First install Jetpack to device, I install via Nvidia SDK Manager. I choose Jetpack version is 5.1.3
```
sudo apt-get -y update; 
sudo apt-get -y install python3-pip libopenblas-dev;
```

### 2- Install Cuda Toolkit
```
sudo apt-get install nvidia-jetpack
```
### 3- Create a venv and activate the venv  first
```
python3 -m venv venv
source venv/bin/activate
```
### 4- Download the Correct PyTorch Version for CUDA 11.4
```
wget https://nvidia.box.com/shared/static/0lcofe2uvslg1zb7lpplzvtzvwv1f6fq.whl -O torch-2.0.0+nv23.05-cp38-cp38-linux_aarch64.whl
sudo apt-get install python3-pip cmake libopenblas-dev libopenmpi-dev sudo apt-get install python3-pip cmake libopenblas-dev libopenmpi-dev 
sudo apt-get -y update
sudo apt-get -y install python3-pip libopenblas-dev
sudo apt-get install python3-pip libopenblas-base libopenmpi-dev libomp-dev libjpeg-dev zlib1g-dev libpython3-dev libopenblas-dev libavcodec-dev libavformat-dev libswscale-dev
export TORCH_INSTALL=https://developer.download.nvidia.cn/compute/redist/jp/v511/pytorch/torch-2.0.0+nv23.05-cp38-cp38-linux_aarch64.whl
python3 -m pip install --upgrade pip; python3 -m pip install numpy==’1.26.1’; python3 -m pip install --no-cache $TORCH_INSTALL
export LD_LIBRARY_PATH=/usr/lib/llvm-8/lib:$LD_LIBRARY_PATH
sudo apt list libcudnn8
```


### 6- Install the Correct Torchvision Version
```
git clone --branch v0.15.1 https://github.com/pytorch/vision torchvision
cd torchvision
export BUILD_VERSION=0.15.1
python3 setup.py install # If you want to install for whole users not only for venv, use this python3 setup.py install --user
```
### 7- Install other packages
```
pip3 install opencv-python
pip3 install ultralytics --no-deps # If you dont use --no-deps parameter for this command, it will remove the cuda enabled torch libraries
pip3 install matplotlib opencv-python>=4.6.0 pandas>=1.1.4 pillow>=7.1.2 psutil py-cpuinfo pyyaml>=5.3.1 requests>=2.23.0 scipy>=1.4.1 seaborn>=0.11.0 tqdm>=4.64.0 ultralytics-thop>=2.0.0
```
