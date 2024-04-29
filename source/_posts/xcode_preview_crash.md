---
title: Xcode Preview Crash
date: 2022-05-13 17:25:15
tags:
---

# Xcode Preview Crash

结论，运行
```
xcrun simctl --set previews delete all
```

崩溃原因：使用了realmswift，然后修改了Object，按照常理说删除应用后，在migration的时候就不会崩溃；但preview不得行，一样会崩溃，猜测是preview应用缓存在其他位置，不是模拟器上看到的那个应用。

网上找到的解决方法：
终端运行
```
xcrun simctl --set previews delete all
```
