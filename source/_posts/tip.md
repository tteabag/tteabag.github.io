---
title: Tip
date: 2022-03-10 18:21:58
tags:
---

1. Xcode在Resolve Package Dependencies时候慢的解决方式
   
    原因：是xcode网络请求不会走代理
   
    解决方式：终端挂代理，在项目root path使用```xcodebuild -resolvePackageDependencies -scmProvider system```

2. Xcode添加swift package时慢的处理方法
   
    使用软件Proxifier软件，在打开xcode

3. terminal指定应用打开文件
   `open -a \Application\xx.app xx`

4. Xcode显示Products目录

        打开.pbxproj文件，搜索mainGroup，将mainGroup的值复制给productRefGroup

   5. sizeof 是一个函数

   6.atomic不是线程安全的，具体是在NSMutableArray这种类型上不安全        

<!-- more -->
