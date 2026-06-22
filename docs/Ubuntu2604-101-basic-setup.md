# Ubuntu 26.04 LTS - Basic "onboarding" tasks
<br/>

## 1) First, install some basics
```bash
sudo apt update && sudo apt upgrade -y

sudo apt install wget gpg
sudo apt install net-tools
sudo apt install gnome-tweaks
sudo apt install exfat-fuse ntfs-3g
sudo apt install fonts-cascadia-code
```
<br/>

## 2) Set a "Public Downloads" folder for GNOME system

### Create a Public Downloads folder and set it accessible to system in general 
```bash
mkdir ~/Public/Downloads
sudo chmod 755 ~/Public
sudo chmod 777 ~/Public/Downloads
```

### Change the location of GNOME's "Downloads" shortcut to the new created folder
```bash
nano ~/.config/user-dirs.dirs
```
**Change the line:**
XDG_DOWNLOAD_DIR="$HOME/Downloads"
**To:**
XDG_DOWNLOAD_DIR="$HOME/Public/Downloads"
**And close the editor:**
Ctrl-O, Enter, Ctrl-X

**Update the shortcut references:**
```bash
xdg-user-dirs-update
nautilus -q
```
<br/>

## 3) Install Google Chrome browser (in a "non-sanboxed" way)

### Download Chrome package directly from Google:
```bash
cd ~/Public/Downloads 
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

### Install Chrome
```bash
sudo apt install ~/Public/Downloads/google-chrome-stable_current_amd64.deb
```

### Verify Chrome installation, must be a non-sandboxed Debian:
```bash
which google-chrome
```
**The response will be one of these:**  
-> **/usr/bin/google-chrome** _(OK, fine, it's a debian package. Go to next step.)_  
-> **/snap/bin/google-chrome** _(BAD, it's a snap package, sandboxed: better remove it / reinstall debian.)_  
If it is a "snap package", go to "App Center" GNOME UI, and remove (uninstall) Google Chome.  
After uninstall, run the command to "apt install" the .deb package, and verify again.  

<br/>

## 4) Configure Cloud Accounts
This is done directly in GNOME UI, instead of terminal:  
**Settings -> Online Accounts**  

<br/>

## 5) Hide start-up GRUB menu

### Open and modify the GRUB configuration file
```bash
sudo nano /etc/default/grub
```
**Modify (or add) the three specific lines below:**
GRUB_TIMEOUT=0 
_(This tells the system not to wait)_

GRUB_TIMEOUT_STYLE=hidden 
_(This hides the menu entirely)_

GRUB_RECORDFAIL_TIMEOUT=0 
_(IMPORTANT: by default, if Ubuntu thinks the last boot failed, it ignores the "0" timeout and waits for 30 seconds. **This line overrides that behavior**)_

**Close the editor:**
Ctrl+O, Enter, Ctrl+X

### Regenerate the bootloader files
```bash
sudo update-grub
```

Then, **restart your Ubuntu** (only to check the new boot behavior). 

<br/>

## 6) Add Microsoft official repository to your local sources **(mandatory)**

### Add the Microsoft GPG key to shared keyrings
```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /usr/share/keyrings/microsoft.gpg > /dev/null
```

### Add the Microsof deb repository to your sources, using the GPG key
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
```

**Update your package list:**
```bash
sudo apt update -y
```
<br/>

## 7) Install "VS Code" IDE (optional)

### Install package
```bash
apt search "visual studio code"
sudo apt install code
sudo apt update && sudo apt upgrade -y
```
<br/>

## 8) Install "Edge" browser from Microsoft (optional)

### Install package
```bash
apt search "microsoft-edge"
sudo apt install code
sudo apt update && sudo apt upgrade -y
```
<br/>

## 9) Power settings (optional, useful for notebooks or RDP-accessed desktops)

### Disable suspend when closing the lid (only when plugged-in)
```bash
sudo nano /etc/systemd/logind.conf
```
**Edit these lines:**
HandleLidSwitch=suspend  
HandleLidSwitchExternalPower=ignore  
#LidSwitchIgnoreInhibited=yes -> (keep commented)  

**Close the editor:**
Ctrl-O, Enter, Ctrl-X

**Restart the login daemon:**
```bash
sudo systemctl restart systemd-logind
```

### Disable automatic suspend (only when plugged in):
**Set the GNOME option:**
```bash
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout 0
```
**Verify:**
```bash
gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout
```

### Disable both the screen blanking and the automatic lock

**1. Disable Screen Blanking (Set timeout to "Never"):**
```bash
gsettings set org.gnome.desktop.session idle-delay 0
```

**2. Disable the Screen Lock:**
```bash
gsettings set org.gnome.desktop.screensaver lock-enabled false
```

**3. Prevent the Lock Screen from triggering on suspend (Optional but recommended for RDP):**
```bash
gsettings set org.gnome.desktop.screensaver ubuntu-lock-on-suspend false
```
<br/>

## 10) RDP Client: Remmina (optional)

1. **It is often pre-installed, but you can grab the complete package with all protocol plugins via the terminal:**

```bash
sudo apt update
sudo apt install remmina remmina-plugin-rdp remmina-plugin-secret
```

2. **Connect**

- Open Remmina.

- In the _quick-connect_ bar at the top, ensure the dropdown is set to **RDP**.

- Type the _target IP address_ and hit **Enter**.

- To "_save the connection_" (so you don't have to re-type the credentials every time):

    - Click the _New Connection Profile_ icon (the small paper sheet with a + in the top left).

    - Fill in your Server IP, Username, Password, and set the Resolution to "_Use client resolution_" so it matches your own screen perfectly.

---

<br/>

## (Extra) Some basic (but useful) tips for beginners

### Standard Terminal Commands
```bash
ls
cd
mkdir
rm 
cat
clear
cp
```

### Use embedded "Manual Pages" inside Terminal

**--> Call "man" + desired topic**  
```bash
man apt
```

**-> Call "man" + section + topic**  
```bash
man 8 apt
```

**-> Call "man" + search by keyword**  
```bash
man -k apt
```

### Commands for Hardware information
```bash
lsusb

lspci 

lspci -nnk
```

### Other commands

```bash
apt search "app name"
chmod 1777 -> Only owner can delete
```

---