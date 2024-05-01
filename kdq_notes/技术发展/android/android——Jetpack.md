---
日期: 2024-04-23T20:35:00
作者: 孔大庆
tags:
  - android
  - jetpack
---
# 1. 什么是Jetpack
   Jetpack 是一个由多个库组成的套件，可帮助开发者遵循最佳实践、减少样板代码并编写可在各种 [Android](https://so.csdn.net/so/search?q=Android&spm=1001.2101.3001.7020) 版本和设备中一致运行的代码，让开发者可将精力集中于真正重要的编码工作。


![](https://img-blog.csdnimg.cn/img_convert/6ffc869e35a91cf6f356bd4729289c79.png)



# 2. Jetpack组成部分

## 架构（Architecture）：

由 8 个库和工具组成，它们负责构建健壮且可维护的应用程序。该组件库还有助于正确管理应用程序使用的数据以及设计应用程序架构模式。

Data Binding : 提供将应用数据与 XML 布局绑定的工具。数据绑定对于动态更新视图的数据非常有帮助。
Lifecycles : 管理 Activity 和 Fragment 生命周期，也可监听其它组件的生命周期事件。
LiveData : 通过观察者模式感知数据变化，类比 RxJava。
Navigation : 包含应用内导航所需的所有资源。
Paging : 从数据源逐渐加载数据到应用的 RecyclerView 中。
Room : 简化了在 android 应用中访问 SQLite 数据库的过程。
ViewModel : 以生命周期感知的方式促进与 UI 相关的数据管理。
WorkManager : 解决了在不同版本的 Android 中编写不同代码来管理后台任务的问题。
## 基础（foundation）：

该组件库包含了 Android 应用程序的核心系统组件。语言支持的 Kotlin 扩展和测试库也包含在其中。此外，该组件中的库提供了向后兼容性。

AppCompat： v7 库的所有组件，如 RecyclerView、GridLayout、CardView 等都包含在 AppCompat 库中。
Android KTX : Kotlin 的扩展支持库。
Multidex : 提供对应用程序的集合 dex 文件的支持，突破“65,536”限制。
Test : 这部分包括用于运行时 UI 测试的 Espresso UI 测试框架和用于在 Android 中进行单元测试的 AndroidJUnitRunner。
## 行为（behavior）：

该组件涵盖了那些使用户能够通过 UI 与应用程序交互的库。集成了 Android 标准的通知、下载、权限、分享、助手等服务。

DownloadManager : 帮助在后台下载文件。它自行管理并解决下载过程中的连接丢失、重试和系统重启等问题。
Media & Playback： 多媒体播放和一些向后兼容的 API。主要包含 MediaPalyer 和 AudioManager。
Notifications : 提供向后兼容的通知 API，支持 Wear 和 Auto
Permissions : 权限管理，用于检查和请求应用权限
Settings : Preference 相关 API。基本每个应用都会用到
Share Action : 促进与其他应用程序共享和接收信息/内容。
Slices : 可以让应用通过外部（其他 APP）显示 APP 界面（通过设备自带的搜索，语音助手等）
## 用户界面（UI）：

包括 widgets, animations, palettes 等，以改善用户体验。它还提供可在应用程序中使用的最新 emoji。

Animations and Transitions : 动画，界面转场等
Auto : 它包括用于开发 Android Auto 应用程序的组件。这些应用程序可以使用桌面主机 (DHU) 在汽车屏幕上进行测试。。
Emoji : Emoji 相关
Fragment : 是包含 ListFragment、DialogFragment、PreferenceFragmentCompat 等可组合 UI 单元的片段支持类。
Layout : 包含有关不同类型布局声明的信息，如 LinearLayout、RelativeLayout、ContraintLayout。
Palette-Colors : 该库允许开发人员在 Palette.Builder 类的帮助下创建调色板并选择不同的颜色。
TV : 包括用于开发 Android TV 应用程序的组件。
Wear： 包含用于为智能手表等可穿戴 Android 设备开发应用程序的库和类。
