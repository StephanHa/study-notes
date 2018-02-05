Create Bootable USB Drive

参考[Making a Kali Bootable USB Drive](http://docs.kali.org/downloading/kali-linux-live-usb-install)



Mac和Linux类似写入命令，只是查看挂载路径不同。

（Linux# fdisk -l ; Mac# diskutil list）

# 1 查看U盘设备挂载路径

记录U盘在系统中挂载的路径，比如/dev/disk1，==必须慎重确定这个路径==，否则写入系统硬盘就杯具了。

```shell
[ StephanX@kali-linux-2.0-i386 ]$ diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *121.3 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                  Apple_HFS Macintosh HD            120.5 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *8.0 GB     disk1
   1:                 DOS_FAT_32 ParrotSec               8.0 GB     disk1s4
```



# 2 卸载U盘的挂载

```shell
[ StephanX@kali-linux-2.0-i386 ]$ diskutil unmountDisk /dev/disk1
Unmount of all volumes on disk1 was successful
```



# 3 将ISO写入U盘

```shell
[ StephanX@kali-linux-2.0-i386 ]$ sudo dd if=kali-linux-2.0-i386.iso of=/dev/disk1 bs=3m
1081+1 records in
1081+1 records out
3403579392 bytes transferred in 1926.533436 secs (1766686 bytes/sec)
[ Aaron@kali-linux-2.0-i386 ]$ 
```



# 4 退出U盘

```shell
[ StephanX@LinuxIOS ]$ diskutil eject /dev/disk1
Disk /dev/disk1 ejected
```



# 5 加速写入

将`of=/dev/disk1` 改成` of=/dev/rdisk1`，就是在“disk1”前加“r”可以比较快速写入，实践过程确实证明，正常4G写入要1926秒，而加入“r”后423秒。

```shell
[ StephanX@LinuxIOS ]$ sudo dd if=ubuntu-mate-15.10-desktop-armhf-raspberry-pi-2.img of=/dev/rdisk1 bs=3m
1250+0 records in
1250+0 records out
3932160000 bytes transferred in 423.962735 secs (9274777 bytes/sec)
```

