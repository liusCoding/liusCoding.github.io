---
layout: post
title: 这一次彻底掌握Intellij IDEA中Debug的使用
categories: Idea
description: 写的第一篇博客。
keywords: Idea,Debug
---

熟练掌握Idea  Debug的使用

## 一、Idea Debug 介绍

1. 以Debug模式启动服务，左边的一个按钮则是以Run模式启动。在开发中，一般会直接启动Debug模式，方便随时调试代码。
2. 断点：在左边行号栏单击左键，或者快捷键Ctrl+F8 打上/取消断点，断点行的颜色可自己去设置。
3. Debug窗口：访问请求到达第一个断点后，会自动激活Debug窗口。如果没有自动激活，可以去设置里设置，如图1.2。
4. 调试按钮：一共有8个按钮，调试的主要功能就对应着这几个按钮，鼠标悬停在按钮上可以查看对应的快捷键。在菜单栏Run里可以找到同样的对应的功能，如图    1.4。
5. 服务按钮：可以在这里关闭/启动服务，设置断点等。
6. 方法调用栈：这里显示该线程调试所经过的所有方法，勾选右上角的[Show All Frames]按钮，就不会显示其它类库的方法了，否则这里会有一大堆的方法。
7. Variables：在变量区可以查看当前断点之前的当前方法内的变量。
8. Watches：查看变量，可以将Variables区中的变量拖到Watches中查看 


图1.1
![Markdown](http://i2.tiimg.com/695115/4a3fadf30a61c565.png)

图1.2在设置里勾选Show debug window on breakpoint，则请求进入到断点后自动激活Debug窗口
![Markdown](http://i2.tiimg.com/695115/0dbf8fa20806fa1d.png)

图1.3如果你的IDEA底部没有显示工具栏或状态栏，可以在View里打开，显示出工具栏会方便我们使用。可以自己去尝试下这四个选项。
![Markdown](http://i2.tiimg.com/695115/7ac7cc20b16a804e.png)

图1.4菜单栏Run里有调试对应的功能，同时可以查看对应的快捷键。

![QQ截图20190731101326.png](https://i.loli.net/2019/07/31/5d40fc64dd96534624.png)

## 二、基本用法与快捷键

Debug调试的功能主要对应着图一中4和5两组按钮：

1.首先说第一组按钮，共有8个按钮，从左到右依次为：
图2.1
![QQ截图20190731103408.png](https://i.loli.net/2019/07/31/5d40fe63ba96e35359.png)

1.Show Exception Point(Alt + F10):如果你的光标在其它行或者其它的页面，点击这个按钮可跳转到当前代码执行的行。
2.Step Over(F8): 跳过，一行一行的向下走，如果当前行有方法不进入方法。
3.Step Into(F7): 跳入，如果当前有方法，进入方法内部，一般用于进入自定义的方法，不会jdk类库的方法。
4.Force Step Into(Alt+Shift+F7)：强制跳入，可以进入任何方法，想查看底层源代码可以用这个进入jdk的方法。
5.Step Out(Shift + F8):跳出，从跳入的方法内退出到方法调用处，这个时候方法已经执行完毕，只是还没有完成赋值。
6.Drop  Frame(默认无快捷键)：回退断点。
7.Run to Cursor(Alt + F9): 运行到光标处，你可以将光标定位到你需要查看的行，然后使用这个功能，代码会运行到光标处，不需要打断点。
8.Evaluate Expression(Alt + F8): 计算表达式的值。

图2.2
![QQ截图20190731112835.png](https://i.loli.net/2019/07/31/5d410af776cca54397.png)
2.第二组的按钮，共7个按钮，从上到下依次如下：
 
  
1.Return  `xxx`:  重新运行程序，会关闭服务后重新启动程序。
2.Update ‘Tomcat xx’ application(Ctrl + f5): 更新程序，一般在你的代码有改动后可以执行这个功能，而这个功能对应的操作则是在服务配置里，如图2.3
3.Resume Program(F9): 恢复程序，比如，你在第10行和20行各自都打了断点，当前运行到第10行，按F9,则运行到下一个断点即第20行，再按F9，则运行完整个流程，因为后面已经没有断点了。
4.Pause Program: 暂停程序，启用Debug。
5.Stop 'xxx'(Ctrl + F2): 连续按两下，关闭程序。有时候你会发现服务再启动时，报端口被占用，这是因为没完全关闭服务的原因，你就需要查杀所有JVM进程了。
6.View Breakpoints(Ctrl + First+ F8): 查看你设置的所有断点。
7.Mute BreakPoints: 哑的断点，选择这个后，所有断点变为灰色，断点失效，再按F9则可以直接运行完程序。再次点击，断点变为红色，有效。如果只想使某一个断点失效，可以在断点上有点取消Enable，如图2.4，则该行断点失效。

更新程序，On 'Update' actions, 执行更新操作时所做的事情，一般选择'Update classes resouces' 即更新类和资源。
  一般配合热部署插件会更好用，如JRebel,这样就不用每次更改代码后还要去重新启动服务。
  
<b>如何激活Jrebel呢?</b>
参考下面这篇博客：
[Jrebel最新激活破解方式(持续更新) - BlueKitty的博客 - CSDN博客](https://blog.csdn.net/xingbaozhen1210/article/details/81093041)

图2.3
![C3}5D{EBN$KS}EYLTRG5ZWX.png](https://i.loli.net/2019/07/31/5d410f3c8dc1272400.png)

图2.4
![QQ截图20190731114939.png](https://i.loli.net/2019/07/31/5d410ffd6c2c342405.png)


## 三、变量查看
在Debug过程中，跟踪查看变量的变化是非常必要的，这里就简单说下IDEA中可以查看变量的几个地方，相信大部分人都了解。

1.如下图，在Idea中，参数所在行的后面会显示当前变量的值。
    ![QQ截图20190731115927.png](https://i.loli.net/2019/07/31/5d4112473c3b747113.png)
    
2.光标悬停到参数上，显示当前变量信息。点击打开详情如图。
![QQ截图20190731120218.png](https://i.loli.net/2019/07/31/5d4112e644b1d42607.png)

3.在Variables里查看，这里显示当前方法里的所有变量。
![QQ截图20190731120501.png](https://i.loli.net/2019/07/31/5d41137ad74ab73868.png)

4.在Watches里，点击New Watch，输入需要查看的变量。或者可以从Variables里拖到Watche里查看。
![QQ截图20190731120652.png](https://i.loli.net/2019/07/31/5d4113f6c028385762.png)

## 四、计算表达式

在前面提到的计算表达式如下图的按钮，快捷键是Evaluate Expression (Alt + F8) 。可以使用这个操作在调试过程中计算某个表达式的值，而不用再去打印信息。
![QQ截图20190731120954.png](https://i.loli.net/2019/07/31/5d41149de674761948.png)

1.按Alt + F8或按钮，或者，你可以选中某个表达式再Alt + F8，弹出计算表达式的窗口，如下，回车或点击Evaluate计算表达式的值。
   这个表达式不仅可以是一般变量或参数，也可以是方法，当你的一行代码中调用了几个方法时，就可以通过这种方式查看查看某个方法的返回值
   
![QQ截图20190731121344.png](https://i.loli.net/2019/07/31/5d4115d303b1083067.png)


## 五、智能进入
如果一行代码里有好几个方法，怎么才能只选择某一个方法进入。之前提到过使用Step Into (Alt + F7) 或者 Force Step Into (Alt + Shift + F7)进入到方法内部，但这两个操作会根据方法调用顺序依次进入，这比较麻烦。

那么智能进入就很方便了，智能进入，这个功能可以在菜单栏Run里面看到，Smart Step Into（Shift + F7） 如下图：
![5d415a6bc9ca293776.png](https://i.loli.net/2019/07/31/5d416492495f622887.png)


按Shift + F7，会自动定位到当前断点行，并列出需要进入的方法，如下图,选中需要进入的方法。
![QQ截图20190731171009.png](https://i.loli.net/2019/07/31/5d415b0856ced46743.png)

如果只有一个方法，则直接进入，类似Force Step Into。

## 六、断点条件设置

通过设置断点条件，在满足条件时，才停在断点处，否则直接运行。

通常，当我们在遍历一个比较大的集合或数组时，在循环内设置了一个断点，难道我们要一个一个去看变量的值？那肯定很累，说不定你还错过这个值得重新来一次

1.在断点上右键直接设置当前断点的条件，如下图，我设置b==false时断点才生效。

![QQ截图20190731172904.png](https://i.loli.net/2019/07/31/5d4172398fda011274.png)

2.点击View Breakpoints (Ctrl + Shift + F8)，查看所有断点。

 Java Line Breakpoints 显示了所有的断点，在右边勾选Condition，设置断点的条件。

   勾选'BreakPoint hit message'，则会将当前断点行输出到控制台，如下图
    
![QQ截图20190731174901.png](https://i.loli.net/2019/07/31/5d41642d6b97891646.png)
   
   勾选Evaluate and log，可以在执行这行代码是计算表达式的值，并将结果输出到控制台,如下图。
    
 ![QQ截图20190731174457.png](https://i.loli.net/2019/07/31/5d416329055af30083.png)

3.再说下其他三个的功能，一般情况不用
Instance filters：实例过滤

Class filters：类过滤，根据类名过滤

Pass count：用于循环中，如果断点在循环中，可以设置该值，循环多少次后停在断点处，之后的循环都会停在断点处。