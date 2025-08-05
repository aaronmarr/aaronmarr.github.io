---
title: Setting up Fedora Linux for audio production
description: >-
  In this post I will look at setting up Fedora (KDE) for audio production. I'll install Bitwig, and optimise the system for audio. I'll also look at running VSTs via yabridge. 
author: aaron
date: 2025-08-04 00:00:00 +0000
categories: [Music Production]
tags: [linux, audio]
pin: true
media_subpath: '/posts/20180809'
---

## Installing Bitwig

Bitwig is available for Linux in 2 flavours: there's a Flatpak version, and an official Ubuntu package. I'm staying away from the Flatpak version due to Flatpak's sandboxing, which prevents use of tools such as [Jack](https://jackaudio.org/).

To install the Ubuntu version on Fedora we will need to convert the `.deb` package into `rpm` format. For this, I'm using [alien](https://packages.fedoraproject.org/pkgs/alien/alien/#:~:text=Alien%20is%20a%20program%20that,package%20format%20and%20install%20it.):

```bash
sudo dnf install alien
```
    
Next, [download Bitwig for Ubuntu](https://www.bitwig.com/download/) and convert the `.deb` package to `rpm`:

```bash
cd ~/Downloads
sudo alien -r bitwig-studio-5.3.12.deb
```

I THINK THIS CAN BE REMOVED

```bash
sudo dnf copr enable aflyhorse/libjpeg 
sudo dnf install libjpeg8
```

Having created the `.rpm` package, I was still no able to `dnf install` it, so we'll manually install the binary and create a `.desktop` entry for KDE's launcher.

```bash
# Create a temporary directory
mkdir ~/bitwig-extract
cd ~/bitwig-extract

# Extract the Bitwig package contents
rpm2cpio ~/Downloads/bitwig-studio-5.3.12-2.x86_64.rpm | cpio -idmv

# Install the Bitwig binary to the system
sudo mkdir -p /opt/bitwig-studio
sudo cp -r opt/bitwig-studio/* /opt/bitwig-studio/

# Create system-wide command
sudo ln -s /opt/bitwig-studio/bitwig-studio /usr/local/bin/bitwig
```

At this point you should be able to run Bitwig from the command line:

```bash
bitwig
```

Next, we will create a desktop entry for KDE's launcher:

```bash
# Copy desktop entry
cp usr/share/applications/bitwig.desktop ~/.local/share/applications/

# Copy icon to permanent location
mkdir -p ~/.local/share/icons/hicolor/128x128/apps/
cp /home/aaron/bitwig-extract/usr/share/icons/hicolor/128x128/apps/com.bitwig.BitwigStudio.png ~/.local/share/icons/hicolor/128x128/apps/bitwig.png
```

Note that unfortunately, until Bitwig support Wayland natively, you will still see the X11 icon in the panel when Bitwig is running. There may be a workaround for this, but I've not been able to find one at the time of writing.

Edit the desktop file to fix paths:

```bash
nano ~/.local/share/applications/bitwig.desktop
```

Update the content to use correct paths:

```ini
[Desktop Entry]
Categories=AudioVideo;Audio;Music;Midi;
Comment=
Exec=/opt/bitwig-studio/BitwigStudio
Icon=com.bitwig.BitwigStudio
Name=Bitwig Studio
NoDisplay=false
Path=
PrefersNonDefaultGPU=false
StartupNotify=true
Terminal=false
TerminalOptions=
Type=Application
X-KDE-SubstituteUID=false
X-KDE-Username=
StartupWMClass=com.bitwig.BitwigStudio
```

Finally, update the system desktop entries and icons using: 

```
kbuildsycoca5
gtk-update-icon-cache ~/.local/share/icons/hicolor/
```

Now Bitwig should be available in the application launcher.

Clean up temporary files:

```bash
cd ~
rm -rf ~/bitwig-extract
```

## System Optimization

### Audio Performance Tweaks

Install real-time kernel for lower latency:

```bash
sudo dnf install kernel-rt kernel-rt-devel
```

Add user to audio group:

```bash
sudo usermod -a -G audio $USER
```

Configure audio limits:

```bash
sudo nano /etc/security/limits.d/audio.conf
```

Add these lines:

```
@audio - rtprio 95
@audio - memlock unlimited
```

Reboot to apply changes:

```bash
sudo reboot
```

### Install JACK

Install JACK audio system and control interface:

```bash
sudo dnf install jack-audio-connection-kit qjackctl
```

## Running Windows VSTs with yabridge

### Install yabridge

```bash
sudo dnf copr enable patrickl/yabridge
sudo dnf install yabridge yabridgectl wine
```

### Configure Wine for VSTs

Create separate Wine prefix for plugins:

```bash
export WINEPREFIX=~/.wine-vst
winecfg  # Set to Windows 10
```

### Install VST Plugins

Install Windows VSTs in Wine:

```bash
export WINEPREFIX=~/.wine-vst
wine your-vst-installer.exe
```

### Setup yabridge

Register VST directories and sync:

```bash
# Add VST3 directory
yabridgectl add ~/.wine-vst/drive_c/Program\ Files/Common\ Files/VST3/

# Add VST2 directory if needed
yabridgectl add ~/.wine-vst/drive_c/Program\ Files/VSTPlugins/

# Create bridge files
yabridgectl sync
```

Your VSTs should now appear in Bitwig's plugin browser after rescanning.
