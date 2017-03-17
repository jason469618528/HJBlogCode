---
title: IOS 开发经验总结(不定时更新)
date: 2017-03-16 10:00:00
tags:
---

## 使用Podfile管理Pods依赖库版本
```
pod 'AFNetworking'      //不显式指定依赖库版本，表示每次都获取最新版本    
pod 'AFNetworking', '2.0'     //只使用2.0版本    
pod 'AFNetworking', '> 2.0'     //使用高于2.0的版本    
pod 'AFNetworking', '>= 2.0'     //使用大于或等于2.0的版本    
pod 'AFNetworking', '< 2.0'     //使用小于2.0的版本    
pod 'AFNetworking', '<= 2.0'     //使用小于或等于2.0的版本    
pod 'AFNetworking', '~> 0.1.2'     //使用大于等于0.1.2但小于0.2的版本    
pod 'AFNetworking', '~>0.1'     //使用大于等于0.1但小于1.0的版本    
pod 'AFNetworking', '~>0'     //高于0的版本，写这个限制和什么都不写是一个效果，都表示使用最新版本  
```
