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

This guide covers setting up Fedora Linux for audio production, including installing Bitwig Studio, optimizing for low-latency audio, and running Windows VSTs via yabridge.

## Installing Bitwig

Bitwig is available for Linux in 2 flavours: there's a Flatpak version, and an official Ubuntu package. I'm staying away from the Flatpak version due to Flatpak's sandboxing, which prevents use of tools such as [Jack](https://jackaudio.org/).

To install the Ubuntu version on Fedora:

Install [alien](https://packages.fedoraproject.org/pkgs/alien/alien/#:~:text=Alien%20is%20a%20program%20that,package%20format%20and%20install%20it.) (converts .deb packages to .rpm):
    ```bash
    sudo dnf install alien
    ```
    
[Download Bitwig for Ubuntu](https://www.bitwig.com/download/) and convert the .deb package to RPM:
    ```bash
    cd ~/Downloads
    sudo alien -r bitwig-studio-5.3.12.deb
    ```

If you get `libjpeg` errors, install the required library:

```bash
sudo dnf copr enable aflyhorse/libjpeg 
sudo dnf install libjpeg8
```

Extract and install Bitwig manually:

```bash
# Create temporary extraction directory
mkdir ~/bitwig-extract
cd ~/bitwig-extract

# Extract the RPM contents
rpm2cpio ~/Downloads/bitwig-studio-5.3.12-2.x86_64.rpm | cpio -idmv

# Install to system location
sudo mkdir -p /opt/bitwig-studio
sudo cp -r opt/bitwig-studio/* /opt/bitwig-studio/

# Create system-wide command
sudo ln -s /opt/bitwig-studio/bitwig-studio /usr/local/bin/bitwig
```

Test the installation:

```bash
bitwig
```

Create a desktop entry for KDE's launcher:

```bash
# Copy desktop entry
cp usr/share/applications/bitwig.desktop ~/.local/share/applications/

# Copy icon to permanent location
mkdir -p ~/Applications/Bitwig
cp usr/share/icons/hicolor/128x128/apps/com.bitwig.BitwigStudio.png ~/Applications/Bitwig/
```

Edit the desktop file to fix paths:

```bash
nano ~/.local/share/applications/bitwig.desktop
```

Update the content to use correct paths:

```ini
[Desktop Entry]
Name=Bitwig Studio
Exec=/usr/local/bin/bitwig
Icon=/home/$USER/Applications/Bitwig/com.bitwig.BitwigStudio.png
Type=Application
Categories=AudioVideo;Audio;Music;Midi;
Terminal=false
StartupNotify=true
```

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
