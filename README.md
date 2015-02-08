# pcDuino3Nano-sdcard

pcDuino3 Nano sdcard build script and auxiliary files

Build sdcard Process
--------------------
cd ./staging

sdcard="/dev/sdb"

dd if=/dev/zero of=${sdcard} bs=1M count=1

dd if=output/u-boot/u-boot-sunxi-with-spl.bin of=${sdcard} bs=1024 seek=8

sfdisk -R ${sdcard}  
cat <<EOT | sfdisk --in-order -L -uM ${sdcard}  
1,16,c  
,,L  
EOT  

mkfs.vfat ${sdcard}1  
mkfs.ext4 ${sdcard}2  

mkdir bootfs  
mkdir rootfs  

mount ${sdcard}1 bootfs  
cp output/boot/boot.scr bootfs  
cp output/boot/linksprite-pcduino3-nano.bin bootfs  
tar zxvf output/boot/linux-3.4.106-pcduino3-nano.tar.gz \\  
&nbsp;&nbsp;&nbsp;&nbsp; --wildcards --strip-components=1 -C bootfs/ boot/System.map* boot/config* boot/vmlinuz*
umount ${sdcard}1

mount ${sdcard}2 rootfs

tar xzvf output/rootfs/wheezy-rootfs.tar.gz -C rootfs  
tar zxvf output/boot/linux-3.4.106-pcduino3-nano.tar.gz -C output/rootfs/ lib  

umount ${sdcard}2

sync
