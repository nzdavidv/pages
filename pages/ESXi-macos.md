---
title: "Installing Mac OS on a VM on ESXi"
date: 2025-08-11
---
# Overview
Just because you can doesn't mean you should. Technically yep it worked and I wound up with a VM running macOS Sonoma.

It's not super painfully slow but it's slow enough to be annoying. 
It certainly doesn't spark joy.. and for me it required having a physical Mac OS machine to get through the first step which is quite a hurdle.

But I guess if you needed an Apple VM for managing apple devices or some reason then yep it works.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/macosvm1.png" alt="macosvm" width="800px"></kbd>

### The 15 thousand step journey
This entire post is a refresh and short version of the really well written post <a>https://www.nakivo.com/blog/run-mac-os-on-vmware-esxi/</a>
Steps:
1. Download macOS
2. Creating a bootable macOS ISO
3. Preparing ESXi Host
4. Creating and Configuring new macOS VM
5. Installing Mac OS on the VM
6. Install VMware Tools


