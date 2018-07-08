#### 工厂镜像刷机步骤

1. 开发者选项解锁 `OEM`
2. 进入刷机模式 `adb reboot bootloader`
3. 解锁引导加载程序 `fastboot flashing unlock`
4. 进入要刷入的 `factory` 解压目录
5. 执行刷机脚本 `flash-all.sh` ，等待刷机完成
6. 重锁引导加载程序 `fastboot flashing lock`
7. 开发者选项重锁 `OEM`


#### 命令行日志

```shell
➜  angler-opm6.171019.030.e1 fastboot flashing unlock
(bootloader) Please select 'YES' on screen if you want to continue...
(bootloader) Unlocking bootloader...
(bootloader) Unlocked!
OKAY [ 13.067s]
Finished. Total time: 13.067s

➜  angler-opm6.171019.030.e1 flash-all.sh
Sending 'bootloader' (3552 KB)                     OKAY [  0.128s]
Writing 'bootloader'                               OKAY [  0.206s]
Finished. Total time: 0.366s
rebooting into bootloader                          OKAY [  0.126s]
Finished. Total time: 0.126s
Sending 'radio' (48728 KB)                         OKAY [  1.323s]
Writing 'radio'                                    OKAY [  2.162s]
Finished. Total time: 3.511s
rebooting into bootloader                          OKAY [  0.009s]
Finished. Total time: 0.009s
extracting android-info.txt (0 MB) to RAM...
--------------------------------------------       
Bootloader Version...: angler-03.79
Baseband Version.....: angler-03.87
Serial Number........: ************
--------------------------------------------       
Checking product                                   OKAY [  0.020s]
Checking version-bootloader                        OKAY [  0.021s]
Checking version-baseband                          OKAY [  0.020s]
extracting boot.img (11 MB) to disk... took 0.062s
archive does not contain 'boot.sig'
archive does not contain 'dtbo.img'
archive does not contain 'dt.img'
archive does not contain 'odm.img'
archive does not contain 'product.img'
extracting recovery.img (18 MB) to disk... took 0.121s
archive does not contain 'recovery.sig'
extracting system.img (1913 MB) to disk... took 20.984s
archive does not contain 'system.sig'
archive does not contain 'vbmeta.img'
extracting vendor.img (188 MB) to disk... took 1.360s
archive does not contain 'vendor.sig'
mke2fs 1.43.3 (04-Sep-2016)
Creating filesystem with 6694270 4k blocks and 1676080 inodes
Filesystem UUID: ********************************************
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done   

mke2fs 1.43.3 (04-Sep-2016)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

Sending 'boot' (12089 KB)                          OKAY [  0.342s]
Writing 'boot'                                     OKAY [  0.174s]
Sending 'recovery' (18961 KB)                      OKAY [  0.506s]
Writing 'recovery'                                 OKAY [  0.275s]
Sending sparse 'system' 1/5 (482756 KB)            OKAY [130.905s]
Writing sparse 'system' 1/5                        OKAY [  7.279s]
Sending sparse 'system' 2/5 (475238 KB)            OKAY [ 41.136s]
Writing sparse 'system' 2/5                        OKAY [  6.603s]
Sending sparse 'system' 3/5 (475606 KB)            OKAY [299.583s]
Writing sparse 'system' 3/5                        OKAY [  8.172s]
Sending sparse 'system' 4/5 (476548 KB)            OKAY [132.664s]
Writing sparse 'system' 4/5                        OKAY [  6.898s]
Sending sparse 'system' 5/5 (49108 KB)             OKAY [  1.428s]
Writing sparse 'system' 5/5                        OKAY [  0.681s]
Sending 'vendor' (192565 KB)                       OKAY [  5.151s]
Writing 'vendor'                                   OKAY [  3.434s]
Erasing 'userdata'                                 OKAY [  0.731s]
Sending 'userdata' (4316 KB)                       OKAY [  0.150s]
Writing 'userdata'                                 OKAY [  0.095s]
Erasing 'cache'                                    OKAY [  0.018s]
Sending 'cache' (72 KB)                            OKAY [  0.026s]
Writing 'cache'                                    OKAY [  0.030s]
Rebooting                                          
Finished. Total time: 1279.833s

➜  angler-opm6.171019.030.e1 adb reboot bootloader
➜  angler-opm6.171019.030.e1 fastboot flashing lock
(bootloader) Please select 'YES' on screen if you want to continue...
(bootloader) Locking bootloader...
(bootloader) Locked!
OKAY [  6.517s]
Finished. Total time: 6.517s
```
