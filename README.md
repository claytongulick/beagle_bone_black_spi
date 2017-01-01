# Working with SPI on the Beagle Bone Black
This document is for maintaining current information on making SPI work on the beagle bone black and beagle bone black wireless.

As of the time of this writing, the documentation on this is fragmented across the internet and mosly out of date. In an effort to save others some time, I'll document a little bit about how all this works, the history to help explain where the other examples on the internet are coming from, and the current (very easy) method of managing pins on the beagle bone black.

I'm posting this here hoping to save others some frustration/pain that I've gone through trying to get SPI working on the beagle bone black wireless.

If you see any errors or omissions, please create an issue or a PR, and I'll get the information merged in/fixed.

## I'm so confused...
Ok, for starters when you googled this, you've probably found a bunch of information on dts, overlays, capemgr, slots, etc... these terms can be confusing for someone just getting started with embedded linux, but once we go through the history and evolution it'll make a lot more sense.

## History
At its core, the job of an operating system is to provide a layer of abstraction between running applications and hardware (as well as scheduling execution). When you develop with something like an arduino, or an AVR microcontroller, you don't use an OS, you just write firmware that gets directly executed by the microcontroller. This is pretty simple, and great for small projects but as requirements grow, it's advantageous to have a full OS running - especially if you care about portability. The beagle bone black uses Linux, by default.

Linux traditionally used kernel modules (drivers) for every piece of supported hardware in order to provide the abstraction layer. This strategy didn't work very well when all these ARM devices like the beagle bone started showing up because there were so many devices with different configurations, and it really didn't make sense to add all that to the kernel. As a solution to this, Linus came up with this idea for a more generic system called a device tree.

### Device Tree
The device tree is basically a way to describe the mapping and purpose of physical hardware to the kernel. This is done via a 'dts' file, which is a source code file that lists the specific properties of the hardware. This source code file is compiled into a binary that the kernel can understand, a 'dtb' or 'dtbo' file. So, early on in beagle bone history, this was how things were done. There were lots of dts and dtbo files that were made for all sorts of different purposes, and you, as the user could swap these out depending on how you want pins configured. You can also create your own. This is where some of the older articles you see that have instructions about creating a dts file and compiling it to enable SPI come from.

Well, that whole thing was pretty spiffy, but there were some drawbacks. One, it wasn't very approachable for new folks. You basically had to learn a new language and toolchain just to configure pins. While better than writing a kernel module, that still wasn't great. Also, all of this configuration happened at boot time, so every time you wanted to make a change you had to reboot. This really doesn't work well for a device where you want to be able to hot swap extension boards and reconfigure things at runtime. Third, this all happened in kernel space, which as an industry we try not to do. It's better to keep as much as possible in user space.

### Cape Manager
To address those issues: enter the Cape Manager. The cape manager is a pretty fancy piece of kernel module software that has the ability to dynamically load and swap out device tree overlays, and the tools live in userspace. When you see instructions on the web about adding lines to uEnv.txt like:
optargs=quiet drm.debug=7 capemgr.enable_partno=BB-SPIDEV0

This is telling the cape manager to load the SPI device tree overlay at boot time. Everywhere you look on the internet, this is the recommended solution for enabling SPI on 'current' devices. But, it doesn't work.

Why? Well, to explain that requires one more step.

### Universal IO
Even though the cape manager is neat software from an engineering perspective, and really accomplished its goals well, it still leaves something to be desired from a new user perspective. Folks who are just getting into the whole maker scene are reasonably confused by all this.

To address that, some new software was created (which is enormously fancy), called universal io.

Basically what this is, is a device tree overlay that's loaded by the cape manager at boot time that has the ability to dynamically configure all of the pins at runtime using a tool called config-pin.

You can see it and read more about it here: https://github.com/cdsteinkuehler/beaglebone-universal-io

So, with this utility all of the pins that aren't reserved for HDMI can be hot configured by using the simple config-pin command, and this includes SPI!

### Ok, so can we just make it work now please?
So, finally after that long bit of history, here's how you actually set up and use SPI on a new beagle bone black wireless with a current image:

#data out
config-pin P.18 spi
#clock out
config-pin P.22 spi

Rinse, repeat if you need other pins like CS, or MISO.

Once you do this, you should be able to write SPI from your language of choice, and be able to measure a reliable wave form with your oscope.

One thing to note, is that the above commands need to be executed each time the device boots, since the default pin configuration is loaded by the cape manager when the universal io device tree overlay is applied.



