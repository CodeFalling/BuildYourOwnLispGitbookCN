交互式提示符(interactive prompt)
=====================


读取，计算，打印(Read, Evaluate, Print)
---------------------

![reptile](img/reptile.png "Reptile &bull; Sort of like REPL")


既然我们打造我们自己的编程语言，我们需要和它交互的方法。C语言使用了一个编译器，所以你可以修改程序，重新编译然后运行它。如果我们能做的更好并且能够动态的和编程语言交互那就更棒了。然后我们就很快的在若干条件下测试它的特性。为了做到这一点我们要构造一个叫做*交互式提示符(interactive prompt)*的东西。

这是一个程序，它可以提示用户输入，并且得到输入后返回一些信息。用这个是测试我们编程语言的最简单的方法。这种系统也叫 *REPL*, 代笔 *read(读)*-*evaluate(计算)*-*print(打印)* *loop(循环)*。这是常用的一种和编程语言交互的方法，你可能在例如 *Python* 的其他语言里使用过。

在建立一个完整的 *REPL* 之前我们将从一些简单的事情开始。我们将构造一个系统，它可以提示用户，并且直接将输入返回给用户。 如果我们做到这一点，我们以后就可以扩展它来解析用户输入并计算，就像一个实际的 Lisp 程序一样。


一个交互式终端
---------------------

为了一个基本的体系，我们将要写一个循环，它反复的写出消息，然后等待输入。为了获取用户的输入我们可以用一个叫 `fget` 的函数，它读取输入直到遇到换行符。我们需要找一个地方来存储用户输入。为了这一点我们可以声明一个不变大小的输入缓冲区(buffer)。 

一旦我们存储了用户的输入后就可以把它输出给用户，这要用到一个叫做 `printf` 函数。

```c
#include <stdio.h>

/* 声明一个静态缓冲区，允许用户最大的大小为2048的输入 */
static char input[2048];

int main(int argc, char** argv) {

  /* 输出版本和退出信息 */
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");

  /* 一个永远不会停下的循环 */
  while (1) {

    /* 输出提示 */
    fputs("lispy> ", stdout);

    /* 读取一行用户的输入，最大大小2048 */
    fgets(input, 2048, stdin);

    /* Echo input back to user */
    printf("No you're a %s", input);
  }

  return 0;
}
```

<div class="alert alert-warning">
  **这个灰色的文本是什么?**

  上面包含*注释*的代码。它们是在符号 `/*` 和 `*/` 之间的部分, 注释会被编译器忽略，但是被用于告诉阅读这段代码的人到底是怎么回事。注意它们！
</div>

让我们深入的复习一下这个程序。

`static char input[2048];` 这一行声明了一个公有的2048个字符数组。这是一个保留的数据块，我们可以在程序的任何地方使用它。我们将要把用户在命令行里的输入存储到里面。`static` 这个关键词(keyword)让这个变量属于这个文件局部，并且 `[2048]` 这个部分声明了它的大小。

我们用 `while (1)` 写了一个无限循环。条件块 1 总是等同于 true(真)。因此循环里的命令将会永远运行。

为了在我们使用函数 `fputs` 时能够输出我们的提示。这和 `puts` 略有不同，它并不尾部追加一个换行符。我们用 `fgets` 函数来获取用户在命令行的输入。这两个函数需要一些文件来读写。为了这一点我们提供了两个特殊的变量 `stdin` 和 `stdout`。它们被声明在 `<stdio.h>` 里，而且是特殊的文件变量，代表着从命令行的输入输出。当传递这个变量的时候函数 `fputs`将会等待用户输入一行文本，得到输入后便将它存储在 `input` 里，包括换行符。所以 `fgets` 并不会读取太多数据，我们提供一个大小为2048的缓冲区便足够了。

为了将消息显示给用户我们需要用到函数 `printf`。这个函数提供了一个方法可以打印由不同元素组成的信息。它按照给定的格式字符串来匹配参数。例如在我们的例子中我们在格式字符串里可以看到 `%s`。这意味着它将被接下来传递的参数替代，解释为一个字符串。

关于更多关于 `printf` 的不同格式字符串的信息请看 [documentation](http://en.cppreference.com/w/c/io/printf)

<div class="alert alert-warning">
  **我如何了解像 `fgets` 和 `printf`这样的函数呢?**

  如何了解这些标准函数，以及什么时候使用它们并不是显而易见的。当面对一个问题时知道有什么已经解决了您的问题的库函数是需要经验的。

  幸运的是C语言有一个非常小的标准库而且它们中的大部分可以在练习中被学习。如果你想做一些基础或者根本的事情，那你值得看一看标准库的[参考文档](http://en.cppreference.com/w/c)，并检查这里是否有你想要的函数。
</div>


编译
-----------

你可以使用和第二章中一样的命令编译

```shell
cc -std=c99 -Wall prompt.c -o prompt
```

在编译后你可以试着运行它。但你完成的时候你可以使用 `Ctrl+c` 来退出程序。如果一切都正确的话你应该可以像这样运行你的程序

```lispy
Lispy Version 0.0.0.0.1
Press Ctrl+c to Exit

lispy> hello
No You're a hello
lispy> my name is Dan
No You're a my name is Dan
lispy> Stop being so rude!
No You're a Stop being so rude!
lispy>
```


编辑输入
-------------

如果你正在 Linux 或者是 Mac 下工作你将会发现在你输入时使用方向键时有一些奇怪的表现。

```lispy
Lispy Version 0.0.0.0.3
Press Ctrl+c to Exit

lispy> hel^[[D^[[C
```

用方向键会产生一些奇怪的字符: `^[[D` 或者 `^[[C`, 而不是在编辑区移动光标。我们真正想要的是，在我们弄错的情况下能够移动，删除和编辑输入。

在 Windows 下这种行为是默认的。在 Linux 和 Mac 下这个功能由一个叫做 `editline` 的库。在 Linux 和 Mac 下我们用这个库提供的函数替换我们的 `fputs` 和 `fgets` 。

如果你正在 Windows 下开发，而只是想要继续下去，那么可以跳到这一章的结尾，因为接下来的几节可能不相关。

使用 Editline
--------------

`editline` 库 提供两个我们要用的函数，叫做 `readline` 和 `add_history`。第一个函数, `readline` 是用来从提示符读取输入的,但是它允许编辑输入。第二个韩式 `add_history` 让我们记录输入的历史一变我们可以通过上下键再次使用之前的输入。

我们用这些函数取代了 `fputs` 和 `fgets` 得到了下面的代码

```c
#include <stdio.h>
#include <stdlib.h>

#include <editline/readline.h>
#include <editline/history.h>

int main(int argc, char** argv) {

  /* 打印版本信息和退出信息 */
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");

  /* 无限循环 */
  while (1) {

    /* 输出提示符并得到输入 */
    char* input = readline("lispy> ");

    /* 将输入记录进历史 */
    add_history(input);

    /* 将输入反过来显示给用户 */
    printf("No you're a %s\n", input);

    /* 释放 input */
    free(input);

  }

  return 0;
}
```


我们 *included(包含)* 一些新的 *(头文件)headers*. 这里有 `#include <stdlib.h>`, 让我们可以使用 `free` 函数在代码的最后。我们还添加了 `#include <editline/readline.h>` 和 `#include <editline/history.h>` ，这让我们可以使用 `editline` , `r eadline` 和 `add_history` 三个函数。

我们用一个 `readline` 取代了提示，并且用 `fgets` 来获得输入。我们将结果传递给 `add_history` 来记录它。 最后我们像之前一样使用 `printf` 来打印它。

不像 `fgets`, `readline` 函数去掉了输入结尾的换行符，所以我们需要把换行符加到我们的 `printf` 函数中。我们还需要使用 `free` 来删除 `readline` 给我们的输入。因为它并不像 `fgets` 一样写入到一个已经存在的缓冲区，`readline` 函数被调用时申请新的空间。 When to free memory is something we cover in depth in later chapters.

Compiling with Editline
-----------------------

If you try to compile this right away with the previous command you'll get an error. This is because you first need to install the `editline` library on your computer.


```
fatal error: editline/readline.h: No such file or directory #include <editline/readline.h>
```

On **Linux** this can be done using the command `sudo apt-get install libedit-dev`. On Fedora or similar you can use the command `su -c "yum install libedit-dev*"`

On **Mac** the `editline` library should have been installed alongside the *Command Line Tools*. If you get an error about the history header not being found you can either try removing that line of code or installing the `readline` library, which can be used as a drop-in replacement. This can be installed using Homebrew or MacPorts.

Once you have installed this you can try to compile it again. This time you'll get a different error.

```
undefined reference to `readline'
undefined reference to `add_history'
```

This means that you haven't *linked* your program to `editline`. This *linking* process allows the compiler to directly embed calls to `editline` in your program. You can make it link by adding the flag `-ledit` to your compile command, just before the output flag.

```shell
cc -std=c99 -Wall prompt.c -ledit -o prompt
```

Hopefully now you should be able to *compile* and *link* your program with `editline`. Run it and check that now you can edit inputs as you type them in.

<div class="alert alert-warning">
  **It's still not working!**

  Some systems might have slight variations on how to install, include, and link to `editline`. For example on Mac and some other systems the history header may not be required and so that line of code can be removed. On Arch linux the editline history header is `histedit.h`. If you are having trouble search online and see if you can find distribution specific instructions on how to use `editline` or `readline`, an equivalent library.
</div>


The C Preprocessor
------------------

For such a small project it might be okay that we have to program differently depending on what operating system we are using, but if I want to send my source code to a friend on different operating system to give me a hand with the programming, it is going to cause problem. In an ideal world I'd wish for my source code to be able to compile no matter where, or on what computer, it is being compiled. This is a general problem in C, and it is called *portability*. There is not always an easy or correct solution.

![octopus](img/octopus.png "Octopus &bull; Sort of like Octothorpe")

But C does provide a mechanism to help, called *the preprocessor*.

The preprocessor is a program that runs before the compiler. It has a number of purposes, and we've been actually using it already without knowing. Any line that starts with a octothorpe `#` character (hash to you and me) is a preprocessor command. We've been using it to *include* header files, giving us access to functions from the standard library and others.

Another use of the preprocessor is to detect which operating system the code is being compiled on, and to use this to emit different code.

This is exactly how we are going to use it. If we are running Windows we're going to let the preprocessor emit code with some fake `readline` and `add_history` functions I've prepared, otherwise we are going to include the headers from `editline` and use these.

To declare what code the compiler should emit we can wrap it in `#ifdef`, `#else`, and `#endif` preprocessor statements. These are like an `if` function that happens before the code is compiled. All the contents of the file from the first `#ifdef` to the next `#else` are used if the condition is true, otherwise all the contents from the `#else` to the final `#endif` are used instead. By putting these around our fake functions, and our editline headers, the code that is emitted should compile on Windows, Linux or Mac!

```c
#include <stdio.h>
#include <stdlib.h>

/* If we are compiling on Windows compile these functions */
#ifdef _WIN32

#include <string.h>

static char buffer[2048];

/* Fake readline function */
char* readline(char* prompt) {
  fputs(prompt, stdout);
  fgets(buffer, 2048, stdin);
  char* cpy = malloc(strlen(buffer)+1);
  strcpy(cpy, buffer);
  cpy[strlen(cpy)-1] = '\0';
  return cpy;
}

/* Fake add_history function */
void add_history(char* unused) {}

/* Otherwise include the editline headers */
#else

#include <editline/readline.h>
#include <editline/history.h>

#endif

int main(int argc, char** argv) {

  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");

  while (1) {

    /* Now in either case readline will be correctly defined */
    char* input = readline("lispy> ");
    add_history(input);

    printf("No you're a %s\n", input);
    free(input);

  }

  return 0;
}
```


Reference
---------

<div class="panel-group alert alert-warning" id="accordion">
  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseOne">
          prompt_windows.c
        </a>
      </h4>
    </div>
    <div id="collapseOne" class="panel-collapse collapse">
      <div class="panel-body">
```c
#include <stdio.h>

/* Declare a static buffer for user input of maximum size 2048 */
static char input[2048];

int main(int argc, char** argv) {

  /* Print Version and Exit Information */
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");

  /* In a never ending loop */
  while (1) {

    /* Output our prompt */
    fputs("lispy> ", stdout);

    /* Read a line of user input of maximum size 2048 */
    fgets(input, 2048, stdin);

    /* Echo input back to user */
    printf("No you're a %s", input);
  }

  return 0;
}
```
      </div>
    </div>
  </div>

  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseTwo">
          prompt_unix.c
        </a>
      </h4>
    </div>
    <div id="collapseTwo" class="panel-collapse collapse">
      <div class="panel-body">
```c
#include <stdio.h>
#include <stdlib.h>

#include <editline/readline.h>
#include <editline/history.h>

int main(int argc, char** argv) {

  /* Print Version and Exit Information */
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");

  /* In a never ending loop */
  while (1) {

    /* Output our prompt and get input */
    char* input = readline("lispy> ");

    /* Add input to history */
    add_history(input);

    /* Echo input back to user */
    printf("No you're a %s\n", input);

    /* Free retrived input */
    free(input);

  }

  return 0;
}
```
      </div>
    </div>
  </div>

  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseThree">
          prompt.c
        </a>
      </h4>
    </div>
    <div id="collapseThree" class="panel-collapse collapse">
      <div class="panel-body">
```c
#include <stdio.h>
#include <stdlib.h>

/* If we are compiling on Windows compile these functions */
#ifdef _WIN32

#include <string.h>

static char buffer[2048];

/* Fake readline function */
char* readline(char* prompt) {
  fputs("lispy> ", stdout);
  fgets(buffer, 2048, stdin);
  char* cpy = malloc(strlen(buffer)+1);
  strcpy(cpy, buffer);
  cpy[strlen(cpy)-1] = '\0';
  return cpy;
}

/* Fake add_history function */
void add_history(char* unused) {}

/* Otherwise include the editline headers */
#else

#include <editline/readline.h>
#include <editline/history.h>

#endif

int main(int argc, char** argv) {

  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");

  while (1) {

    /* Now in either case readline will be correctly defined */
    char* input = readline("lispy> ");
    add_history(input);

    printf("No you're a %s\n", input);
    free(input);

  }

  return 0;
}
```
      </div>
    </div>
  </div>

</div>


Bonus Marks
-----------

<div class="alert alert-warning">
<ul class="list-group">
  <li class="list-group-item">&rsaquo; Change the prompt from `lispy>` to something of your choice.</li>
  <li class="list-group-item">&rsaquo; Change what is echoed back to the user.</li>
  <li class="list-group-item">&rsaquo; Add an extra message to the *Version* and *Exit* Information.</li>
  <li class="list-group-item">&rsaquo; What does the `\n` mean in those strings?</li>
  <li class="list-group-item">&rsaquo; What other patterns can be used with `printf`.</li>
  <li class="list-group-item">&rsaquo; What happens when you pass `printf` a variable does not match the pattern?</li>
  <li class="list-group-item">&rsaquo; What does the preprocessor command `#ifndef` do?</li>
  <li class="list-group-item">&rsaquo; What does the preprocessor command `#define` do?</li>
  <li class="list-group-item">&rsaquo; If `_WIN32` is defined on windows, what is defined for Linux or Mac?</li>
</ul>
</div>
