# upgradeSSDforMac

MacBook Pro (Retina, 15-inch, Mid 2014) with 256G SSD   
Catalina 10.15.4 (19E287)  

Upgrade to 500G SSD  

购买Samsung 970 EVO Plus 500G  
由于上述SSD是M.2接口的，还需要一个转接头转换成Apple的接口  
购买转接头、导热硅胶片、螺丝刀(五角星、六角星各一)  
淘宝(迪奥科技)可以把除SSD外的上述工具买齐  

先把MacOS更新到最新版，以便支持更多的硬盘协议  

Time Machine备份  
进入Disk Utility把移动硬盘格式化为Mac OS Extended (Journaled)  
https://support.apple.com/zh-cn/HT201250  

以防万一，可以先用U盘做一个系统安装盘  
https://support.apple.com/zh-cn/HT201372  

关机，拔掉电源线  
用五角星螺丝刀打开后盖，注意转轴处的两颗螺丝比其他的更短一些  
拆下来的每一颗螺丝都尽量安装回原来的位置，以免损坏螺纹  
拆螺丝时尽量小心，螺丝太小，螺帽容易磨损导致打滑  
手上先去除静电  
这个时候主板还没有断电的，操作时候手以及导电的东西都尽量不要碰到电路，以免短路  
又或者可以先把主板断电(我不知道怎么做)  
顺便清理一下灰尘  
用六角星螺丝刀卸下固定SSD的螺丝  
拔下SSD  

新的SSD底部贴上导热硅胶，此硅胶也可以起到固定和缓冲的作用  
建议先插上转接头，再插上新的SSD  
螺丝固定好SSD，并装上后盖  
插上电源，按电源键开机，同时按住command和R两个键直到出现苹果标志  
(这一步有其他说法，说要电源、command、option、R四个键同时按住直到小地球出现)  
Disk Utility -> View All  
把新的SSD格式化为  
Format:APFS, Scheme:GUID Partition Map  

插上移动硬盘  
从Time Machine恢复备份  

检查SSD是否接触良好  
左上角苹果图标 -> About This Mac -> System Report -> NVMExpress  
如果显示Link Width:	x4 说明没问题(应该是)

更换SSD后出现的问题： 

1. 每次睡眠后的唤醒速度都比较慢，无论插着电源还是使用电池，而且唤醒时发热很严重，风扇高速转动  
于是我重置了SMC   
https://support.apple.com/zh-cn/HT201295  
也有建议顺便重置NVRAM的  
https://support.apple.com/zh-cn/HT204063  
我重置SMC后，唤醒速度变快了，而且发热低，唤醒时也听不到风扇转动，但出现了另一个问题   

2. 重置SMC后发现在不插电源的情况下合盖休眠一段时间后，开盖无法唤醒，系统自动重启，或者在输入开机密码后死机必须手动重启，重启后会弹出Sleep Wake Failure In EFI的错误提示  
重置SMC和NVRAM也解决不了问题  
有人说可能是Paragon NTFS的原因，我关掉Paragon NTFS并关闭开机时自动开启，关掉它在顶部状态栏的图标，也解决不了问题  

问题出现的原因：  

网上找到很多人都有这种情况，共同点就是更换过硬盘并系统升级到了Catalina  
原因在于Catalina下休眠后从硬盘恢复会出错。具体而言，
```
在hibernatemode 3情形下，其休眠受standbydelayhigh和standbydelaylow的两个数值之间的某个时间点，机器自动将内存里的数据写入到硬盘。而第三方的硬盘往往无法在此过程中被识别出来，导致了长时间休眠后睡死唤不醒。
```   
(https://blog.csdn.net/hitpisces/article/details/90907498)

可以在terminal中查看Power Management的standby和hibernate模式  
```pmset -g```  
若standby为1且在电池模式下  
1. 若电量大于highstandbythreshold，则在睡眠standbydelayhigh秒后把内存的内容(sleep image)写入硬盘  
2. 若电量小于highstandbythreshold，则在睡眠standbydelaylow秒后把内存的内容(sleep image)写入硬盘  
(https://www.qiansw.com/in-depth-study-of-mac-os-sleep-mode.html)  

关于macOS的睡眠(sleep)、休眠(hibernate)以及standby模式的资料如下： 
```
Hibernate mode : supports values of 0, 3, or 25.

Whether or not a hibernation image gets written is also dependent on the values of standby and autopoweroff.

0 by default on desktops. The system will not back memory up to persistent storage (the disk). The system must wake from the contents of memory; the system will lose context on power loss. This is, historically, plain old sleep.

3 by default on portables. The system will store a copy of memory to persistent storage (the disk), and will power memory duringsleep. The system will wake from memory, unless a power loss forces it to restore from hibernate image.

25 is only settable via pmset. The system will store a copy of memory to persistent storage (the disk), and will remove power to memory. The system will restore from disk image. If you want "hibernation" slower sleeps, slower wakes, and better battery life, you should use this setting.

standby causes kernel power management to automatically hibernate a machine after it has slept for a specified time period. This saves power while asleep. This setting defaults to ON for supported hardware. The setting standby will be visible in pmset -g if the feature is supported on this machine.
```  

解决方法：  

禁用standby  
```sudo pmset -a standby 0```  
参考资料(https://www.reddit.com/r/MacOS/comments/dme38s/sleep_wake_failure_in_efi/)  

后续问题：  
禁用standby后，拔掉电源sleep一段时间后不会进入hibernate，但会同时把内存的内容制作成镜像(sleep image)写入硬盘。因此电脑会持续给内存供电，唤醒时从内存唤醒，除非电池没电了才会从硬盘唤醒。这虽然解决了第三方硬盘不支持standby模式的问题，但也会导致睡眠时电池耗电加快。  
