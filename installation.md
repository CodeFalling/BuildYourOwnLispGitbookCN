安装和配置
============


安装
-----

![capttop](img/cattop.png "Cat &bull; Install at own risk")

在我们开始用C语言编程之前，我们需要安装一些东西和配置我们的环境。因为C是如此通用的编程语言所以这些事情应该会比较简单。实质上我们要安装两个主要的东西。一个*文本编辑器*和一个*编译器*。


文本编辑器
-----------

文本编辑器是一种让你用适合编程的方法来编辑文本文件的程序。

在 **Linux** 上我推荐的文本编辑器是 [gedit](http://projects.gnome.org/gedit/)。任何和你的发行版一起安装的基础的文本编辑器也可以很好的工作。如果你是一名 Vim 或者是 Emacs 的用户这些都很好。请不要使用 IDE。在这样一个小项目中并不需要它，并且不利于理解这是怎么一回事。

在 **Mac** 上一个可以使用的简单的文本编辑器是 [TextWrangler](http://www.barebones.com/products/textwrangler/)。如果你有不同的偏好也也不错，但是请不要使用 XCode 来编辑文本，这是一个小项目并且使用 IDE 不利于你理解这是怎么回事。

在 **Windows** 上我选择的文本编辑器是 [Notepad++](http://notepad-plus-plus.org/)。如果你有其他偏好的选择也不错。 请*不要*使用 *Visual Studio* 因为它对 C 语言编程没有很好的支持。如果你试图使用它你将会遇到很多问题。


编译器
--------

*编译器*是一个程序，它可以将C语言源代码转换成可以运行的程序。安装它的过程是不同的，这取决于你所运行的操作系统。

编译和运行C程序也需要一些命令行的基本用法。这方面我将不会涉及，所以我会假设你至少熟悉使用命令行， 如果你担心这方面的问题可以在网上搜索信息使用它， 这和你的操作系统有关。

在 **Linux** 上你可以通过下载一些包来安装一个编译器。如果你正在使用Ubuntu 或者 Debian 你可以通过下面的命令安装你需要的东西 `sudo apt-get install build-essential`. 如果你正在使用Fedora或者一个类似的 Linux 变种你可以使用下面的命令`su -c "yum groupinstall development-tools"`.

在 **Mac** 上你可以通过安装并下载最新版的 XCode 来安装一个编译器。如果你不确定如何做这些事你可以在网上搜索"安装xcode"并且按照这些建议来。 你需要安装*命令行工具(Command Line Tools)*. 在 Mac OS X 10.9 上可以通过从命令行运行命令 `xcode-select --install` 来完成这些. 在 Mac OS X 10.9 之前的版本可以在 XCode Preferences, Downloads, 并选择 *Command Line Tools* 来安装

在 **Windows** 上你可以下载并安装 [MinGW.](http://www.mingw.org/")来安装一个编译器。如果你在使用安装器有时候它还会给你一个可能的包的列表。确保你至少选中了 `mingw32-base` 和 `msys-base`。一旦安装后你需要添加编辑器和其他程序的的路径到你的 `PATH` 系统变量中. 按照 [这些指令](href="http://www.computerhope.com/issues/ch000549.htm")将  `;C:\MinGW\bin` 附加到叫做 `PATH` 的系统变量中。你可以创造这个变量如果它不存在。你可能需要重启 `cmd.exe` 来应用这些改变。这将会允许你在 `cmd.exe` 的命令行中运行一个编译器。他也会安装其他程序使得`cmd.exe`表现得更像一个 Unix 命令行。


测试编译器
--------------------

通过在命令行里运行下面的命令来测试你的C语言编译器是否被正确安装。

`cc --version`
如果你得到一些编译器版本信息，那么编译器应该被正确安装了。你准备好了！如果你得到任何形式的错误或是未找到命令，那么它并没有被准备好，你可能需要重启命令行或者你的电脑来使这些改变生效。


Hello World
-----------

现在你的环境已经配置好了，打开你的文本编辑器并且输入下面的程序。创造一个文件夹，把这个文件保存为 `hello_world.c`。这是你的第一个C程序！

```c
#include <stdio.h>

int main(int argc, char** argv) {
  puts("Hello, world!");
  return 0;
}
```

这些看起来也许像一大堆疯狂的符号，几乎没有意义，我将试图一步一步的解释它。

在第一行我们 *include* 的叫做*头(header)*。这个*声明(statement)*允许我们使用来自 `stdio.h` 的函数，这是是C语言的标准输入输出库。你可以在这段程序中看到的这个库的其中一个函数 `puts` 。

接下来我们*声明(declare)*一个叫做`main`的函数。这个函数被声明返回一个`int`,并且接受一个叫做`argc`的`int`和叫做`argv`的`char**`。所有的C程序都要包含这个函数。所有的程序从这个函数开始运行。

在 `mian` 函数里 `puts` 函数被*调用(called)*，参数(argument)是`"Hello,world!"`。这输出 `Hello,world!` 这条信息到命令行。函数 `puts` 是 *put string* 的缩写。函数中的第二个语句是`return 0`.这告诉 `main` 函数结束并返回 0。这表示这个程序的运行没有错误。


Compilation
-----------
在我们运行这个成寻之前我们需要编译它，这将产生真正可以在我们电脑上运行的 *可执行文件(executable)*。打开命令行并浏览至 `hello_world.c` 保存的文件夹。你可以执行下面的命令来编译你的程序

`cc -std=c99 -Wall hello_world.c -o hello_world`

这将编译 `hello_world.c` 里的代码，报告任何一个警告(warning),并且输出程序为一个叫做 `hello_world` 的新文件。我们用 `std=c99` 标记(flag)来告诉编译器我们在用C语言的哪一个*版本(version)*或者*标准(standard)*。这让编译器确信我们的代码是标准化的，所以用不同操作系统和编译器的人可以使用我们的代码。

如果成功你应该看到当前文件夹输出的文件。你可以输入 `./hello_world`(或者`hello_world`在 Windows 下)。如果没有什么错误的话你应该可以看到一个友好的 `Hello,world!` 消息出现。

**祝贺！** 你已经编译运行了你的第一个C语言程序


错误
------
如果有一些错误你的C语言程序编译过程可能会失败。这些问题的范围可能从简单的语法错误到其他难以理解的复杂错误。

有时候编译器的错误信息可能会有意义，但是难以理解你可以尝试在网上搜索它。你应该看看能不能找到一个简洁的解释，并找出如何改正它。记住这一点：有很多人都曾面临和你一样的问题。

![smash](img/smash.png "Rage &bull; A poor debugging technique")

有时候很多错误源于一个源头。所以你最好总是从第一个问题处理起。

有时候编译器会编译一个程序，但是当你运行它的时候程序会崩溃。调试(Debug) C程序在这种情况下很困难。它是一种艺术，远远超出了这本书的范围。

如果你是一名初学者，我常用的调试一个崩溃的C程序的办法是在它还在运行时输出大量的信息。用这种方法你可以试着分离代码的哪一部分是正确的和是否有哪部分出了什么错误。对于初学者我推荐这项技术。这是一种*敏捷(active)*的调试方法。这是件很重要的事。只要你在做一些事，而不是只是盯着代码，这个过程就不会那么痛苦，放弃的欲望就减小了。

> 第一句的原文是:If you are a beginner, my first port of call for debugging a crashing C program would be to print out lots of information as the program is running. 
> 不知道 first port of 在这里怎么翻译，若有知道请告知

对于更加自信的人来说一个叫做 `gdb` 的程序可以用来调试你的C程序。这可能很复杂难用，但是它也很强大而且能给你非常有价值的信息和错误以及产生错误的位置。关于如何使用 `gdb` 的信息可以在网络上找到。

在 Mac 上最新版本的 OS X 没有自带 `gdb`。作为替代你可以用 `lldb` 做同样的工作。

在 **Linux** 或者 **Mac** `valgrind` 可以帮助调试内存泄露和其他更严重的错误。Valgrind 是一个可以节约你数小时，甚至数天调试时间的工具。它并不难学会，所以强烈建议研究一下它。关于如何使用的信息可以在网上找到。


文档
-------------
在这本书中你可能会遇到你不认识的一些函数或者一些示例代码。你可能会想这是什么。在这种情况下看看这些标准库的[在线文档](http://en.cppreference.com/w/c)。这可以解释标准库的所有函数，它们做了什么，和怎么用它们。


Reference
---------

<div class="alert alert-warning">
  **这一节是干什么的?**

在这一节我会链接到我特别为这一章所写的代码。当完成了一章后你的代码看起来会跟我的相似。这个代码可以作为参考如果解释的不够清楚。

如果遇到 bug 请不要复制粘贴我的代码进你的工程。尝试自己找出 bug 并且用我的代码作为参考来突出什么是错的和错误在哪。

</div>

<div class="panel-group alert alert-warning" id="accordion">
  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseOne">
          hello_world.c
        </a>
      </h4>
    </div>
    <div id="collapseOne" class="panel-collapse collapse">
      <div class="panel-body">
```c
#include <stdio.h>

int main(int argc, char** argv) {
  puts("Hello, world!");
  return 0;
}
```
      </div>
    </div>
  </div>
</div>


附加题
-----------

<div class="alert alert-warning">
  **这一节是干什么的?**

  在这一节我会列出一些可以尝试的事情，去娱乐或是为了更好的学习

  如果你可以尝试这些挑战会非常好。有一些会比较容易，但是有些会非常困难。所以如果你没有解决它们，不要担心，有些甚至是不可能的！

  很多问题需要在网上搜索。这是学习一门新语言不可或缺的一部分，所以不应该被回避。自学能力是最有价值的编程技能之一。看看你每章能完成多少。当你觉得无聊的时候就离开。

 
</div>

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo;  在你的程序里将 `Hello World!` 改成其他东西</li>
    <li class="list-group-item">&rsaquo; 当没有 `main` 函数的时候发生了什么</li>
    <li class="list-group-item">&rsaquo; 使用在线文档查看 `puts` 函数</li>
    <li class="list-group-item">&rsaquo; 查查如何使用 `gdb`，并用它调试你的程序 </li>
  </ul>
</div>
