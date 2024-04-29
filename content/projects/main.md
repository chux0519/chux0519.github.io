+++
title = "个人项目"
+++

## wayab

<video  width="100%" autoplay loop controls>
    <source src="https://user-images.githubusercontent.com/14276970/144039851-497ed32e-fd20-4b27-9c84-86b1bab2fccd.mp4" type="video/mp4">
    您的浏览器不支持 video 标签
</video>


wayland 下动态壁纸软件，支持多屏幕输出，使用 GPU 实现（初衷就是发现有个开源项目 cpu 的实现十分占用资源，于是简单实现了一版）

项目地址：[wayab](https://github.com/chux0519/wayab)
- - -

## runcat-tray

<img src="https://raw.githubusercontent.com/chux0519/runcat-tray/master/runcat.gif" width="36"/>
<img src="https://c.tenor.com/5IWFYb4D1WMAAAAi/swan_hack-dab.gif" width="36" />
<img src="https://github.githubassets.com/images/mona-loading-default.gif" width="36" />
<img src="https://cultofthepartyparrot.com/guests/hd/partyblobcat.gif" width="36" />

Linux 下的 runcat，支持自定义图标


项目地址：[runcat-tray](https://github.com/chux0519/runcat-tray)
- - -

## pegasocks

<img style="margin: 15px;" src="https://raw.githubusercontent.com/chux0519/pegasocks/master/logo/icon.svg" width="150" align="left" />

用 C 编写的轻量级代理客户端

支持多种协议（v2ray, trojan-gfw, trojan-go 等）

与其他大多数支持多协议的客户端不同

pegasocks 不依赖各种第三方 core(比如 v2ray-core 等)，而是真的去实现相关协议的拆装，并且尽可能的照顾性能。

因此它

1. 🍃 足够轻量，没有 QT 或是 boost 或是其他第三方二进制的依赖。
2. 🚀 性能优先，默认多个 worker 线程，因此理论上吞吐量会比较高（待benchmark）
3. ❌ 没有 GUI，可以直接配合 systemd, launchd, rc 或是各种自定义脚本配置开机启动。后期计划开发一个简单的 tray indicator，在系统的托盘里显示，并且提供一些简单的交互，总之重型的 GUI 是不在考虑范围内的。

项目地址：[pegasocks](https://github.com/chux0519/pegasocks)
- - -

## 像素画家

iOS 平台上的像素画编辑器

<img style="display: inline;" width=370 alignment="left" src="https://i.imgur.com/CVHC2gi.png" />
<img style="display: inline;" width=370 alignment="left" src="https://i.imgur.com/egnDUd0.png" />

项目链接：[像素画家](https://apps.apple.com/cn/app/%E5%83%8F%E7%B4%A0%E7%94%BB%E5%AE%B6/id1546046976#?platform=iphone)

- - -

## purr

是 rust 版本的 [primitive](https://github.com/fogleman/primitive) 实现，使用简单的图形，拟合输入的图片。对原始的 go 语言本本做了一些改动，运行速度会比原版的更快。

<img width="512" src="https://i.imgur.com/oGO2rnR.gif" />

另外还提供 Qt 版本的跨平台 APP（正在开发中）

<img src="https://i.imgur.com/6S6s2BB.png" />
<img src="https://i.imgur.com/CZmpXGQ.png" />

项目地址：

- [purr](https://github.com/hexyoungs/purr)
- [purrmitive-qt](https://github.com/chux0519/purrmitive-qt)

- - -

## c8-emu

是一个 chip8 模拟器，c++ 编写，利用了 Qt5 做界面，支持打开断点、单步运行等功能。

![example](https://raw.githubusercontent.com/chux0519/c8-emu/master/images/example.png)

项目地址：[c8-emu](https://github.com/chux0519/c8-emu)


