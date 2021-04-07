# RT-kernel compilation instructtions
Instructions as to how to compile the rt-kernel for the Raspberry pi 4b
#Download Debian 10.7 Virtualbox

#Dependencies 
sudo apt install git bc bison flex libssl-dev make libc6-dev libncurses5-dev
sudo apt install crossbuild-essential-arm64

#Pull kernel and patch
mkdir kernel-out
cd linux/

git clone --depth=1 https://github.com/raspberrypi/linux -b rpi-5.10.y
wget https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/5.10/patch-5.10.25-rt35.patch.gz
gzip -cd ../patch-5.10.25-rt35.patch.gz |  patch -p1 --verbose

#Configure
KERNEL=kernel8
make  O=../kernel-out/  ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
make  O=../kernel-out/  ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig menuconfig
#Menuconfig 
Disable Virtualisation/KVM
General -> Preemtion Model and select Real Time option

#Compile
make -j12  O=../kernel-out/ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs

#Perpare the tar archive
export INSTALL_MOD_PATH=~/rpi-kernel/rt-kernel
export INSTALL_DTBS_PATH=~/rpi-kernel/rt-kernel
make O=../kernel-out/ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- modules_install dtbs_install
cp ../kernel-out/arch/arm64/boot/Image ../rt-kernel/boot/kernel8.img
mkdir ../rt-kernel/boot
cp ../kernel-out/arch/arm64/boot/Image ../rt-kernel/boot/kernel8.img
cd $INSTALL_MOD_PATH
tar czf ../rt-kernel.tgz *

#Install the kernel on the raspi
cd /tmp
tar xzf rt-kernel.tgz
cd boot
sudo cp -rd * /boot/
cd ../lib
sudo cp -dr * /lib/
cd ../overlays
sudo cp -dr * /boot/overlays
cd ../broadcom
sudo cp -dr bcm* /boot/

#Edit /boot/config.txt
kernel=kernel8.img

