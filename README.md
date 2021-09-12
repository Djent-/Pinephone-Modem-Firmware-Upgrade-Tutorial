# Pinephone-Modem-Firmware-Upgrade-Tutorial
Unified instructions to update Pinephone modem firmware, no SIM required

This is a compilation of four sets of instructions + Googling

This process was followed successfully by me on Manjaro Arm. If any modifications are needed, please submit an issue or a pull request

https://wiki.pine64.org/wiki/PinePhone#Firmware_update

https://github.com/Biktorgj/quectel_eg25_recovery

https://xnux.eu/devices/feature/modem-pp.html#toc-unlock-adb-access

https://github.com/qhuyduong/arm_adb

```
sudo pacman -S git

mkdir ~/build && cd ~/build
git clone https://github.com/qhuyduong/arm_adb
cd arm_adb
wget https://xnux.eu/devices/feature/0001-Fix-build-on-modern-Arch-Linux.patch
git config user.email admin@pinephone
git config user.name admin@pinephone
git am 0001-Fix-build-on-modern-Arch-Linux.patch

sudo pacman -S make automake autoconf m4 gcc libtool
sudo ln -s /usr/bin/aclocal /usr/bin/aclocal-1.15
sudo ln -s /usr/local/automake /usr/bin/automake-1.15
./configure
make

cd ..
mkdir qadbkey-unlock && cd qadbkey-unlock
wget https://xnux.eu/devices/feature/qadbkey-unlock.c
gcc qadbkey-unlock.c -lcrypt -o unlock

sudo screen /dev/ttyUSB2 115200
AT
AT+QADBKEY?
# take note of response
# Ctrl+A k y to kill screen
./unlock [code]
sudo screen /dev/ttyUSB2 115200
AT+QADBKEY=[crypt]
AT+QCFG="usbcfg",0x2C7C,0x125,1,1,1,1,1,1,0
# screen may terminate automatically

cd ../arm_adb/src
sudo ./adb start-server
sudo ./adb shell
id
uname -a
exit
lsusb
sudo ./adb reboot edl
lsusb

killall org_kde_powerdevil

cd ../..
git clone https://github.com/Biktorgj/quectel_eg25_recovery
cd quectel_eg25_recovery
sudo ./qfirehose -f ./

# Ignore these
# errno = 108 (Cannot send after transport endpoint shutdown)
# firehose_protocol.c fh_recv_cmd 309 fail

sudo screen /dev/ttyUSB2 115200
AT
AT+QCFG="usbcfg",0x2C7C,0x125,1,1,1,1,1,1,0
# Ctrl+A k y to kill screen

sudo systemctl restart eg25-manager

reboot

sudo screen /dev/ttyUSB2 115200
AT
AT+QGMR
# This should show your new 002 version
# Ctrl+A k y to kill screen
```
