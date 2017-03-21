---
title: IOS 开发经验总结(不定时更新)
date: 2017-03-21 14:00:00
tags:
---

1.使用Podfile管理Pods依赖库版本
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
 
### 2.使用系统原生 nabber backItem显示文字问题
eg: A,B两个VC, A navItem.Title = 首页,A push 到 B, B的backItem Title 返回显示 **首页**，这里想更改这个文字 ，在A界面设置navitem 的 BackItem title = 返回，再push ，的backItem Title 显示 **返回**；使用原生的backItem 必须先提前设置backItem title; 
`self.navigationItem.backBarButtonItem = [UIBarButtonItem alloc] initWithTitle:<#(nullable NSString *)#> style:<#(UIBarButtonItemStyle)#> target:<#(nullable id)#> action:<#(nullable SEL)#>`

### 3.  设置 uibarbuttonitem image 时按钮变成蓝色

*着色（Tint Color）是iOS7界面中的一个.设置UIImage的渲染模式：UIImage.renderingMode重大改变，你可以设置一个UIImage在渲染时是否使用当前视图的Tint Color。UIImage新增了一个只读属性：renderingMode，对应的还有一个新增方法：imageWithRenderingMode:，它使用UIImageRenderingMode枚举值来设置图片的renderingMode属性。该枚举中包含下列值：

UIImageRenderingModeAutomatic   根据图片的使用环境和所处的绘图上下文自动调整渲染模式。  
UIImageRenderingModeAlwaysOriginal    始终绘制图片原始状态，不使用Tint Color。  
UIImageRenderingModeAlwaysTemplate    始终根据Tint Color绘制图片，忽略图片的颜色信息。  
 
renderingMode属性的默认值是UIImageRenderingModeAutomatic，即UIImage是否使用Tint Color取决于它显示的位置。其他情况可以看下面的图例*
设置selectedImage 的 renderingMode 为UIImageRenderingModeAlwaysOriginal
UIImage *selectedImage=[UIImage imageNamed: @"btn_nav_share_lawyer"];
        selectedImage = [selectedImage imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
  创建baritem
 shareBarBtn = [[UIBarButtonItem alloc] initWithImage:selectedImage style:UIBarButtonItemStyleDone target:self action:@selector(shareAction)];
