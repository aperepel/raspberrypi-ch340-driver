# CH340/341 UART Driver for Raspberry Pi

A pre-compiled binary for CH340/341 (HL340/341) USB-to-Serial UART Driver for Raspberry Pi. This is a very common chip, especially in those millions of esp8266-based dev boards (also often referred to as NodeMCU). Raspberry Pi doesn't have an out of the box driver for it, so plugging your device into RPI's USB won't do much other than power it. I found it surprisingly difficult to locate working instructions and binaries, hence this repository came to life.

The driver will work both for CH340 & CH341 chips. Sometimes you will see the HL340/HL341 designation, it's just different names for the same hardware.

# 1-Minute How-To
## Download the `ch34x.ko` onto the Raspberry Pi. See the **Releases** tab above. E.g.
```
wget https://github.com/aperepel/raspberrypi-ch340-driver/releases/download/4.4.11-v7/ch34x.ko
```
## Install the kernel module:
```
sudo insmod ch34x.ko
```

## Verify it's showing up in the kernel. You should see the **ch341** listed:
```
$ lsmod
Module                  Size  Used by
bnep                   10340  2
hci_uart               17943  1
btbcm                   5929  1 hci_uart
bluetooth             326105  22 bnep,btbcm,hci_uart
brcmfmac              186599  0
brcmutil                5661  1 brcmfmac
cfg80211              427855  1 brcmfmac
rfkill                 16037  4 cfg80211,bluetooth
snd_bcm2835            20511  0
snd_pcm                75698  1 snd_bcm2835
snd_timer              19160  1 snd_pcm
snd                    51844  3 snd_bcm2835,snd_timer,snd_pcm
i2c_bcm2708             4770  0
bcm2835_gpiomem         3040  0
bcm2835_wdt             3225  0
ch341                   4932  0
usbserial              22115  1 ch341
uio_pdrv_genirq         3164  0
uio                     8000  1 uio_pdrv_genirq
i2c_dev                 5859  0
ipv6                  347530  30
```

## Verify the module has been installed (so it's picked up on reboot). You should see the **ch34x.ko** file listed:
```
ls /lib/modules/$(uname -r)/kernel/drivers/usb/serial
```
If not, try running
```
sudo depmod -a
```

## Plug In ESP8266
Plug in your dev board into any of the Raspberry Pi's USB port. A few things should happen. Again, look for the **340** strings:
```
$ lsmod
[   65.597856] usbcore: registered new interface driver ch34x
[   65.598537] usbserial: USB Serial support registered for ch34x

$ lsusb
Bus 001 Device 004: ID 1a86:7523 QinHeng Electronics HL-340 USB-Serial adapter
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

At this point you're all set. Upload the new firmware to esp8266 and, given it prints output to Serial at 115200 baud rate, this should print stuff:
```
cat /dev/ttyUSB0
```

# Compilation Instructions
Normally isn't required, but if for some reason you need to compile it for a different kernel version of Raspberry Pi, here it goes.

## Download the CH34x Driver Sources
On the Raspberry Pi, go to http://www.wch.cn/download/CH341SER_LINUX_ZIP.html and click that big blue **Download** button. This is the manufacturer's website, but it's all in Chinese.

## Prepare the Kernel Build Environment
```
sudo apt-get -y install linux-headers-rpi
sudo apt-get install rpi-update
sudo rpi-update

# reboot here, you're on the latest kernel

sudo wget https://raw.githubusercontent.com/notro/rpi-source/master/rpi-source -O /usr/bin/rpi-source && sudo chmod +x /usr/bin/rpi-source && /usr/bin/rpi-source -q --tag-update

sudo apt-get install bc
sudo apt-get install libncurses5-dev
rpi-source
```

## Build the Driver Module
```
# unzip the driver sources archive
# cd into the new directory
make
```
This will produce the `ch34x.ko` file, which you can install as described above.
