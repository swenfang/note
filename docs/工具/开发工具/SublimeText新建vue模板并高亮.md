---
tags:
	- 开发工具使用
categories: Sublime Text
title: Sublime Text新建.vue模板并高亮
---
# Sublime Text新建.vue模板并高亮

## 准备工作

- 下载安装新建文件模板插件 `SublimeTmpl`
- 下载安装vue语法高亮插件 `Vue Syntax Highlight`

##### Sublime Text安装插件的方法有两种：

1. 使用Sublime Text自带的安装库 `Package Control` 去安装
   点击菜单栏的 `Preferences -> Package Control` 或使用快捷键 `CTRL+SHIFT+P` 打开终端窗口，输入`Install`选择`Package Control: Install Package`来安装
2. 下载直接放入包目录 `(Preferences / Browse Packages)` `中文:(首选项 / 包浏览器)` 文件夹里面
   - [SublimeTmpl](https://github.com/kairyou/SublimeTmpl)
   - [Vue Syntax Highlight](https://github.com/vuejs/vue-syntax-highlight)

## 创建.vue模板并让语法高亮

安装完`Vue Syntax Highlight`之后，你打开`.vue`格式的文件就已经可以高亮了，我们现在来设置用快捷键直接创建`.vue`格式的文件。

###### `SublimeTmpl` 默认只有6种语法模板：

- html `ctrl+alt+h`
- javascript `ctrl+alt+j`
- css `ctrl+alt+c`
- php `ctrl+alt+p`
- ruby `ctrl+alt+r`
- python `ctrl+alt+shift+p`

###### 我们现在新增创建 `vue` 格式的模板

1. 创建`vue`文件模板

   - 直接打开插件包的文件夹 `Preferences -> Browse Packages`

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/ckm0BJkjl9.png)

   ​

   首选项 -> 浏览程序包

   ​

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/JBE3g91b7a.png)

   ​

   包文件夹

   - 打开存放模板的文件夹 `templates`，随便复制一项，改名为`vue.tmpl`

   ​

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/8mc9145hab.png)

   创建vue.tmpl

   - `vue.tmpl`内容改为你想要的模板

   ​

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/JhL8j52hdC.png)

   vue.tmpl内容

2. 修改新建菜单，增加新建`vue`选项

   - `SublimeTmpl`新建菜单默认是没有`vue`的，如图

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/EhKlBifH5b.png)

   ​

   新建 -> New File (SublimeTmpl)

   - 点击上图的 `Menu` 选项，或者打开 `Preferences -> Package Settings -> SublimeTmpl -> Settings - Menu`，如图

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/aebbHgC3jG.png)

   ​

   打开菜单配置项

   - 复制一项，然后粘贴修改为 `vue` 项，如图

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/L4hKlbbIm7.png)

   ​

   新增vue项

   - 保存修改，就会在新建菜单里面出现`vue`项，如图

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/Fkl7AbGj6e.png)

   ​

   出现vue项

   - 点击上图`vue`新建项，就会出现之前设置的模板内容，只不过没有语法高亮，并且是纯文本格式，如图

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/DDD66EB98b.png)

   ​

   新建vue文件

3. 模板绑定vue语法高亮

   - 打开 `Preferences -> Package Settings -> SublimeTmpl -> Settings - Default`，如图

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/JCecEJgji4.png)

   ​

   打开默认设置项

   - 复制一项并修改为vue，路径如下

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/Jgf4ef10f5.png)

   ​

   绑定vue语法

   - 绑定语法关联文件路径请查看目录 `Sublime Text3\Data\Cache`，寻找vue高亮语法插件名，并打开，如图

   ​

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/a3d1AcCf25.png)

   Sublime Text3\Data\Cache目录

   ​

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/67kf30ea7f.png)

   ​

   Sublime Text3\Data\Cache\vue-syntax-highlight

   - 再次菜单新建vue就语法高亮了，如图

   ​

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/1lA8G1g1Gd.png)

   新建vue文件

4. 绑定新建`vue`文件快捷键

   - 打开 `Preferences -> Package Settings -> SublimeTmpl -> Key Bindings - Default`，如图

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/CICf0f6gad.png)

   ​

   打开设置快捷键文件

   - 复制一项，粘贴创建新建vue快捷键为 `ctrl+alt+v`，如图

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/DCDLl85ejk.png)

   ​

   创建快捷键

   - 保存后，菜单新建里也有了，如图

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/eLF86ba49j.png)

   ​

   新建文件菜单

   - 试试，完美！

   ![mark](https://blogimg.nos-eastchina1.126.net/180310/f1jmIeJlka.png)

   ​

   完美

#### 最后

`Preferences -> Package Settings -> SublimeTmpl -> Settings - Commands` 文件好像是配置命令的，配置方法也跟上面相同，照猫画虎即可~

#### 最后的最后

通过这种方法，其他的语言模板也可以自己去创建。
