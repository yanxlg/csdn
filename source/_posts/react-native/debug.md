---
title: RN 真机调试
categories: ReactNative
abbrlink: b90162ba
date: 2019-03-26 23:47:51
password: mikemessi
message: 本文暂不开放，需要密码才可阅读.
wrong_pass_message: 密码错误，请输入正确的密码.
---
 React-Native项目在Android真机上调试？接下来直奔主题，通过USB将手机和电脑连接，打开手机上的USB调试。不通型号的手机可能设置方式不一样，这里具体不在细说……

1. 确保你的设备已经成功连接。可以终端输入adb devices来查看:
```bash
    $ adb devices

    List of devices attached

    "Your device Name" device
```


注意：为避免调试出现其他问题，此处只需有一台设备连接，如果模拟器打开需要关闭模拟器；

2. 终端运行npm start 开启本地服务，成功后运行react-native run-android来在设备上安装并启动应用，或者VSCode等编辑器进行Debug Android

应用成果安装后不出意外的话会提示无法连接服务器，如下图：
   {% asset_img img-left 6463111-2e92fdace8995f6a.webp %}
   {% asset_img img-left 6463111-e6e0e9d7e30f7e26.webp %}

出现此问题是因为我们未给手机设置访问开发服务地址，模拟器是直接访问电脑本地服务，真机则需要我们手动配置

3. 设置设备访问开发服务器

    - (Android 5.0及以上)使用adb reverse命令

        * 运行adb reverse tcp:8081 tcp:8081

        * 不需要更多配置，你就可以使用Reload JS和其它的开发选项了。

    - (Android 5.0以下)通过Wi-Fi连接你的本地开发服务器

        * 首先确保你的电脑和手机设备在同一个Wi-Fi环境下。

        * 在设备上运行你的React Native应用。和打开其它App一样操作。

        * 你应该会看到一个“红屏”错误提示。这是正常的，下面的步骤会解决这个报错。

        * 摇晃设备，或者运行adb shell input keyevent 82，可以打开开发者菜单。

        * 点击进入Dev Settings。

        * 点击Debug server host for device。

        * 输入你电脑的IP地址和端口号（电脑网络IP:8081）。查看电脑IP这里就不用多说啦。

        * 回到开发者菜单然后选择Reload JS。

 备注：理想状态下已经可以看到APP页面了，但是，如果上面步骤都已经做好，并且电脑本地服务终端已显示加载成功，但是APP的页面还未加载出来，显示白屏状态！是我们的步骤有问题？这里并不是我们的步骤有问题，此时只需要退出正在运行的APP，重新打开即可，就可以成功加载到APP页面啦！

4. 调试
   摇晃手机打开app的开发菜单，可以看到有两个选项，一个是Enable live reload另一个是Enable hot reloading。Enable live reload表示刷新时全局刷新,而hot reloading是局部刷新。这两个我们都选择允许后，我们改完代码并保存，可以实时看到修改效果，不用重新编译运行或者 reload。
 {% asset_img img-left 5146067-7aabe3ae51b7f0a6.png %}
  在 app 设置 菜单中选择 Debug JS Remotely，同时PC端打开localhost:8081/debugger-ui 页面即可查看控制台，并且可以在debuggerWorker.js中设置断点进行调试

