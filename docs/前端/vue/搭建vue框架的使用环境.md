---
tags:
	- 前端框架
categories: vue
title: 搭建 vue 框架的使用环境
---
# 搭建 vue 框架的使用环境

## 安装 nodejs

node 附带了 npm 指令。

查询是否安装了 nodejs ,输入以下命令：

```
node -v
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/3C85DheJcl.png)

这里我们是需要 npm 命令才安装的 nodejs 。

## 安装 webpack

webpack：自动化。对于需要反复重复的任务，例如压缩（minification）、编译、单元测试、linting等，自动化工具可以减轻你的劳动，简化你的工作。

webpack 可以通过 https://webpack.js.org/ 中的 [DOCUMENTATION](https://webpack.js.org/concepts/) 来学习它。

### 安装 淘宝 NPM 镜像

注意：我们不使用 npm ，因为 npm 在国外速度非常慢，所以我们使用 cnpm (淘宝的镜像代替)。

```
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/CcgLK07dHe.png)

![mark](https://blogimg.nos-eastchina1.126.net/180312/69b3mleHlf.png)

命令安装 淘宝 NPM 镜像

![mark](https://blogimg.nos-eastchina1.126.net/180312/03E38F0gjj.png)

查看淘宝 NPM 镜像是否安装成功

```
cnpm
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/Ll7mc8cEIE.png)

说明安装成功。

### 安装 webpack

```
cnpm install -g webpack
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/H5CGLJhm3j.png)

测试 webpack  是否安装成功

```
webpack
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/Eg4dhkjAka.png)

## 全局安装 vue-cli

![mark](https://blogimg.nos-eastchina1.126.net/180312/d03gDH0GhL.png)

这是 vue 提供的一种创建项目架构的方式，使用命令就可以生成。

安装 vue-cli 使用命令：

```
cnpm install --global vue-cli
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/dIc4Lc7dCD.png)

安装会出现此结果。

判断是否安装成功

```
vue
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/d7FHm3jhAL.png)

说明安装成功。

## 创建项目

创建一个基于 webpack 模板的新项目

```
vue init webpack my-project
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/a1GaHF4J90.png)



## 初始化项目

生成  package.json 文件

```
cnpm install
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/7351m030Ig.png)

创建完成可以在我们的项目目录下看到 package.json 文件：

![mark](https://blogimg.nos-eastchina1.126.net/180312/KJEKblihLi.png)

完整的项目结构

![mark](https://blogimg.nos-eastchina1.126.net/180312/cB469ALikF.png)

## 运行项目

```
cnpm run dev
```

![mark](https://blogimg.nos-eastchina1.126.net/180312/h9fFk3L9gd.png)

访问：http://localhost:8080

![mark](https://blogimg.nos-eastchina1.126.net/180312/44AKB0iA5I.png)
