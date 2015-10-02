# Introduction

"Snake" or "worm" is a game where you pilot a snake or worm around an arena,
eating fruit as you go. The controls are simple; the snake keeps moving in
the same direction unless you press up/down/left/right, which cause the
snake's head to turn and move in the relevant direction. The snake's body
always follows the path the head took. As fruit is eaten, the snake gets
longer. You lose the game if the snake eats itself, crashes into the edge of
the arena, or eats a moldy fruit.

On its own, this game is fun. However, this project adds a twist: The four
control buttons are physically separate and controlled by four different
people, who must co-ordinate their actions to play the game.

We'll host the game on a Raspberry Pi 2 running the Raspbian distribution. A
version of snake is included in the standard package repository. We'll develop
large physical buttons to use as the input mechanism, the wiring to connect
them to the Pi, and software/configuration to make the buttons show up as a
regular keyboard to control the game.

# BOM

https://docs.google.com/spreadsheets/d/1XbSBI3CyhftJEIavfGvj2m_YPL8bZqrddurS3Z7UsZc/edit#gid=0

# Hardware

The buttons will be simple push-to-make switches. Each switch will either make
no contact when not pressed, or connect a specific GPIO to 3.3V. The Pi will
provide an internal pull-down to ensure that the GPIO reads a known value when
the switch is not connected.

We simply need to connect four pairs of wires to the RPi's expansion
connector; one side of each pair to 3.3V and the other to a specific GPIO.
For testing we'll just hack something together. For "production", we may make
a simple PCB and solder a few connectors onto it, simply to make it easier to
hook everything up easily, and share the 3.3V wire amongs all the buttons.

For details on the expansion connector pinout, see
https://github.com/raspberrypi/documentation/blob/master/hardware/raspberrypi/schematics/Raspberry-Pi-B-Plus-V1.2-Schematics.pdf

The following GPIO assignments will be used:

| GPIO | Key name | Linux Key Code Define | Linux Key Code Number |
| ---- | -------- | --------------------- | --------------------- |
| 17   | Up       | KEY_UP                | 103                   |
| 18   | Down     | KEY_DOWN              | 108                   |
| 22   | Left     | KEY_LEFT              | 105                   |
| 23   | Right    | KEY_RIGHT             | 106                   |

These GPIOs appear at the exact same location on all Pi models.

For a reference to the key code names and values, see
`include/dt-bindings/input/input.h` in the Linux kernel source tree, e.g.
https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/include/dt-bindings/input/input.h .
Ideally, we'd simply include that file in our `.dts` file. However, we'd then
require a Linux kernel source tree to compile our device tree (see below).

The image file used to engrave the foot switches is `arrow.eps` in this
repository.

# Software

## Boot config for fixed display resolution

The snake game in the Raspbian distro appears to create a window with a fixed
hard-coded pixel size. Luckily, this fits well into the 1024x768 resolution
of our projector. If the Pi doesn't pick this resolution by default, or you're
using a different display device with a different resolution and need to force
the output resolution, simply edit `/boot/config.txt` on the SD card, and add
the following lines:

```
hdmi_group=2
hdmi_mode=16
```

With the HDMI to VGA converter that we're using, HDMI hotplug detection does
not work correctly; the Pi will not detect an attached HDMI monitor and will
fall back to composite video output instead. In order to force the Pi to use
its HDMI output, place the following in `/boot/config.txt` (it's probably
already present; so you can simply uncomment it):

```
hdmi_force_hotplug=1
```
 
## Software installation

Grab the Raspbian install image from
https://www.raspberrypi.org/downloads/raspbian/ and follow the installation
instructions there.

Once the system is up and running, you'll need to install a few extra
packages. Run:

```
sudo apt-get install snake4 xfonts-base device-tree-compiler evtest
```

## Keyboard emulation

The Linux kernel contains a device driver that emulates a keyboard using GPIOs
(input signals). We'll instantiate this using a Device Tree overlay. What is
a Device Tree overlay? See
https://www.raspberrypi.org/documentation/configuration/device-tree.md or
search Google for "Device Tree overlay" to receive many answers!

The file `fcch-gpio-keys.dts` in this git repository is the DT overlay source.
To compile this into a binary image, run `fcch-gpio-keys-build.sh` in this git
repo. To install this onto the Pi, copy the resultant `fcch-gpio-keys.dtb` to
`/boot/overlays/`, and edit `/boot/config.txt` to add `fcch-gpio-keys` to the
`dtoverlay` line (which you may need to add if not already present). The line
will look like:

```
dtoverlay=fcch-gpio-keys
```

Now wire up the buttons, reboot, and run `evtest` to test them out.

A reference for the gpio-keys driver may be found at
https://www.npmjs.com/package/gpio-button

Note that `config.txt` appears to use `dtoverlay` not `device_tree_overlay`
now.
