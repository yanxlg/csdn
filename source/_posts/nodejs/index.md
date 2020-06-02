---
abbrlink: 92faf83c
password: mikemessi
message: 本文暂不开放，需要密码才可阅读.
wrong_pass_message: 密码错误，请输入正确的密码.
---

`#!/usr/bin/env node`：从系统PATH中查找Node，解决不同用户Node安装问题

在开发npm包使用bin时，该行代码必须放在bin文件头部，否则，安装包后生成的.bin中可执行文件会报错


require的方法  require.resolve
