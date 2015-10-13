# Flash an alpha C.H.I.P. (v0x21) from NixOS

Resources:

- [Installing CHIP-SDK](https://nextthingco.zendesk.com/hc/en-us/articles/210863457-Installing-C-H-I-P-SDK-)
- [Flashing Guide](https://nextthingco.zendesk.com/hc/en-us/articles/210864097-Flash-C-H-I-P-from-C-H-I-P-SDK-Virtual-Machine-) from SDK VM
- [Flashing Guide](https://nextthingco.zendesk.com/hc/en-us/articles/209757858-Flash-C-H-I-P-with-NTC-buildroot-Ubuntu-) from Ubuntu (useful setup and debugging tips)
- [NextThing BBS thread](https://bbs.nextthing.co/t/unable-to-flash-alpha-c-h-i-p/544/47) about issues flashing the CHIP

These notes describe what I did to get the CHIP SDK VirtualBox thing running and able to flash my CHIP. Running the tools directly on NixOS would be cooler, and maybe I'll get around to it, but at least for now _this works_.

## 1. NixOs Dependencies

### VirtualBox

The USB requirements for flashing mean we get to deal with extra joy thanks to Oracle's licensing of their extensions for VirtualBox.

**Key resource**: [NixOs Wiki on virtualBox](https://nixos.org/wiki/Installing_VirtualBox_on_NixOS) (read it first)

The wiki is a little vague on how to install the extensions, so here is a little more detail:

- You have to match the extensions version number to the version of VirtualBox you'll be installing. nixpkgs is at 5.0.4 at the moment.
    - I did have some git checkout errors that I think were due to symlink issues specific to this version of virtualbox. I ignored them and flashed successfully, but it's probably worth fixing...
- Go to http://download.virtualbox.org/virtualbox/&lt;YOUR_VERSION_NUMBER&gt;/, eg http://download.virtualbox.org/virtualbox/5.0.4/, and copy the link for `Oracle_VM_VirtualBox_Extension_Pack-VERSION.NUMBER-STUFF.vbox-extpack`
- Pull it to your nix store with
  ```bash
  $ nix-prefetch-url http://download.virtualbox.org/virtualbox/5.0.4/Oracle_VM_VirtualBox_Extension_Pack-5.0.4-102546.vbox-extpack`
  ```
  (substituting the link you copied)

Then you can edit your nix config as described by the wiki, and installing VirtualBox should build from source with the extension. **This takes a long time**

**Don't forget** to add yourself to the group **`vboxusers`** (see [wiki](https://nixos.org/wiki/Installing_VirtualBox_on_NixOS#Enabling_VirtualBox)) so the SDK VM can access CHIP over USB.

Kernel modules, group changes... it's probably time for a reboot after getting VirtualBox going.

### Vagrant, Git

Install in your usual NixOs way, it should just work.


## 2. Install the SDK

Follow the [Installing CHIP-SDK Guide](https://nextthingco.zendesk.com/hc/en-us/articles/210863457-Installing-C-H-I-P-SDK-), starting at [Clone the CHIP-SDK Git repository](https://nextthingco.zendesk.com/hc/en-us/articles/210863457-Installing-C-H-I-P-SDK-#user-content-clone-the-chip-sdk-git-repository).

## 3. Flash it!

The [Flash C.H.I.P. from C.H.I.P. SDK](https://nextthingco.zendesk.com/hc/en-us/articles/210864097-Flash-C-H-I-P-from-C-H-I-P-SDK-Virtual-Machine-) guide _almost_ nails it, with one small thing to note:

In the section, **To Flash C.H.I.P. with the NTC buildroot image...**, it says to `cd` to `~/CHIP-tools`. This should be **`cd ~/CHIP-SDK/CHIP-tools/`**. Other than that, it worked for me!
