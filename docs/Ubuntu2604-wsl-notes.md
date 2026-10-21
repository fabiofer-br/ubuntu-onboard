
---

I need to fully remove my WSL content (distributions, files, virtual disks, and so on) from this computer, so I can start again from scratch, with a clean WSL configuration and an Ubuntu 26.04 distribution.
Can you help me with this? 

---

I'll start by checking what's currently installed. I won't remove anything until you confirm.

Ran terminal command: 
```powershell
wsl --version; "----"; wsl -l -v; "----"; wsl -l --online
```

Ran terminal command: 
```powershell
Test-Path "$env:USERPROFILE\.wslconfig"; Get-Content "$env:USERPROFILE\.wslconfig" -ErrorAction SilentlyContinue; "----"; Test-Path "$env:USERPROFILE\.wslgconfig"; "----"; Get-ChildItem HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss -ErrorAction SilentlyContinue | ForEach-Object { $p = Get-ItemProperty $_.PSPath; [pscustomobject]@{Name=$p.DistributionName; Path=$p.BasePath; Pkg=$p.PackageFamilyName; Ver=$p.Version} } | Format-List
```

Ran terminal command: 
```powershell
Get-AppxPackage | Where-Object { $_.Name -match 'Ubuntu|Debian|Linux|SUSE|Kali|WSL|Fedora|Alma|Oracle' } | Select-Object Name, PackageFullName | Format-Table -AutoSize; "----"; Get-ChildItem "$env:LOCALAPPDATA\Packages" -Directory -ErrorAction SilentlyContinue | Where-Object Name -match 'Canonical|Ubuntu|Debian|WSL|Linux|SUSE|Kali' | Select-Object FullName; "----"; Get-ChildItem "$env:LOCALAPPDATA\wsl" -Recurse -Filter *.vhdx -ErrorAction SilentlyContinue | Select-Object FullName, @{n='GB';e={[math]::Round($_.Length/1GB,2)}}
```

Here's what I found. Let me also check the swap disk and VHDX sizes.

Ran terminal command: 
```powershell
Get-Item "C:\Users\fabio\AppData\Local\wsl\{067a2096-6aad-479f-b454-c52372689f87}\ext4.vhdx", "D:\temp\wsl-swap.vhdx" -ErrorAction SilentlyContinue | Select-Object FullName, @{n='GB';e={[math]::Round($_.Length/1GB,2)}}; "----"; Get-ChildItem "$env:LOCALAPPDATA\wsl" -Force | Select-Object Name; "----"; Get-ChildItem "$env:LOCALAPPDATA\Temp" -Filter "wsl-crashes" -Directory -ErrorAction SilentlyContinue | Select-Object FullName
```

Ask User
Completed with input: {
  "message": "Removing this can't be undone. Please confirm how far the cleanup should go.",
  "requestedSchema": {
    "properties": {
      "backup": {
        "type": "boolean",
        "title": "Back up Ubuntu-26.04 first?",
        "description": "Export the current distro to a .tar file (about 3 GB) before deleting it, just in case.",
        "default": false
      },
      "wslconfig": {
        "type": "string",
        "title": ".wslconfig handling",
        "oneOf": [
          {
            "const": "backup_delete",
            "title": "Rename it to .wslconfig.bak (clean defaults, but old settings kept for reference)"
          },
          {
            "const": "delete",
            "title": "Delete it completely"
          },
          {
            "const": "keep",
            "title": "Keep it as-is"
          }
        ],
        "default": "backup_delete"
      },
      "platform": {
        "type": "string",
        "title": "WSL platform itself",
        "oneOf": [
          {
            "const": "keep",
            "title": "Keep the WSL app installed (unregister distro + wipe files only), then reinstall Ubuntu 26.04"
          },
          {
            "const": "reinstall",
            "title": "Also uninstall and reinstall the WSL app (fully fresh; needs admin rights)"
          }
        ],
        "default": "keep"
      },
      "install": {
        "type": "boolean",
        "title": "Install a fresh Ubuntu-26.04 afterwards?",
        "description": "You'll need to create the Linux username/password in a terminal window.",
        "default": true
      }
    }
  }
}

Got it: no backup, rename `.wslconfig`, reinstall the WSL platform, and install a fresh Ubuntu-26.04. First I'll shut WSL down, then unregister the distro and delete the leftover files.

Ran terminal command: 
```powershell
$isAdmin = ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator); "Admin: $isAdmin"
wsl --shutdown; "shutdown exit: $LASTEXITCODE"
wsl --unregister Ubuntu-26.04; "unregister exit: $LASTEXITCODE"
wsl -l -v
```
Tool execution failed with exit code -1

The distro is unregistered. Next I'll clean up the leftover files and rename `.wslconfig`.

Ran terminal command: 
```powershell
wsl --shutdown
Remove-Item "$env:LOCALAPPDATA\wsl" -Recurse -Force -ErrorAction Continue
Remove-Item "D:\temp\wsl-swap.vhdx" -Force -ErrorAction Continue
if (Test-Path "$env:USERPROFILE\.wslconfig.bak") { Remove-Item "$env:USERPROFILE\.wslconfig.bak" -Force }
Rename-Item "$env:USERPROFILE\.wslconfig" ".wslconfig.bak"
"wsl dir exists: $(Test-Path "$env:LOCALAPPDATA\wsl")"; "swap exists: $(Test-Path D:\temp\wsl-swap.vhdx)"; "wslconfig exists: $(Test-Path "$env:USERPROFILE\.wslconfig")"; "bak exists: $(Test-Path "$env:USERPROFILE\.wslconfig.bak")"
Get-ChildItem HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss -ErrorAction SilentlyContinue
```
The files are cleaned up. Next I'll uninstall the WSL platform. Before that I'm checking whether it was installed as an MSI as well as the Store package.

Ran terminal command: 
```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*, HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* -ErrorAction SilentlyContinue | Where-Object DisplayName -match 'Subsystem for Linux|WSL' | Select-Object DisplayName, DisplayVersion, PSChildName | Format-List; Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux, VirtualMachinePlatform -ErrorAction SilentlyContinue | Select-Object FeatureName, State
```
Tool execution failed with exit code 1

Ran terminal command: 
```powershell
$log = "$env:TEMP\wsl-reinstall.log"
$script = @"
Start-Transcript -Path '$log' -Force
wsl --shutdown
wsl --uninstall
"uninstall exit: `$LASTEXITCODE"
Get-AppxPackage MicrosoftCorporationII.WindowsSubsystemForLinux -AllUsers | Remove-AppxPackage -AllUsers -ErrorAction SilentlyContinue
Remove-Item 'C:\Users\fabio\AppData\Local\Packages\MicrosoftCorporationII.WindowsSubsystemForLinux_8wekyb3d8bbwe' -Recurse -Force -ErrorAction SilentlyContinue
"after uninstall MSI present: " + [bool](Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* -ErrorAction SilentlyContinue | Where-Object DisplayName -match 'Subsystem for Linux')
wsl --install --no-distribution
"install exit: `$LASTEXITCODE"
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform | Select-Object FeatureName, State
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux | Select-Object FeatureName, State
Stop-Transcript
"@
$p = Start-Process pwsh -Verb RunAs -ArgumentList '-NoProfile','-ExecutionPolicy','Bypass','-Command',$script -Wait -PassThru
"elevated exit: $($p.ExitCode)"
Get-Content $log
```
The echo lines failed because of quoting, but the real commands ran: WSL was uninstalled, then 3.0.3 was downloaded and installed fresh. I'll check that state, then install Ubuntu-26.04.

Ran terminal command: 
```powershell
wsl --version; "----"; wsl -l -v; "----"; "wsl data dir: $(Test-Path "$env:LOCALAPPDATA\wsl")"; "wslconfig: $(Test-Path "$env:USERPROFILE\.wslconfig")"; Get-AppxPackage *WindowsSubsystemForLinux* | Select-Object PackageFullName
```

Ran terminal command: 
```powershell
wsl --install -d Ubuntu-26.04 --no-launch; "exit: $LASTEXITCODE"; wsl -l -v
```

WSL is wiped and set up again, and a fresh Ubuntu-26.04 is installed. The only step left is creating your Linux user.

**What was removed:**
- The old `Ubuntu-26.04` distro. I unregistered it, which deleted its ~2.9 GB virtual disk.
- Everything left in `%LOCALAPPDATA%\wsl`, plus the swap disk `D:\temp\wsl-swap.vhdx`.
- The WSL platform itself (MSI and Store app) and its app data folder.

**What's in place now:**
- WSL 3.0.3, installed fresh. The check before installing Ubuntu showed no distributions and no leftover data folder.
- No `.wslconfig`, so WSL uses its default settings. Your old settings are saved in `C:\Users\fabio\.wslconfig.bak`. If you want any of them back (the 12 GB memory and 4 processor limits, nested virtualization, `firewall=false`), copy those lines into a new `.wslconfig`.
- `Ubuntu-26.04` (WSL 2) is installed, set as the default, and currently stopped.

During the install, WSL reported that it couldn't create its virtual network on the old address range because that network still existed. It switched to a new range (172.28.48.0/20) on its own and the install still finished, so this shouldn't need any action.

**Your next step:** open a terminal and run this to create your Linux username and password:
```powershell
wsl -d Ubuntu-26.04
```

---

Thanks! Question: where is the Ubuntu virtual hard disk located?

---

Ran terminal command: 
```powershell
Get-ChildItem HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss | ForEach-Object { $p = Get-ItemProperty $_.PSPath; $p.DistributionName; $p.BasePath; $p.VhdFileName }; Get-ChildItem "$env:LOCALAPPDATA\wsl" -Recurse -Filter *.vhdx | Select-Object FullName, @{n='GB';e={[math]::Round($_.Length/1GB,2)}}
```

Ubuntu's virtual hard disk is here:

```
C:\Users\fabio\AppData\Local\wsl\{aef6bdf0-e2a7-41df-8e30-ce68d9f68457}\ext4.vhdx
```

It's about 1.36 GB right now and grows as you add files to Ubuntu. The folder name inside `{…}` is the distro's new ID, so it's different from the old install's folder.

If you'd like it on another drive, for example `D:`, shut WSL down first and then move it:

```powershell
wsl --shutdown
wsl --manage Ubuntu-26.04 --move D:\WSL\Ubuntu-26.04
```

