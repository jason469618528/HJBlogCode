---
title:  Mac 系统重新设置新的管理员账户
date: 2017-8-4 12:00:00
tags:
---

开机，启动时按“cmd+S”。这时，你会进入Single User Model，出现像DOS一样的提示符 #root\>。请在#root\>下输入 (注意空格, 大小写)
fsck -y
mount -uaw /
rm /var/db/.AppleSetupDone
reboot
电脑重新启动进入激活界面
