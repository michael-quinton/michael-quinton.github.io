---
title: "Switching to Linux: A Windows Gamer's Journey"
description: "How I went from a sketchy bootloader hack to a full-time Kubuntu setup — and why I haven't looked back."
date: 2026-03-30T17:23:55+02:00
tags: ["linux", "kubuntu", "gaming", "dual-boot"]
author: "Michael Quinton"
draft: false
---

Recently I decided to make the big move to Linux on my home machine — the one I use for gaming, personal projects, entertainment, the lot. I say "made the move"; technically it's a dual boot and Windows is still sitting there in hibernation. But to my surprise, I haven't jumped back to it in a while.

## Gaming on Linux

The reason I kept Windows around was that some games I like to play don't support Linux, largely because of their anti-cheat software. PUBG is a good example, along with a lot of EA titles. Funnily enough, EA were recently hiring a lead engineer to spearhead the development of making their anti-cheat compatible with Linux. I don't know enough about this area in detail, but I do know that in PUBG's case, BattleEye is compatible with Linux — it's just that the game doesn't use that specific version. From what I vaguely recall reading somewhere, the devs simply didn't want to update it so it would be playable on Linux.

That said, the support out there is bigger than you might think. Jump over to [ProtonDB](https://www.protondb.com/) and you can see how well games run across various distributions. I've played a fair few myself — mostly smaller indie titles like Terraria or that new survival horror game R.E.P.O. — and they've been working well. I'd like to try something more graphically intensive, but saying my hardware is "quite dated" is an understatement, so that'll have to wait for another day.

## The Installation (A.K.A. The Sketchy Way)

Now, I didn't have a bootable USB and I was in one of those "I want to do this *now*" moods. So I went about it in a very roundabout way: I used a piece of software that would modify the boot configuration. It was sort of like GRUB2, but not quite. An interesting tool — quite sketchy, to be honest — but it did the job. I think it was grub2win; if you Google it, you'll see what I mean by sketchy. Honestly, I got quite lucky not to break my Windows installation.

The tool let you boot from an ISO, and I tried many. Originally I wanted to use Fedora, then I tried Pop!\_OS, and so on — but none of them worked. You had to do a lot with the configuration yourself. It looked something like this:

```
menuentry "Ubuntu ISO 25.10" --class ubuntu --class icon-ubuntu {
  set iso=/ISOs/ubuntu-25.10-desktop-amd64.iso
  set bootparms=`boot=casper iso-scan/filename=$iso quiet splash`

  search -f $iso --set=root
  echo root=$root
  loopback loop $iso
  linux (loop)/casper/vmlinuz $bootparms
  initrd (loop)/casper/initrd
}
```

> **Note:** I don't actually know if this config works as-is, so don't blindly copy it — it's just to give you an idea of what the process looked like.

The only ISO I could actually get to work was Ubuntu. At that point I thought, fine, let's just go with it and I'll do a proper USB install later — or just leave it.

## Settling In

Once I was up and running, one of the first things I did was set up Timeshift for snapshots. I was terrified of breaking something. There was a whole "ricing" phase where I thought it'd be fun to customise everything, but it ended up being more intensive than I expected, so I reverted back to my initial snapshot and stuck with a vanilla setup for the time being.

I installed the essentials — VS Code, web browsers, and the usual suspects — then gradually started changing things up. I switched to Kubuntu, set up zsh with Powerlevel10k, started using LazyVim and LazyGit, and honestly, I'm having a great time.

## My Current Setup

Here's what I'm running these days:

```
mike@linux
----------
OS: Kubuntu 24.04.4 LTS (Noble Numbat) x86_64
Kernel: Linux 6.17.0-19-generic
Uptime: 8 hours, 56 mins
Packages: 3038 (dpkg), 13 (flatpak-system), 19 (flatpak-user), 17 (snap)
Shell: zsh 5.9
Display (24G2W1G4): 1920x1080 in 24", 60 Hz [External]
Display (PLE2483H): 1920x1080 in 24", 60 Hz [External]
DE: KDE Plasma 5.27.12
WM: KWin (X11)
WM Theme: Dracula
Theme: Breeze (Dracula) [Qt], Breeze [GTK2/3]
Icons: dracula-icons-circle [Qt], dracula-icons-circle [GTK2/3/4]
Font: Noto Sans (10pt) [Qt], Noto Sans (10pt) [GTK2/3/4]
Cursor: Dracula (24px)
Terminal: konsole 23.8.5
CPU: AMD Ryzen 5 2600 (12) @ 3.40 GHz
GPU: NVIDIA GeForce GTX 1650 [Discrete]
Memory: 7.78 GiB / 15.54 GiB (50%)
Swap: 1.31 GiB / 4.00 GiB (33%)
Disk (/): 53.34 GiB / 108.44 GiB (49%) - ext4
Disk (/media/mike/684C56CA4C569324): 102.70 GiB / 111.68 GiB (92%) - ntfs3
Disk (/mnt/linux-storage): 6.33 GiB / 457.37 GiB (1%) - ext4
Local IP (enp3s0): 192.168.0.167/24
Locale: en_GB.UTF-8
```

Nothing fancy hardware-wise — the Ryzen 5 2600 and GTX 1650 have been holding up well enough — but I'm really happy with how the software side has come together. The Dracula theme across everything gives it a cohesive feel, and between zsh, LazyVim, and KDE Plasma, the workflow is genuinely enjoyable.

If you're thinking about making the switch, I'd say just go for it. Worst case, you keep Windows on a spare partition — like me — and never end up going back.
