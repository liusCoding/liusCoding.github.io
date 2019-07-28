---
layout: post
title: Intellij IDEA开发提升效率必备的10大快捷键
categories: Idea
description: Idea常用快捷键
keywords: Idea
---
#  Intellij IDEA开发提升效率必备的10大快捷键

Idea中很多快捷键，可以快速提高效率，你常用的有哪些呢

## 1.查找打开快捷键（个人开发经常用）

Idea的<b>`Ctrl+N/Ctrl+Shift+N`<b/>可以打开类或资源。新版本的IDEA还加入了Search Everywhere功能，只需按<b>`Shift+Shift`<b>即可在一个弹出框中搜索任何东西，包括类、资源、配置项、方法等等。

类的继承关系则可用`Ctrl+H`打开类层次窗口，在继承层次上跳转则用`Ctrl+B/Ctrl+Alt+B`分别对应父类或父方法定义和子类或子方法实现，查看当前类的所有方法用`Ctrl+F12`。

要找类或方法的使用也很简单`Alt+F7`。要查找文本的出现位置就用`Ctrl+F/Ctrl+Shift+`F`在当前窗口或全工程中查找，再配合`F3/Shift+F3`前后移动到下一匹配处

## 2.智能提示快捷键

基本的代码提示用`Ctrl+Space`，还有更智能地按类型信息提示`Ctrl+Shift+Space`，但因为Intellij总是随着我们敲击而自动提示，所以很多时候都不会手动敲这两个快捷键(除非提示框消失了)。

用`F2/ Shift+F2`移动到有错误的代码，`Alt+Enter`快速修复(即Eclipse中的Quick Fix功能)。

当智能提示为我们自动补全方法名时，我们通常要自己补上行尾的反括号和分号，当括号嵌套很多层时会很麻烦，这时我们只需敲`Ctrl+Shift+Enter`就能自动补全末尾的字符。而且不只是括号，例如敲完if/for时也可以自动补上{}花括号。


## 3.重构快捷键

先说一个无敌的重构功能大汇总快捷键`Ctrl+Shift+Alt+T`，叫做Refactor This。

按法有点复杂，但也符合Intellij的风格，很多快捷键都要双手完成，而不像Eclipse不少最有用的快捷键可以潇洒地单手完成(不知道算不算Eclipse的一大优点)。

此外，还有些最常用的重构技巧，因为太常用了，若每次都在Refactor This菜单里选的话效率有些低。比如`Shift+F6`直接就是改名，`Ctrl+Alt+V`则是提取变量。

## 4.代码生成

常用的有`fori/sout/psvm+Tab`即可生成循环、System.out、main方法等boilerplate样板代码，用`Ctrl+J`可以查看所有模板。
后面“辅助”一节中将会讲到`Alt+Insert`，在编辑窗口中点击可以生成构造函数、toString、getter/setter、重写父类方法等。这两个技巧实在太常用了，几乎每天都要生成一堆main、System.out和getter/setter。

另外，Intellij IDEA 13中加入了后缀自动补全功能(Postfix Completion)，比模板生成更加灵活和强大。例如要输入for(User user : users)只需输入user.for+Tab。再比如，要输入Date birthday = user.getBirthday();只需输入user.getBirthday().var+Tab即可。

## 5.编辑快捷键

编辑中不得不说的一大神键就是能够自动按语法选中代码的`Ctrl+W`以及反向的`Ctrl+Shift+W`。

此外，`Ctrl+Left/Right`移动光标到前/后单词，`Ctrl+[/]`移动到前/后代码块，这些类Vim风格的光标移动也是一大亮点。

以上`Ctrl+Left/Right/[]`加上Shift的话就能选中跳跃范围内的代码。`Alt+Forward/Backward`移动到前/后方法。还有些非常普通的像`Ctrl+Y`删除行、`Ctrl+D`复制行、`Ctrl+</>`折叠代码就不多说了。

## 6.其他常用快捷键


以上这些神键配上一些辅助快捷键，即可让你的双手90%以上的时间摆脱鼠标：

1. 命令：`Ctrl+Shift+A`可以查找所有Intellij的命令，并且每个命令后面还有其快捷键。所以它不仅是一大神键，也是查找学习快捷键的工具。

2. 新建：`Alt+Insert`可以新建类、方法等任何东西。

3. 格式化代码：格式化import列表`Ctrl+Alt+O`，格式化代码`Ctrl+Alt+L`。

4. 切换窗口：`Alt+Num`，常用的有1-项目结构，3-搜索结果，4/5-运行调试。Ctrl+Tab切换标签页，Ctrl+E/Ctrl+Shift+E打开最近打开过的或编辑过的文件。

5. 单元测试：`Ctrl+Alt+T`创建单元测试用例。

6. 运行：`Alt+Shift+F10`运行程序，`Shift+F9`启动调试，`Ctrl+F2`停止。

7. 调试：F7/F8/F9分别对应Step into，Step over，Continue。

## 7.最终排行版
1.重构一切：`Ctrl+Shift+Alt+T`

2.自我修复：`Alt+Enter`

3.智能补全：`Ctrl+Shift+Space`

4.创造万物：`Alt+Insert`

5.自动完成：`Ctrl+Shift+Enter`

6.无处藏身：`Shift+Shift`

7.发号施令：`Ctrl+Shift+A`

10.切来切去：`Ctrl+Tab`

11.代码生成：`Template/Postfix +Tab`

12.选你所想：`Ctrl+W`



