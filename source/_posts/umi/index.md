---
title: umi.js
abbrlink: a96cfee5
date: 2019-11-06 11:51:44
tags:
---

definePlugins 中参数：
'process.env': { NODE_ENV: '"production"' },
  'process.env.BASE_URL': '"/"',
  __IS_BROWSER: 'false',
  __UMI_BIGFISH_COMPAT: undefined,
  __UMI_HTML_SUFFIX: 'false'
  


通过 __IS_BROWSER 可以配置ssr 对应的webpack配置，及代码中环境判断，仅在编译环境有效，
