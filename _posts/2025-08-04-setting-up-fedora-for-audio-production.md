---
title: Setting up Fedora Linux for audio production
description: >-
  In this post I will document setting up Fedora for audio production. I'll look at installing Bitwig, and optimising the system for audio. I'll also look at running VSTs via yabridge. 
author: aaron
date: 2025-08-04 00:00:00 +0000
categories: [Music Production]
tags: [linux, audio]
pin: true
media_subpath: '/posts/20180809'
---

## Installing Bitwig

Installing Bitwig on Fedora requires converting the official Ubuntu package into RPM format so it can be installed on Fedora. 

[Download Bitwig](https://www.bitwig.com/download/), use `alien` to convert the `.deb` package, then `dnf install` the resulting RPM package.

```sh
sudo alien -r bitwig-studio-5.3.12.deb
sudo dnf install ./bitwig-studio-5.3.12-2.x86_64.rpm
```