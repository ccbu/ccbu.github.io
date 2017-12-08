title: Android知识体系
author: John Doe
tags: []
categories:
  - android
date: 2017-12-04 22:17:00
---
## Android知识体系

### 基础组件

* Application
  * PackageManager
* Activity
  * Activity生命周期
  * Activity启动模式
  * Stack与Task
  * ActivityManager
* Service
  * Service创建方式（2种）
  * IntentService
  * ServiceManager
* ContentProvider
  * 联系人Demo
* BroadcastReceiver
  * 注册方式、区别
  * LocalBroadcast
* Fragment
  * 生命周期
  * Fragment的管理和事务处理
  * 创建方式
  * 与Activity通信
* Intent
  * 基础概念
  * 过滤匹配方式
* Loader
  * CursorLoader
  * AsyncTAskLoader
* Window
  * WindowManager
  * 与Activity、View关系
* View视图
### 视图控件
* 基础布局
  * LinearLayout
  * RelativeLayout
  * FrameLayout
  * TableLayout（继承自LinearLayout）
  * AbsoluteLayout（已被标注过时）
* 常用控件
  * TextView
  * Button
  * ImageView
  * ListView
  * ...
* 视图的工作原理
  * layout/measure/draw
  * VSYNC/DisplayList...
  * hwui
* 事件体系、传递机制
  * 拦截/分发/处理
  * 滑动冲突解决
  * 自定义视图
  * 继承、组合
  * Paint、Canvas使用
  
### 资源

* Resource
  * assets
  * raw
  * res
* Theme应用与管理
  * layout
  * anim
  * values
  * ...
* R文件相关
  * Drawable
  * mipmap
  