---
title: Dual-booting Linux Mint
date: 2019-11-08
modified: 2019-11-08
categories: [Linux]
excerpt: How and why I installed Linux Mint and chose to dual-boot between it and Windows
tags: [Linux]
classes: wide
header:
  overlay_image: /assets/images/linux-dual-boot/header.jpg
  overlay_filter: 0.5 # same as adding an opacity of 0.5 to a black background
  caption: "Photo by [Marko Blažević](https://unsplash.com/@kerber) on [Unsplash](https://unsplash.com)"
---
## Why Linux
I've always heard that Linux is the way to go but I never tried it. I had Windows and it worked fine for me. I took some training at work that required Linux so I started using it inside a virtual machine. I got comfortable with it and decided it would be fun to try at home.

## Why Mint
Based on this [Dev.to](https://dev.to/pluralsight/which-distribution-of-linux-should-i-use-51g7) article it sounded like where a Linux newbie like myself should start. I tried a couple versions inside VirtualBox before committing. I used [OSBoxes](https://www.osboxes.org/) to quickly get them up and running.

## Why Dual-Boot
I chose to dual-boot because I didn't want to risk losing Windows if I messed up the Mint install. Also because the Mint install made it really easy.

## How I did it
### Disclaimer!
I'll recount the steps I took and the references I used but can't guarantee any of it for anyone else.
Also it's a good idea to follow along with [Mint's install docs](https://www.linuxmint.com/documentation.php).

### 1. Back Up Data
I backed up my data because there's always a chance it could get wiped from existence.

### 2. Download Linux Mint
I grabbed the 64bit Cinnamon version from [here](https://linuxmint.com/download.php).

### 3. Create a Bootable USB
I used [Etcher](https://www.balena.io/etcher/) to flash the image onto my USB drive but any flashing software should do the trick.
![Etcher](/assets/images/linux-dual-boot/etcher.png)

### 4. Create Disk Space
My first attempt didn't take because I didn't have any room. I ended up freeing up some space from my Windows partitions.
![Disk Management](/assets/images/linux-dual-boot/disk.png)

### 5. Update Boot Configuration
I had to disable secure boot and change the boot order in the BIOS.

### 6. Install Mint
I followed the on-screen instructions at this point. Here are the important bits:
* Dual booting with Windows
* Create partitions
* * Root (I used 20Gb)
* * Swap (I used 8Gb)
* * Home (I used the rest of my free space)
A few more on-screen instructions and I was ready to go!

### 7. Use Mint
Mint is installed and ready to go. I'm on a Razer Blade Stealth and everything works out of the box except for closing the lid. I'm sure there are other things that don't quite work that I haven't encountered yet. When I close the lid Mint is supposed to suspend but when I open the lid back up I have to hard shutdown before my laptop will wake up and respond.
Other than that I'm very happy with Mint and hope that this article helps you!
