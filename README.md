# Gamecube-Adapter-Slippi-Launcher-Linux-Install

A guide for getting the USB GameCube adapter working with slippi and dolphin on Linux systems

(Will Probably work with Fedora, Arch. Likely other distros too. Haven't checked)

Most of this guide is taken from this [reddit post](https://www.reddit.com/r/linux_gaming/comments/rdt87u/slippi_super_smash_bros_melee_guide_on_fedora_35/) written by a deleted user. I didn't have the crashing issues decribed in step 1 so I skipped that. The one change to this guide is that I have substituted `--no-sandbox` for `--device=all`

## Install Slippi Launcher

I had to do a lot of work arounds to get Slippi working on Fedora 35 as it seems to assume it will be running on Ubuntu.

### Step 1 - AppImage

Download the AppImage from https://slippi.gg/ (Linux should already be selected in the drop down)

in terminal navigate to the AppImage location (usually Downloads) and `run chmod +x <AppImage name>` (for example `chmod +x Slippi-Launcher-2.1.7-x86_64.AppImage`)

Run with `./Slippi-Launcher-2.1.7-x86_64.AppImage --device=all` (I got errors without that - it seems to weaken security and is related to an out of date bundled chrome/electron dependency, use at your own risk). Try it once without `--device=all` just to see if it runs. The errors I got were Invalid asm.js: Invalid member of stdlib and The display compositor is frequently crashing. Goodbye.

Login to your account etc using the prompts. When you click the green "Play" button, watch the terminal output. You should see a line that says something like Launching dolphin at path: /home/petertree/.config/Slippi Launcher/netplay/Slippi_Online-x86_64.AppImage - Note that path down

At the time of writing, libgio on Fedora is too new for the AppImage. However running it with `LD_PRELOAD=/lib64/libgmodule-2.0.so "/home/petertree/.config/Slippi Launcher/netplay/Slippi_Online-x86_64.AppImage"` opened Dolphin OK. (If I ran it without the LD_PRELOAD I got the error /lib64/libgio-2.0.so.0: undefined symbol: g_module_open_full)

### Step 2 - Controller

I have the 4-port Mayflash Adapter with the switch in Wii U mode.

If running, close Dolphin

Using sudo, create a file called `/etc/udev/rules.d/51-gcadapter.rules` and paste `SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", ATTRS{idVendor}=="057e", ATTRS{idProduct}=="0337", MODE="0666"`

Reload udev rules with: `sudo udevadm control --reload-rules`

Connect the Mayflash adapter into a USB 3.0 (Blue/"SS") port and open dolphin with the above command `./Slippi-Launcher-2.1.7-x86_64.AppImage --device=all` 

Step 3: Fixing the Polling Rate

The polling rate is 125Hz-ish by default. Use https://github.com/HannesMann/gcadapter-oc-kmod to fix that (`git clone https://github.com/HannesMann/gcadapter-oc-kmod ; cd gcadapter-oc-kmod; make && sudo insmod gcadapter_oc.ko`)

If that worked, you can set the polling rate to 1000Hz by running `echo 1 | sudo tee /sys/module/gcadapter_oc/parameters/rate` (1 is the interval, 2 would be 500Hz, 4 would be 250Hz etc.)

Open Dolphin and see if the Adapter is polling at 1000 Hz (Controller icon or Options, Controller Settings, Configure tab)

Misc.

Remember to run `sudo insmod /path/to/gcadapter_oc.ko` before you run Dolphin everytime you reboot. You could also make a script in `~/.local/bin/slippi`

`#!/bin/bash
 sudo insmod /path/to/gcadapter_oc.ko
 LD_PRELOAD=/lib64/libgmodule-2.0.so "/path/to/Slippi_Online-x86_64.AppImage`

You could also make a Desktop Entry so it shows up in the Applications.

Click on the Controller icon or go to Options, Controller Settings and click on Configure next to any of the GameCube Controller entries (like Port 1: GameCube Adapter for Wii U)

See if it says Adapter Detected - if it does, note the polling rate

If it doesn't, try unplugging and replugging, make sure the adapter is set to Wii U mode, try plugging in both the black and gray USB cables, try switching from Wii U to PC and back while plugged in and restarting dolphin - the last one worked for me.


