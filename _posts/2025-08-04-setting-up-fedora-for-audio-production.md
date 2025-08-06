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

Now we'll optimise the system for audio. Firstly, we'll install `rtcqs` which is a utility diagnosing potetial issues relating to audio performace.

As a prerequisite, ensure that `python3-tkinter` is installed.


```bash
sudo dnf install python3-tkinter
```

```bash
mkdir -p ~/path/to/rtcqs
cd ~/path/to/rtcqs
python3 -m venv venv &&
. venv/bin/activate &&
pip install --upgrade rtcqs
```

then

```bash
rtcqs_gui
```

Later on, if you come back to `rtcqs` you will need to activate the virtual environment again.

```bash
cd ~/path/to/rtcqs
. venv/bin/activate
rtcqs_gui
```

`rtcqs` found a range of potential issues on my system. I've recorded here the issues it found, and the fixes which I've applied. YMMV. 

### Group limits

> User aaron is currently not member of a group that has sufficient rtprio (0) and memlock (8388608) set. Add yourself to a group with sufficent limits set, i.e. audio or realtime, with 'sudo usermod -a -G <group_name> aaron. See also https://wiki.linuxaudio.org/wiki/system_configuration#audio_group 
{: .prompt-warning }

My current user was not assigned to a group with sufficient rtprio and memlock set. Run the following command to list groups with rtprio and memlock limits:

```bash
$ grep -r 'rtprio\|memlock' /etc/security/limits.d/

/etc/security/limits.d/25-pw-rlimits.conf:@pipewire   - rtprio  70
/etc/security/limits.d/25-pw-rlimits.conf:@pipewire   - memlock 4194304
/etc/security/limits.d/95-jack.conf:@jackuser - rtprio 70
/etc/security/limits.d/95-jack.conf:@jackuser - memlock 4194304
/etc/security/limits.d/95-jack.conf:@pulse-rt - rtprio 20
/etc/security/limits.d/realtime.conf:@realtime       -       rtprio          99
/etc/security/limits.d/realtime.conf:@realtime       -       memlock         unlimited
```

I could see that the `@realtime` group has the appropriate `rtprio` and `memlock` limits. To fix this issue I had to assign myself to the `realtime` group: 

```bash
sudo usermod -a -G realtime aaron

# this will reboot the system (needed to apply changes)
sudo reboot
```

Once the system is rebooted, we can double check to see if the group change has been applied:

```bash
$ groups aaron

aaron : aaron wheel video docker realtime
```

### Simultaneous Multithreading

> Simultaneous Multithreading (SMT, also called hyper-threading) is enabled. This can cause spikes in DSP load at higher DSP loads. Consider disabling SMT when experiencing such spikes with 'echo off | sudo tee /sys/devices/system/cpu/smt/control'. See also https://wiki.linuxaudio.org/wiki/system_configuration#simultaneous_multithreading
{: .prompt-warning }

To check if smt is enabled, run the following command:

```bash
cat /sys/devices/system/cpu/smt/control

# command output
on
```

We can see in the output that `smt` is set to `on`.

To disable temporariy `smt` run the following:

```bash
echo off | sudo tee /sys/devices/system/cpu/smt/control

# output
off
```

If you want to make this change persistent across reboots, we have to update `grub` config. 

```bash
sudo nano /etc/default/grub
```

Find the `GRUB_CMDLINE_LINUX` line, and add `nosmt`


```bash
GRUB_CMDLINE_LINUX="rhgb quiet nosmt"
```

Save and exit from `nano` (`CTRL+O` then `CTRL+X`).

Then update run the following command to update the `grub` configuration file:

```bash
sudo grub2-mkconfig -o /etc/grub2.cfg

Generating grub configuration file ...
Adding boot menu entry for UEFI Firmware Settings ...
done
```

Then `sudo reboot` to reboot the system.

### CPU Scaling

> The scaling governor of one or more CPUs is not set to 'performance'. You can set the scaling governor to 'performance' with 'cpupower frequency-set -g performance' or 'cpufreq-set -r -g performance' (Debian/Ubuntu). See also https://wiki.linuxaudio.org/wiki/system_configuration#cpu_frequency_scaling
{: .prompt-warning }

For this one I had to set the performance governour of my CPU to `performace`. 

Common governors include performance (highest frequency), powersave (lowest frequency), ondemand (dynamic based on load), conservative (similar to ondemand but with more gradual changes), and userspace (allows user-defined frequencies).

To set temporarily, use the following command:

```bash
sudo cpupower frequency-set -g performance
```

To persist this change, we will need to create a systemd service which runs on boot.

> I haven't set this up on my machine as I don't require `performance` mode to be activated all the time and doing so causes my machine to heat up a fair bit. I prefer to keep my machine running as cool as possible most of the time, so will manually set the governor as required. 
{: .prompt-info }

Since the command requires sudo and is related to system power management, it’s best to create a systemd service to ensure it runs at boot.

First, you'll need to create a systemd service that runs this command when the system boots.

Open a terminal and create a new service file:

```bash
sudo nano /etc/systemd/system/cpupower-performance.service
```

Add the following content to the file:

```bash
[Unit]
Description=Set CPU governor to performance

[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower frequency-set -g performance
User=root
Group=root
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```
This service will run the cpupower command to set the governor to performance during the boot sequence.


Reload systemd and enable the service:

After saving the service file, reload the systemd manager and enable the service to start on boot:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cpupower-performance.service
```

Then start the service

```bash
sudo systemctl start cpupower-performance.service
```

Now, the CPU governor will automatically be set to performance on every boot.

### Preempt RT

> Kernel 6.15.8-200.fc42.x86_64 without 'threadirqs' parameter or real-time capabilities found. See also https://wiki.linuxaudio.org/wiki/system_configuration#do_i_really_need_a_real-time_kernel
{: .prompt-warning }

Threaded IRQs (Interrupt Requests) in the Linux kernel allow interrupt handling to be split into two parts: a "hard IRQ" handler that runs in interrupt context (with minimal work) and a "threaded IRQ" handler that runs in a kernel thread context, enabling more complex operations. This separation helps minimize the time interrupts are disabled, improving system responsiveness and simplifying interrupt handling, according to the Linux Device Drivers book.

```bash
sudo nano /etc/default/grub
```

Update `GRUB_CMDLINE_LINUX` to include `threadirqs`

```bash
GRUB_CMDLINE_LINUX="rhgb quiet nosmt threadirqs"
```

Then make the `grub` configuration file and reboot:

```bash
sudo grub2-mkconfig -o /etc/grub2.cfg
sudo reboot
```

### Spectre/Meltdown Mitigations

> Kernel with Spectre/Meltdown mitigations found. This could have a negative impact on the performance of your system. See also https://wiki.linuxaudio.org/wiki/system_configuration#disabling_spectre_and_meltdown_mitigations
{: .prompt-warning }

This one I am leaving as-is. I don't want to open up my system to potential cyber attacks. For systems which are completely offline, I might consider disabling mitigations, but for the relatively minor performance boost, and considering the amount of third-party code I run on my machine, I'm happy to leave the mitigations in place. 

### Swappiness

> The following mounts should be avoided for audio purposes: /boot. See also https://wiki.linuxaudio.org/wiki/system_configuration#filesystems
{: .prompt-warning }

The solution for this one is pretty straight-forward. We'll need to create a new system configuration file to override the swappiness with a lower value.

```
sudo nano /etc/sysctl.d/99-custom-sysctl.conf
```

Then add the following contents:

```
vm.swappiness=10
```

Then `sudo reboot` to apply the changes.

## Install WineTKG and yabridge

If you're planning on using Windows VSTs, you'll need to install Wine and yabridge. There's an excellent copr repo containing all the steps required to get this working.

https://copr.fedorainfracloud.org/coprs/patrickl/wine-tkg/

> In this section about `pipewire-winasio` you need to run `wine prefix` to generate the `WINEPREFIX`. It took me some time to figure that out, so I'm sharing it here:
{: .prompt-info }

```
wine prefix

/home/aaron/.wine
```

Then use the returned path to set the `WINEPREFIX` in the subsequent commands, e.g., 

```
env WINEPREFIX=/home/aaron/.wine /usr/bin/wineasio-register
```
