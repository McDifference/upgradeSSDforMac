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

网上找到很多人都有这种情况，共同点就是更换过硬盘并系统升级到了Catalina (https://www.reddit.com/r/MacOS/comments/dme38s/sleep_wake_failure_in_efi/)  
原因在于Catalina下休眠后从硬盘恢复会出错。具体而言，
```
在hibernatemode 3情形下，其休眠受standbydelayhigh和standbydelaylow的两个数值之间的某个时间点，机器自动将内存里的数据写入到硬盘。  
而第三方的硬盘往往无法在此过程中被识别出来，导致了长时间休眠后睡死唤不醒。
```   
(https://blog.csdn.net/hitpisces/article/details/90907498)  
我个人认为不只是因为第三方硬盘不支持standby模式，而是不支持从硬盘唤醒，因为我的standbydelayhigh为86400即24小时，standbydelaylow为10800即为3小时，我的情况一般是电池往往在100%电量时拔掉电源合上盖子，过一晚上电量只下降不到5%，然而却无法唤醒。而如果只是拔掉电源合上盖子几个小时，是可以马上唤醒的。  
因此我的情况应该是出现在autopoweroff。我的autopoweroffdelay是28800即8小时，也就是睡眠了一个晚上超过8小时后进行了autopoweroff，然后无法从硬盘唤醒。  

可以在terminal中查看Power Management的standby和hibernate模式  
```pmset -g custom```  
若standby为1且在电池模式下  
1. 若电量大于highstandbythreshold，则在睡眠standbydelayhigh秒后把内存的内容(hibernation image)写入硬盘  
2. 若电量小于highstandbythreshold，则在睡眠standbydelaylow秒后把内存的内容(hibernation image)写入硬盘  
(https://www.qiansw.com/in-depth-study-of-mac-os-sleep-mode.html)  

关于macOS的睡眠(sleep)、休眠(hibernate)以及standby模式的资料如下： 
```
hibernatemode supports values of 0, 3, or 25.   
Whether or not a hibernation image gets written is also dependent on the values of standby and autopoweroff  

For example, on desktops that support standby a hibernation image will be written after the specified standbydelay time.  
To disable hibernation images completely, ensure hibernatemode standby and autopoweroff are all set to 0.  

hibernatemode = 0 by default on desktops.  
The system will not back memory up to persistent storage. The system must wake from the contents of memory; the system will lose context on power loss. This is, historically, plain old sleep.  

hibernatemode = 3 by default on portables.  
The system will store a copy of memory to persistent storage (the disk), and will power memory during sleep.  
The system will wake from memory, unless a power loss forces it to restore from hibernate image.  

hibernatemode = 25 is only settable via pmset.  
The system will store a copy of memory to persistent storage (the disk), and will remove power to memory.  
The system will restore from disk image.  
If you want "hibernation" - slower sleeps, slower wakes, and better battery life, you should use this setting.  

Please note that hibernatefile may only point to a file located on the root volume.  
```  
```
standby causes kernel power management to automatically hibernate a machine after it has slept for a specified time period.  
This saves power while asleep. This setting defaults to ON for supported hardware.  
The setting standby will be visible in pmset -g if the feature is supported on this machine.  

standbydelayhigh and standbydelaylow specify the delay, in seconds, before writing the hibernation image to disk and powering off memory for Standby.  
standbydelayhigh is used when the remaining battery capacity is above highstandbythreshold , and standbydelaylow is used when the remaining battery capacity is below highstandbythreshold.  

highstandbythreshold has a default value of 50 percent.  
```  
```
autopoweroff is enabled by default on supported platforms as an implementation of Lot 6 to the European Energy-related Products Directive.  
After sleeping for <autopoweroffdelay> seconds, the system will write a hibernation image and go into a lower power chipset sleep.  
Wakeups from this state will take longer than wakeups from regular sleep.  

autopoweroffdelay specifies the delay, in seconds, before entering autopoweroff mode.  
```  

解决方法：  

由于我只在电池模式下才出现问题，我只禁用电池模式下的standby和autopoweroff  
```
sudo pmset -b standby 0
sudo pmset -b autopoweroff 0
```    
电源模式下的standby和autopoweroff仍然保持为开启。若要手动开启电源模式的standby和autopoweroff  
```
sudo pmset -c standby 1
sudo pmset -c autopoweroff 1
```    

后续问题：  
禁用standby和autopoweroff后，拔掉电源合上盖子sleep一段时间后不会进入hibernate。电脑会持续给内存供电，唤醒时从内存唤醒，直到电池电量耗尽关机。这解决了不供电睡眠无法唤醒的问题，也因为不需要向硬盘写入镜像而减小硬盘损耗，但同时也会导致睡眠时电池耗电加快(10小时电量下降7%)，而且电量耗尽后关机前的内容无法恢复。因此若要长时间不供电睡眠，请在睡眠前把重要资料保存好，或者直接关机。  
