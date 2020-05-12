---
title: umi.js
abbrlink: a96cfee5
date: 2019-11-06 11:51:44
tags:

password: mikemessi
message: 本文暂不开放，需要密码才可阅读.
wrong_pass_message: 密码错误，请输入正确的密码.
---

definePlugins 中参数：
'process.env': { NODE_ENV: '"production"' },
  'process.env.BASE_URL': '"/"',
  __IS_BROWSER: 'false',
  __UMI_BIGFISH_COMPAT: undefined,
  __UMI_HTML_SUFFIX: 'false'
  


通过 __IS_BROWSER 可以配置ssr 对应的webpack配置，及代码中环境判断，仅在编译环境有效，



umi 和 egg-bin 命令出现部分ts或es6文件转换失败，是因为`umi-core/lib/registerBabel`中配置了babel转换属性`only`，但是在`umi`中并没有传递extra属性过去
