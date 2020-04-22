# upgradeSSDforMac

MacBook Pro (Retina, 15-inch, Mid 2014) with 256G SSD   
Catalina 10.15.4 (19E287)  

Upgrade to 500G SSD  

购买Samsung 970 EVO Plus 500G  
购买转接头、导热硅胶片、螺丝刀(五角星、六角星各一)  
淘宝(迪奥科技)可以把除SSD外的上述工具买齐  

Time Machine备份  
进入Disk Utility把移动硬盘格式化为Mac OS Extended (Journaled)  
https://support.apple.com/zh-cn/HT201250  

以防万一，可以先用U盘做一个系统安装盘  
https://support.apple.com/zh-cn/HT201372  

建议先插上转接头，再插上新的SSD  
换好SSD后，按电源键开机，同时按住command和R两个键直到出现苹果标志  
(这一步有其他说法，说要电源、command、option、R四个键同时按住直到小地球出现)  
Disk Utility -> View All  
把新的SSD格式化为  
Format:APFS, Scheme:GUID Partition Map  

插上移动硬盘  
从Time Machine恢复备份  

如果发热太厉害，可以考虑重置SMC   
https://support.apple.com/zh-cn/HT201295  

另外也有建议重置NVRAM的  
https://support.apple.com/zh-cn/HT204063  

检查SSD是否接触良好  
左上角苹果图标 -> About This Mac -> System Report -> NVMExpress  
如果显示Link Width:	x4 说明没问题(应该是)  
