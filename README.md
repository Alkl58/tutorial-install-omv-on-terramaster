# A tutorial how to install OpenMediaVault on a Terramaster F5-420


Greetings,

because most Terramaster NAS Models aren't shipped with a Display connector, even tho the PCB has a 12-pin VGA Connector, we have to Install OMV without "Display". You can order those adapters on eBay for ~5€, but it would take a minimum of 2 weeks to arrive...

Because I didn't want to wait, I followed a german tutorial. Because it didn't worked as well as in the tutorial, I will give you my approach.
Preparement

❗❗❗❗❗Backup all your Data, because the drives need to be formatted ❗❗❗❗❗

Get a USB 2.0 USB Drive, if you want to put it inside the NAS. (minimum 8GB) | You can use a USB 3.0 Drive, if you use the USB 3.0 Port outside on the rear of the NAS!

* Download OMV 4 (OMV 5 will only recognize 3 out of 5 hard drives!)

* Download VirtualBox (VMWare will also work)

* Download StarWind V2V Image Converter

* Download Win32 Disk Imager

* Download Putty

## First Installation

Before we create the VirtualMachine, we need to know which size the USB-Drive has. Plug it in your computer and look in the Windows Explorer which size it has. Example: My 8GB USB-Drive has only ~7.6GB. So I know that my VM must have less space than that. In that example I would create a VM with ~7.55GB

1. Create a new VirtualMachine. https://imgur.com/a/aljJHOK

2. Mount the OMV 4 ISO in the VM and start it up

3. Select "Install" from the Boot Menu

4. During installation type it your root password etc.

5. After Installation has been completed, it will ask you to continue to reboot or to go back. Press the Button to go back.

6. You should now see a Menu with different options. Press "ALT + F2". This should prompt a Black Screen where you need to press "Enter" to open the console.

Now the important part which is dependant on which Virtualization Software you use. Here is a tutorial which should work for all (VirtualBox, VmWare etc):

7. Type the following in the console: nano /target/etc/network/interfaces and press Enter. It should show you the current network device name e.g. "ens33" or "enp0s3". Press "Ctrl + X" to exit nano. You need to change the current device name to enp2s0:

8. Now type the following command: ```sed -i 's/enp0s3/enp2s0/' /target/etc/network/interfaces``` (replace "enp0s3 " with the current device name)

9. Now type the following command: ```sed -i 's/enp0s3/enp2s0/' /target/etc/collectd/collectd.conf.d/interface.conf``` (replace "enp0s3 " with the current device name)

10. Now type the following command: ```sed -i 's/enp0s3/enp2s0/' /target/etc/openmediavault/config.xml``` (replace "enp0s3 " with the current device name)

11. **Now switch off the VM hard, power off!**

Here is a picture showing it for an VMWare installation: 

![](https://preview.redd.it/wtuk2y07ujx41.png?width=800&format=png&auto=webp&s=617b048ee5e5ababbb3c35a8b80866f585ad201a)

*(Source: https://www.bachmann-lan.de/terramaster-f2-220-nas-mit-openmediavault/)*

## Flashing the USB Drive

1. Open StarWind V2V Image Converter -> select the VMDK file -> Select "RAW" as image format -> Create the image. Wait until its finished.

2. Open Win32 Disk Imager -> Select the created img file from Step 1 -> Select your Device (all Data on the USB Drive will be overwritten!) -> Click "Write". When finished you can remove the drive.


## NAS Installation

1. Remove all Hard Drives

2. Open your NAS, by unscrewing all screws on the backside.

3. Disconnect the Fans from the PCB

4. Slide out the inner part of the NAS to the front.

5. You should see the USB Drive, where TOS is installed. Remove it by removing the glue.

6. Insert your flashed USB 2.0 drive in the same port. If you use a USB 3.0 Drive you can put it in the USB 3.0 Port on the Backside of the NAS.

7. Don't assemble the NAS yet. Connect your Lan-Cable to "Lan-Port 1" and then connect your NAS to power

8. When Powering it on you should see following pattern: The Lan LEDs from the Lanport are going on (current state is the BIOS) -> Lan LEDs goes off (current state is the OS starting) -> Lan LEDs are going on again. If they are not going on after max ~5min, then you might have made an error with the Step replacing the Device Name.

Congrats. You have successfully booted inside of OMV 4. It should now have an IP, which should be reachable from your Browser.

Now to a problem I encountered: The NAS wouldn't boot when the hard drives where connected. To fix this issue you have to do following steps:
1. Disconnect all Hard drives -> System should now boot without problems

2. Access your NAS via SSH as root. (Username: "root" |Password: the password you specified during installation)

3. Execute the following command: `update-grub` and then `shutdown -h now`

4. Insert all hard drives

Here a bad translation why this problem occurs:

```The drive identifiers move when the data disks are inserted. The SSD (USB Drive) previously recognized as / dev/sda now slides onto /dev/sdb or /dev/sdc.Since grub2 entered /dev/sda1 in /boot/grub/grub.cfg as the system partition during installation in the VM, the NAS hangs in the initrd when booting with data disks.```

Here is the source (german!): https://www.heise.de/forum/c-t/Kommentare-zu-c-t-Artikeln/x86-NAS-TerraMaster-F2-220-als-Mikroserver/Re-Mit-eingebauten-Festplatten-bootet-OMV-nicht/posting-31564349/show/
