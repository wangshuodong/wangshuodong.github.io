
# docsify笔记

> 官方文档：[官方文档](https://docsify.js.org/#/zh-cn/quickstart)

```shell
全局安装、初始化、启动
npm i docsify-cli -g
docsify init ./docs
docsify serve docs
```

## 淘宝镜像

> 永久替换淘宝镜像

```shell
npm config set registry https://registry.npm.taobao.org
```

> 安装yarn

```shell
npm install -g yarn

yarn config set registry https://registry.npm.taobao.org -g
yarn config set sass_binary_site http://cdn.npm.taobao.org/dist/node-sass -g
```

> 查看镜像

```shell
npm get registry
```
