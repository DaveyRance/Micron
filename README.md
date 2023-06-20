# Micron
## Hardware
BTT Manta M8P
TMC2240 (Allows for sensorless homing)
CB1 1G 32GB Emmc
BTT EBB36 

To get EB1 working.
Push the DIP switch (USB OTG) and (RPI BOOT) to ON to enter BOOT mode.
(https://github.com/bigtreetech/CB1#cb1-emmc-version)
Then before restarting make sure to set the Overlay Setting for fdtfile else it will not boot.
Refer to Overlays Settings to set fdtfile to sun50i-h616-biqu-emmc

Turn off Reset the DIP switch (USB OTG) and (RPI BOOT) to OFF and turn on. At this stage you should get a booting PI device if not then first confirm the overlay settings (Will need to re-do the dip switches and sunxi-fel command to access the boot drive).

Klipper should allready be on the device from image.
Install KIAUH (https://github.com/th33xitus/kiauh)
Remove mainsail, install fluidd, Add crowsnest.
Update OS
sudo apt-get update
sudo apt-get upgrade

Check if Can bus is up and working on Manta
ifconfig can0
CAN settings that will be used.
bitrate 500000

If no can then check that Overlay settings are set to enable Can.

Download CanBoot to enable flashing of klipper over Can for EBB in the future.
cd ~
git clone https://github.com/Arksine/CanBoot

Install CanBoot to Manta
cd CanBoot
make clean
make menuconfig

Settings to be used.
**Micro-Controller (STM32)**
**Processor (STM32G0B1)**
Build Canboot deployment (Do not build)
Clock (8MHz)
**Communication (Can bus (On PD12/PD13)**
Application start (8KiB offset)
500000 CAN bus speed

Quit and save
make

put Manta in DFU mode (Hold down boot and press reset) 
check it is in DFU by doing lsusb and should show device in DFU mode.
flash CanBoot on to it by running the following.
sudo dfu-util -a 0 -D ~/CanBoot/out/canboot.bin --dfuse-address 0x08000000:force:leave -d 0483:df11

klipper now must be installed on the Manta
cd ~/klipper
make clean
make menuconfig

**Klipper** Settings to be used.
Micro-Controller (STM32)
Processor (STM32G0B1)
**Bootloader offset (8 KiB bootloader)**
Clock (8MHz)
**Communication (USB to CAN bus bridge (USB on PA11/PA12))**
**CAN bus interface (Can bus (On PD12/PD13))**
500000 CAN bus speed

quit and save
make

Set the manta back in DFU mode then flash
sudo dfu-util -a 0 -d 0483:df11 --dfuse-address 0x08002000 -D ~/klipper/out/klipper.bin

LSUSB should now show 
Bus 001 Device 003: ID 1d50:606f OpenMoko, Inc. Geschwister Schneider CAN adapter

Find and record the CAN bus UUID
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0

A single canbus UUID should be found. (If more than one check that the ebb36/42 is not plugged in if so then unplug and check again. Record this uuid for printer.cfg later.
Install the 120R jumper on the Manta.

Connect the EBB via USB (If CAN cable is not connected then jumper USB **Do not have both USB jumper and CAN power connected same time** you will let magic smoke out.
enabled DFU (hold boot and press reset) check with lsusb 

Configure CanBoot for the EBB 
cd ~
cd CanBoot
make clean
make menuconfig

Settings to be used.
**Micro-Controller (STM32)**
**Processor (STM32G0B1)**
Build Canboot deployment (Do not build)
Clock (8MHz)
**Communication (Can bus (On PB0/PB1)**
Application start (8KiB offset)
500000 CAN bus speed

quit save
make

Flash to EBB
sudo dfu-util -a 0 -D ~/CanBoot/out/canboot.bin --dfuse-address 0x08000000:force:mass-erase:leave -d 0483:df11

if connected by USB then disconnect and remove the jumper.
Check if there are 2 devices can be seen via CAN. If not then check the wiring port used on Manta as there are 2 interfaces but only one is terminated by 120 resistor.

~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0

biqu@BTT-CB1:~/klipper$ ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
Found canbus_uuid=eb602e8ed4c3, Application: Klipper
Found canbus_uuid=34c8f1e1cab5, Application: CanBoot
Total 2 uuids found

In this case the Klipper is the Manta and the CanBoot is the EBB. If 2 Klipper devices are found then double cick on the reset button within .5 of a second to enter CanBoot mode. Record the UUID for the CanBoot device as this will be needed for printer.cfg

klipper now must be installed on the EBB
cd ~/klipper
make clean
make menuconfig

**Klipper** Settings to be used.
Micro-Controller (STM32)
Processor (STM32G0B1)
**Bootloader offset (8 KiB bootloader)**
Clock (8MHz)
**CAN bus interface (Can bus (On PB0/PB1))** (These should match what was configured in CanBoot)
500000 CAN bus speed

Quit, Save
make

flash to EBB update the following command with the UUID for the EBB
python3 ~/CanBoot/scripts/flash_can.py -i can0 -u **YOUR_UUID** -f ~/klipper/out/klipper.bin

If you accidentily flash the Manta then re-compile the settings for the Manta, put in to DFU, re-flash and this time use the correct UUID

**ToDo**
Filter
https://github.com/zruncho3d/zerofilter
2 * https://dfh.fm/collections/micron-v2/products/zero-filter-kit-by-zruncho
Carbon https://dfh.fm/products/nevermore-xl-printer-carbon?variant=43229826842846

Replacement feet that are easier to mount.
https://github.com/PrintersForAnts/Micron/blob/main/Mods/L.e.o.p.a.r.d/EasierZDrives/Readme.md

Micron Pins Mod
https://dfh.fm/collections/micron-v2/products/micron-pins-mod-kit

Possibly DoomCube 
https://dfh.fm/collections/micron-v2
 Need Doom Acrlyic Panels https://dfh.fm/collections/micron-v2/products/micron-doom-acrylic-panels
 Doom ACM bottom Pannels: https://dfh.fm/collections/micron-v2/products/acm-panels-for-micron-doom
 Doom Extrusions: https://dfh.fm/collections/micron-v2/products/micron-frame-doom-upgrade?variant=43521937637598

Power Relay module to turn off after printing
https://biqu.equipment/products/bigtreetech-reply-v1-2-automatic-shutdown-module-after-printing

Change Hotend Fan for PWM and read from EBB

M3 slide in nut support. (Nuts used for where needed to add in nuts even though didnt mention. 
https://github.com/MotorDynamicsLab/LDOVoron0/blob/main/STLs/Slide_In_Nut_Support.stl

Printer Tuning
https://ellis3dp.com/Print-Tuning-Guide/

Documents used for this guide.
From discord (Aron) https://discord.com/channels/825469421346226226/1061793771169263726/1116736114846339102 
https://docs.google.com/document/d/1_PcCsZmZDNQKQ4PZVdHbhnT0SBKkc30miwcUaphM4kQ/edit

Canboot Github
https://github.com/Arksine/CanBoot

KIAUH
https://github.com/th33xitus/kiauh

BTT CB1 eMMC Github
https://github.com/bigtreetech/CB1#cb1-emmc-version

BTT EBB github
https://github.com/bigtreetech/EBB

Manta setup. (Ignore the flashing via SD card just use the DFU used above)
https://3dpandme.com/2022/10/02/tutorial-btt-manta-m8p-cb1-klipper-guide/
