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

Bitwig is available for Linux in two flavours. There's a Flatpak version, and an official Ubuntu package. I'm staying away from the Flatpak version due to Flatpak's sandboxing, which prevents us from using [yabridge](https://github.com/robbert-vdh/yabridge) and [Jack](https://jackaudio.org/).

To install the Ubuntu version on Fedora we will need to convert the `.deb` package into `rpm` format. For this, I'm using [alien](https://packages.fedoraproject.org/pkgs/alien/alien/#:~:text=Alien%20is%20a%20program%20that,package%20format%20and%20install%20it.), which is a Linux tool for converting between different package formats:

```bash
# Install alien
sudo dnf install alien
```
    
Next, [download Bitwig for Ubuntu](https://www.bitwig.com/download/) and convert the `.deb` package to `rpm`:

```bash
# Download Bitwig from https://www.bitwig.com/download/ then
cd ~/Downloads

# Create our Bitwig rpm
sudo alien -r bitwig-studio-5.3.12.deb
```

> Having created the `.rpm` package, I was still not able to `dnf install` it, so we'll manually install the binary and create a `.desktop` entry for KDE's launcher.
{: .prompt-info }

```bash
# Create a temporary directory
mkdir ~/bitwig-extract

cd ~/bitwig-extract

# Extract the Bitwig package
rpm2cpio ~/Downloads/bitwig-studio-5.3.12-2.x86_64.rpm | cpio -idmv

# Install the Bitwig binary manually
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

> Until Bitwig supports Wayland natively, you will still see the X11 icon in the panel when Bitwig is running. There may be a workaround for this, but I've not been able to find one at the time of writing.
{: .prompt-info }

Next we will create the `.desktop` file:

```bash
nano ~/.local/share/applications/bitwig.desktop
```

Paste in the following contents, which defines our desktop entry:

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

Save and exit from `nano` (`CTRL+O` then `CTRL+X`):

Next, update the system desktop entries and icons using: 

```
kbuildsycoca5
gtk-update-icon-cache ~/.local/share/icons/hicolor/
```

Finally, clean up temporary files:

```bash
cd ~
rm -rf ~/bitwig-extract
```

At this point you can use Bitwig, and it should be available in the application launcher. If you would like to find out how to configure the system for performance, and set up `yabridge` for VST plugins, please read on!

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





https://www.youtube.com/watch?v=zmmdyX381Go


https://medium.com/@jeromedecinco/tuned-in-linux-optimizing-system-performance-with-profiles-1c852acfb02e

https://discussion.fedoraproject.org/t/pipewire-configuration-for-low-latency/32221/4

Install rtcqs, and fix errors

--

User aaron is currently not member of a group that has sufficient rtprio (0) and memlock (8388608) set. Add yourself to a group with sufficent limits set, i.e. audio or realtime, with 'sudo usermod -a -G <group_name> aaron. See also https://wiki.linuxaudio.org/wiki/system_configuration#audio_group 

aaron@luna ~> grep -r 'rtprio\|memlock' /etc/security/limits.d/

/etc/security/limits.d/25-pw-rlimits.conf:@pipewire   - rtprio  70
/etc/security/limits.d/25-pw-rlimits.conf:@pipewire   - memlock 4194304
/etc/security/limits.d/95-jack.conf:@jackuser - rtprio 70
/etc/security/limits.d/95-jack.conf:@jackuser - memlock 4194304
/etc/security/limits.d/95-jack.conf:@pulse-rt - rtprio 20
/etc/security/limits.d/realtime.conf:@realtime       -       rtprio          99
/etc/security/limits.d/realtime.conf:@realtime       -       memlock         unlimited

sudo usermod -a -G realtime aaron

logout and login

then

groups aaron

--

Simultaneous Multithreading (SMT, also called hyper-threading) is enabled. This can cause spikes in DSP load at higher DSP loads. Consider disabling SMT when experiencing such spikes with 'echo off | sudo tee /sys/devices/system/cpu/smt/control'. See also https://wiki.linuxaudio.org/wiki/system_configuration#simultaneous_multithreading

Check if smt enabled

cat /sys/devices/system/cpu/smt/control
on

To disable temporariy

echo off | sudo tee /sys/devices/system/cpu/smt/control
off

To make persistent across boots

 sudo nano /etc/default/grub

GRUB_CMDLINE_LINUX="rhgb quiet nosmt"

Then update grub

sudo grub2-mkconfig -o /etc/grub2.cfg
Generating grub configuration file ...
Adding boot menu entry for UEFI Firmware Settings ...
done

then sudo reboot

--

The scaling governor of one or more CPUs is not set to 'performance'. You can set the scaling governor to 'performance' with 'cpupower frequency-set -g performance' or 'cpufreq-set -r -g performance' (Debian/Ubuntu). See also https://wiki.linuxaudio.org/wiki/system_configuration#cpu_frequency_scaling

sudo cpupower frequency-set -g performance

--

Kernel 6.15.8-200.fc42.x86_64 without 'threadirqs' parameter or real-time capabilities found. See also https://wiki.linuxaudio.org/wiki/system_configuration#do_i_really_need_a_real-time_kernel

sudo nano /etc/default/grub


GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rhgb quiet nosmt threadirqs"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true

sudo grub2-mkconfig -o /etc/grub2.cfg
