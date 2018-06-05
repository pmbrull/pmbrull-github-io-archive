---
layout: single
title: "Just installed Elementary OS"
date: 2018-06-05
categories: miscellaneous
tags: linux elementary os bug freeze fix grub
author_profile: false
mathjax: true
---

Copy-pasting the title: just formatted my laptopt a few days ago and got myself [Elementary OS](https://elementary.io). Had been using dual boot Windows 10 and Ubuntu 16.04 for a while and got quite tired of them, as it was somehow slow and felt sloppy. I have to say that Elementary OS just feels good: its **design** is really impressing and goes incredibly **smooth**. Also, being a unix distribution, getting ready to play around with softwares like Spark or Airflow is faster than a blink of an eye and saves you the pain of trying to do that on Windows (which I have done and do not recommend).

However, a quick look around Ubuntu forums and Stack Community is enough to say: Nvidia is not friends with Linux, so I needed to do some workaround and fixing before being completely comfortable.

## Preparing the Installation

In this scenario, graphic's card inconsistency with software hanged the pc on the glowing distro logo. How to overcome this? Hold ```shift``` while booting is starting and edit **grub** with ```-e``` adding: 

```bash
nomodeset
```

at the end of the line saying *quiet splash*. Just in case, you can also remove the *quiet splash* keywords in case booting hangs, to be able to check what was going on.

## Timeshift

Once everything is running, you have some software installed and spent a few hours configuring stuff, fear of failure strives in case you mess something up horribly and need to start again. Better be prepared, 'cause mistakes happen:

```bash
sudo apt-add-repository -y ppa:teejee2008/ppa
sudo apt-get update
sudo apt-get install timeshift
```

**Timeshift** is an awesome tool that lets you create system backups whenever you want and also schedule them. I have done like 5 recoveries in a couple of days and I have to say it saves a lot of time and gives extra peace of mind when getting to a somehow stable checkpoint.

## Shutdown and reboot freeze

This has been my nemesis for the last two days and the reason why I hardly got any sleep reading posts and pressing the power button again and again (please forgive me). Imagine the sound your computer makes when you have to force the shutdown and can feel how something has broken inside. So well, after going up and down installing nvidia drivers and hearing the graphics card making weird noises + desktop breaking, decided it was not the best path to follow, altough half of the posts got solved with that strategy.

Instead, I focused on GRUB parameters, and from all I tried **acpi=force** is what fixed the problem:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="nosplash acpi=force"
```

I added *nosplash* to see what is going on in case another problem appears.

Hope it helps someone ;)