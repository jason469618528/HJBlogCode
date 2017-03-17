
---
title: Xcode 创建自定义模板 (来源网络)
date: 2017-03-17 12:00:00
tags:
---

### 1 模板方案路径
系统模板的路径（Xcode7）_Applications_Xcode.app_Contents_Developer_Library_Xcode_Templates_File Templates在这个目录下；
![](/img/1693553-0bed79c964adb6dc.png)
其中的Core Data、Other、Resource、Source和User Interface对应着
![](/img/1693553-0bed79c964adb6dc.png)
### BaseVC.xctemplate介绍

假设已经将BaseVC.xctemplate放入了Source文件夹中，查看BaseVC.xctemplate中有
（1）BaseViewControllerObjective-C 文件夹
用来创建.h 和 .m文件。 其中文件夹的命名规范是[name]+Objective-C. 如果是创建swift修改为swift。
（2）BaseViewControllerXibObjective-C文件夹
用来创建.h，.m和.xib文件。其中文件夹的命名规范是[name]+XibObjective-C. swift类似。
（3）TemplateIcon图片
图片是用来显示在New File的菜单上的。任意放一个自己喜欢的图片，像素138*138即可。
（4）TemplateInfo.plist
配置文件。下面单独讲讲。

###  3.___FILEBASENAME___.h
内容为


  ___FILENAME___
  ___PROJECTNAME___

  Created by ___FULLUSERNAME___ on ___DATE___.
___COPYRIGHT___


#import "___VARIABLE_cocoaTouchSubclass___.h"

@interface ___FILEBASENAMEASIDENTIFIER___ : ___VARIABLE_cocoaTouchSubclass___

@end
里面的参数在生成h文件时，系统会替换了输入的文件名。具体都是什么含义，大家可以自行Google了。

### 4. ___FILEBASENAME___.m

内容为


  ___FILENAME___
  ___PROJECTNAME___

  Created by ___FULLUSERNAME___ on ___DATE___.
___COPYRIGHT___

#import "___FILEBASENAME___.h"
 Controllers
 Model
 Views

#define <#macro#> <#value#>


@interface ___FILEBASENAMEASIDENTIFIER___ ()

@property (nonatomic, strong) <#type#> *<#name#>
@end
@implementation ___FILEBASENAMEASIDENTIFIER___

#pragma mark - View Controller LifeCyle

- [ ] (instancetype)initWithCoder:(NSCoder *)coder
{
    self = [super initWithCoder:coder];
    if (self) {
    }
    return self;
}   
- [ ] (void)viewDidLoad
{
    [super viewDidLoad];

    [self initialNavigationBar];
}
- [ ] (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
}
- [ ] (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
}

- [ ] (void)didReceiveMemoryWarning
{
    [super didReceiveMemoryWarning];

    [[SDImageCache sharedImageCache] setValue:nil forKey:@"memCache"];
}

- [ ] (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}
#pragma mark - Override

#pragma mark - Initial Methods

- [ ] (void)initialNavigationBar
{
    self.navigationItem.title = <#title#>;
}

#pragma mark - Target Methods

#pragma mark - Notification Methods

#pragma mark - KVO Methods

#pragma mark - UITableViewDelegate, UITableViewDataSource

#pragma mark - Privater Methods

#pragma mark - Setter Getter Methods

@end
按编程规范编写的，大家可以自行修改，New File后将自动生成你修改的内容。
### 5. TemplateInfo.plist
![](/img/1693553-0ed0fdac87c08112.png)
SortOrder 这个是排序的值，可以设置在界面中的摆放位置
Options中的Item0，Item1，Item2，Item3 对应了
![](/img/1693553-d8e6c07058f379f0-1.png)
自定义的BaseViewController的类，当然可以使用系统或自己定义。
BaseViewController的后缀。选择后，自动在Class后面添加ViewController的后缀。
是否生成Xib文件，默认是ture。当选择了BaseViewController时，可以进行checkout，不然不可选，不生成Xib文件。
