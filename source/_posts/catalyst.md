---
title: Catalyst
date: 2023-04-11 14:05:59
tags:
---

# Catalyst

### 什么是`Catalyst`?

It is Apple’s mechanism for bringing iPad apps to the Mac.

Introduced in `macOS 10.15`.

It’s not AppKit but UIKit running under the x86 architecture, rendering Mac-style UI.

<!-- more -->

### 如何将实施Catalyst

`Catalyst`基于iPad App进行mac适配，所以对于iPhone App需要先进行iPad适配在进行mac适配。

可以直接添加iPad和mac Destination，而不进行适配，此时在其他平台的表现和iPhone一致。但需要修改iPad 的 present crash。

##### iPad 适配

- 适配SplitViewController

- Drag & Drop

- Setting the Scene & Muti-Window

- Adding Context

- The Keyboard & Shortcuts

- Preferences & Settings Bundle

##### mac 适配

- The Mouse
  
  - Pointer Style Providers
  
  - Adding Effects With Hover Gesture Recognizer & NSCursor

- Menu Bar

- Toolbar

- Touch Bar
