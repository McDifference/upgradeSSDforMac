# upgradeSSDforMac

MacBook Pro (Retina, 15-inch, Mid 2014) with 256G SSD 
Catalina 10.15.4 (19E287) 

Upgrade to 500G SSD 

Buy a Samsung 970 Evo Plus 
Buy a 转接头、导热硅胶片、螺丝刀(五角星、六角星各一) 
淘宝(迪奥科技)可以把除SSD外的上述工具买齐 

Time Machine备份 
Disk Utility把移动硬盘格式化为Mac OS Extended (Journaled) 

以防万一，可以先用U盘做一个系统安装盘 
https://support.apple.com/zh-cn/HT201372 

建议先插上转接头，再插上新的SSD 
换好SSD后，开机 
Disk Utility，View all，把新的SSD格式化为 
Format:APFS, Scheme:GUID Partition Map 

插上移动硬盘 
从Time Machine恢复备份 

如果发热太厉害，可以考虑重置SMC 
https://support.apple.com/zh-cn/HT201295 

检查SSD是否接触良好 
左上角苹果图标 -> About This Mac -> System Report -> NVMExpress 
Link Width:	x4 说明没问题(应该是) 
