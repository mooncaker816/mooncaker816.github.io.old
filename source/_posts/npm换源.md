---
title: npm换源
---

### npm - Node Package Manager

npm 淘宝源

- 搜索地址：<http://npm.taobao.org/> （淘宝NPM使用说明）
- registry地址：<http://registry.npm.taobao.org/>

持久更换

``` bash
$ npm config set registry https://registry.npm.taobao.org
```

临时更换并安装程序

``` bash
$ npm --registry https://registry.npm.taobao.org install express
```





