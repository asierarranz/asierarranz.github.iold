---
layout: post
title: Ubuntu 16.04 + CUDA + cuDNN on a 13" GTX1060 QHD Razer Blade
---

![alt text](https://asierarranz.github.io/images/cuDNNblade.jpg "Ubuntu 16.04 + CUDA + cuDNN on a 13" GTX1060 QHD Razer Blade")
Probably the Razer Blade is the best small notebook for machine learning available in the market right now. So here you have the steps I have followed to deal with some of the issues that the standard Ubuntu installation has.

---

### Installing Ubuntu from Windows

- First, in Windows, go to Disk Management -> Shrink volume, and free around 30-50Gb
- Download Ubuntu 16.04
- Flash it into an USB using Rufus
- Reboot & keep F1 pressed to enter BIOS
- Disable Secure Boot & Save
- Reboot & keep F12 pressed
- Select UEFI: (your externar_usb_name)
- Install Ubuntu
- Connect to your wifi
- Install Third Party & Download updates
- Choose manually partitioning (Something else)
- Select the new partition, check the format box, and add an Ext4 file system. Mount point: / 
We do it in this way to avoid the creation of a swap partition that could damage the SSD due to a high ratio of Writes/Reads. Discard that warning.

-----------

### Configuring Ubuntu 16.04

```bash
sudo apt update
sudo apt upgrade
```
Could be a warning from the last kernel that is not supporting the intel i915 firmware, if that happens, enter here:
<https://01.org/linuxgraphics/intel-linux-graphics-firmwares>

1. Download the *Skylake GUC v6* if it has a tar_x file, rename it to .tar, extract the folder and run:
 
```bash
sudo bash ./install.sh --install
sudo reboot
```

2. Download the *Kabylake DMC v1* if it has a tar_x file, rename it to .tar, extract the folder and run:

```bash
sudo sh install.sh
sudo reboot
```

#### Keyboard

If the keyboard is not working, try rebooting multiple times from Windows in different ways, (or use the on-screen keyboard in the top right menu) it is a power issue that can be solved editing /etc/rc.local and adding this line before exit 0:

```bash
echo -1 /sys/bus/usb/devices/3-1/power/autosuspend_delay_ms
```
if the keyboard is still failing try to enable the compatibility settings, like xHCI, into the BIOS (Pressing F1 during power-on)

#### External Bluetooth mouse

If you need an external bluetooth mouse and it is not working when it is connected:

```bash
sudo hciconfig hci0 sspmode 1
sudo hciconfig hci0 down
sudo hciconfig hci0 up
```
And connect-disconnect until it works. After that it will work like a charm forever :-)


#### Cinnamon Desktop

I prefer to install Cinnamon Desktop due to its better support for high-DPI displays, and I think it looks much better than the standard Unity!

```bash
sudo add-apt-repository ppa:embrosyn/cinnamon
sudo apt update && sudo apt install cinnamon
```
Log-off and re-login clicking in the ubuntu logo next to your username, select Cinnamon.

#### Sound

If like to me, the sound is not working:

```bash
sudo apt-get install pulseaudio pavucontrol
sudo reboot
```

-----------------------

### Install CUDA

This is for the 8.0.44-1 version, the process should be similar for future versions by replacing the url.

```bash
wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
sudo apt-get update # (here you could get an "Invalid Date" warning, you can ignore it)
sudo apt-get install cuda # (Wait, around 2Gb will be downloaded)
gedit .bashrc
```
Add these 2 lines at the end:

```bash
export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

------

### Install nVidia CUDA® Deep Neural Network library (cuDNN)
- Go to: <https://developer.nvidia.com/cudnn>
- Download cuDNN v5.1 Library for Linux (could be newer versions when you read this)

```bash
tar xvzf cudnn-8.0-linux-x64-v5.1.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```

----

### Test CUDA!

Now you can check the details about the card and test the installation:

```bash
nvidia-smi
cd /usr/local/cuda/samples/5_Simulations/nbody
sudo make
./nbody -benchmark -numbodies=256000 -device=0
```

-----


