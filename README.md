# Make SD Card Image Tool

主要是实际操作一下怎么自动生成一个SD卡img文件，仅仅针对i.MX6 Linux Kernel 3.0.35版本。

## 参考文档

* [制作SD卡img文件，并扩容](http://www.cnblogs.com/zengjfgit/p/6443658.html)

## 烧录工具

[MfgToolA51 —— 1.6 Linux3.0.35-img-eMMC-MX6DL-ALL.vbs：](https://github.com/ZengjfOS/MfgToolA51#16-linux3035-img-emmc-mx6dl-allvbs)

## SD自动扩容脚本

[S72resizefs](S72resizefs)

## mksdimg使用说明

* 使用方法：
```
USAGE:
    mksdimg <img | mount | umount | clean>
        1. img: make a img file.
        2. mount: mount img file system.
        3. umount: umount img file system.
        4. clean: clean img file.
```

* 依赖文件：
  * `u-boot.bin`
  * `uImage`
  * [rootfs.tar.bz2](https://github.com/ZengjfOS/ARMBaseFS/tree/i.mx6_base_fs): `cd <your filesystem path> && fakeroot -- tar jcvf rootfs.tar.bz2 *`

## 生成sd.img文件输出信息

```
zengjf@zengjf:~/zengjf/zengjfos/buildroot-2017.02.3/output/mksdimg$ ./mksdimg img
+ BOOT_ROM_SIZE=10
+ echo 10 * 2 * 1024
+ bc
+ BOOT_ROM_SIZE_Sector=20480
+ echo 10 * 2 * 1024 Sector(Sector = 512B) = 20480 Sector(Sector = 512B) = 10 MB
10 * 2 * 1024 Sector(Sector = 512B) = 20480 Sector(Sector = 512B) = 10 MB
+ sudo dd if=/dev/zero of=sd.img bs=1M count=120
120+0 records in
120+0 records out
125829120 bytes (126 MB, 120 MiB) copied, 0.0728065 s, 1.7 GB/s
+ sudo losetup -f --show sd.img
+ node=/dev/loop0
+ echo sd.img device node: /dev/loop0
sd.img device node: /dev/loop0
+ sudo sfdisk --force -uS /dev/loop0
Checking that no-one is using this disk right now ... OK

Disk /dev/loop0: 120 MiB, 125829120 bytes, 245760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Created a new DOS disklabel with disk identifier 0x5fad12a9.
Created a new partition 1 of type 'W95 FAT32 (LBA)' and of size 110 MiB.
/dev/loop0p2: 
New situation:

Device       Boot Start    End Sectors  Size Id Type
/dev/loop0p1      20480 245759  225280  110M  c W95 FAT32 (LBA)

The partition table has been altered.
Calling ioctl() to re-read partition table.
Re-reading the partition table failed.: Invalid argument
The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or kpartx(8).
Syncing disks.
+ sudo kpartx -av /dev/loop0
add map loop0p1 (253:0): 0 225280 linear 7:0 20480
+ sudo dd if=u-boot.bin of=/dev/loop0 bs=512 seek=2 skip=2
406+1 records in
406+1 records out
207924 bytes (208 kB, 203 KiB) copied, 0.0016622 s, 125 MB/s
+ sudo dd if=uImage of=/dev/loop0 bs=1M seek=1 conv=fsync
3+1 records in
3+1 records out
3925476 bytes (3.9 MB, 3.7 MiB) copied, 0.0836364 s, 46.9 MB/s
+ sudo mkfs.ext3 /dev/mapper/loop0p1
mke2fs 1.43.3 (04-Sep-2016)
Discarding device blocks: done                            
Creating filesystem with 112640 1k blocks and 28224 inodes
Filesystem UUID: e46141c9-d05d-4ab0-824f-5896e0c844df
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 

+ sudo mount -t ext3 /dev/mapper/loop0p1 /mnt
+ sudo tar jxvf rootfs.tar.bz2 -C /mnt
THIS_IS_NOT_YOUR_ROOT_FILESYSTEM
bin/
bin/mountpoint
bin/mknod
bin/nice
bin/egrep
bin/pwd
bin/ping
usr/lib32
[省略...]
var/
var/empty/
var/cache
var/run
var/lock
var/www/
var/lib/
var/lib/misc
var/lib/arpd/
var/lib/alsa/
var/spool
var/log
var/tmp
+ sudo umount /mnt
+ sudo kpartx -d /dev/loop0
+ sudo losetup -d /dev/loop0
+ exit 0
zengjf@zengjf:~/zengjf/zengjfos/buildroot-2017.02.3/output/mksdimg$
```
