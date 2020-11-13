# ili9341_tft_ubuntu20_04

Please note this method is tested only on 2.8" TFT with ili9341 driver on Raspberry Pi 3b (Ubuntu20.04 with Kernel 5.4). It uses Device Tree overlay instead of fbtft_device.
Maybe this can not work with Raspberry OS or other Ubuntu distros or other Kernel versions. Please don't ask about it.
Startx was not tested. Only console output was tested.

## 1) Wiring between 2.8" TFT and RPi:
https://github.com/Broodie/ili9341_tft_ubuntu_20_04/blob/main/rpi-display-overlay.dts

## 2) Kernel update
```
sudo curl -L --output /usr/bin/rpi-update https://raw.githubusercontent.com/Hexxeh/rpi-update/master/rpi-update
sudo chmod +x /usr/bin/rpi-update
sudo REPO_URI=https://github.com/notro/rpi-firmware rpi-update
sudo reboot
```

## 3) Install utility to show or change the settings of the frame buffer device
```
sudo apt-get install -y fbset
```

## 4) Get necessary overlay file
```
wget -N https://github.com/Broodie/ili9341_tft_ubuntu_20_04/raw/main/rpi-display.dtbo
sudo cp rpi-display.dtbo /boot/overlays/rpi-display.dtbo
```

#### 4*) LED, DC, RESET pins can be connected in different GPIOs than described in .dts file. In that case you need to modify the .dts file accordingly and compile it into correct destinaton
```
wget -N https://github.com/Broodie/ili9341_tft_ubuntu_20_04/raw/main/rpi-display-overlay.dts
```
Modify LED, DC, RESET pins in fragment@3 and fragment@4 section
```
sudo dtc -@ -I dts -O dtb -o /boot/overlays/rpi-display.dtbo rpi-display-overlay.dts
```

## 5) Enable SPI with Device Tree overlay
```
sudo bash -c "echo 'dtoverlay=rpi-display' >> /boot/firmware/usercfg.txt"
```

## 6) Make it permanent at system startup
If you have `/etc/rc.local` then append `con2fbmap 1 2` before `exit 0`
if you don't have `/etc/rc.local` then:
```
wget -N https://github.com/Broodie/ili9341_tft_ubuntu_20_04/raw/main/rc.local
sudo cp rc.local /etc/rc.local
```

## 7) Console font face and size
```
sudo nano /etc/default/console-setup
```
FONTFACE="Terminus"<br>
FONTSIZE="6x12"

## 8) Reboot
```
sudo reboot
```

## Troubleshooting:
If the console does not appear on the display then maybe your display is not defined the way like me. For me it was `fb2`.
### 1) Check whether the device is recognized by the system
```
cat /sys/bus/spi/devices/spi0.0/modalias
```
You should get:
```
spi:ili9341
```
### 2) Check how the device is loaded
```
dmesg | grep ili
```
You should get:
```
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 227072
[    2.511154] evm: security.capability
[   11.173859] systemd[1]: Listening on initctl Compatibility Named Pipe.
[   13.963942] fb_ili9341: module is from the staging directory, the quality is unknown, you have been warned.
[   14.270895] graphics fb2: fb_ili9341 frame buffer, 320x240, 150 KiB video memory, 16 KiB buffer memory, fps=31, spi0.0 at 32 MHz
```
Here you can see I've got `fb2` so my command in `/etc/rc.local` is `con2fbmap 1 2`. But if your display is `fb1` then you need to change the command to `con2fbmap 1 1`.
