
That is an impressively clean setup for a classic Dell Inspiron 5558!<br/> Running a modern backend stack on Ubuntu 26.04.1 LTS with a fast SSD and 16 GiB of RAM makes perfect sense for development work.

However, since your notebook is completely reliant on that single USB 3.0 Type-A port for a second monitor, and you are running GNOME 50.1 on Wayland with a Linux 7.0+ kernel, you are navigating the exact cutting edge of Linux graphics configuration.

Here is exactly what you need to know and do to get a multi-monitor layout working over USB-A on your machine.

## 1. Hardware Requirement: Buy a DisplayLink DL-6xxx Device
You must buy an external adapter or simple dock powered by a Synaptics DisplayLink chip (ideally from the DL-6xxx series, which supports up to 4K resolutions and handles modern compression better).

* Do not buy standard $10 to $20 "USB 3.0 to HDMI" adapters. Those use Silicon Motion or Fresco Logic chips which do not have working drivers on modern Linux kernels.
* Certified brands like Wavlink, Plugable, or StarTech clearly label if they are powered by "DisplayLink."

## 2. The Linux 7.0 & Wayland Catch
Ubuntu 26.04 LTS dropped standard X11 sessions entirely, meaning you are running Wayland-only by default. This introduces two distinct software hurdles:  
 
* The Kernel Module (EVDI): DisplayLink relies on an open-source kernel module called evdi to create virtual screens. Because you are running a brand-new Linux 7.0 kernel, the standard module bundled in the official driver download may fail to compile initially due to structural changes in the kernel source.  
* Wayland Hotplugging: While newer versions of the [Synaptics Ubuntu Driver](https://www.synaptics.com/products/displaylink-graphics/downloads/ubuntu) natively support Ubuntu 26.04 out of the box, some Wayland compositors occasionally require a system reboot or session restart to recognize a display the very first time you hotplug it.  

## 3. Step-by-Step Installation Process
Once you have your DisplayLink hardware, open your terminal and follow these steps to deploy the driver environment:

### Step 1: Install Build Dependencies
You need the necessary tools to compile the virtual interface module against your running kernel:

```bash
sudo apt update
sudo apt install dkms libdrm-dev build-essential
```

### Step 2: Check Secure Boot Status
Because this builds a third-party kernel module (evdi), if you have UEFI Secure Boot enabled, your system will block it from loading unless it is signed.  
 
* Run mokutil --sb-state to check if it's active.
* If it is enabled, you can temporarily disable Secure Boot in your Dell BIOS settings to guarantee a seamless driver installation.  

### Step 3: Download and Deploy the Driver

   1. Head to the official Synaptics Linux Downloads page and grab the installer package for Ubuntu 26.04.
   2. Extract the .zip archive to find the script file (it ends in .run).
   3. Make it executable and run it with superuser permissions:  

```bash
chmod +x displaylink-driver-xxxx.run
sudo ./displaylink-driver-xxxx.run
```

(If the installer throws a DKMS compilation error on Linux 7.0, you may need to fetch the latest evdi module directly from the [DisplayLink EVDI GitHub Repository](https://github.com/DisplayLink/evdi) to override the legacy bundled version).  

### Step 4: Connect and Enable
Plug the adapter into your USB 3.0 Type-A port. If the display is dark initially, head to Settings ➔ Displays inside GNOME. Because you run your notebook layout with the main internal screen disabled, you will see your original external monitor alongside a newly recognized second layout that you can toggle "On" and position accordingly. 

---
