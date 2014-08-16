解析 (Parsing)
=======

![polish man](img/polish_man.png "A Polish Nobleman &bull; A typical Polish Notation user")

波兰表示法 (Polish Notation)
---------------

为了尝试 `mpc` 我们将要实现一种类似我们的 Lisp 中的数学运算子的简单语法。它叫做[波兰表示法 (Polish Notation)](http://en.wikipedia.org/wiki/Polish_notation)，这是一种操作符在操作数之前的算术表示方法。

例如。。。
<table class='table' style='display: inline'>
  <tr><td>`1 + 2 + 6`</td><td>*是*</td><td>`+ 1 2 6`</td></tr>
  <tr><td>`6 + (2 * 9)`</td><td>*是*</td><td>`+ 6 (* 2 9)`</td></tr>
  <tr><td>`(10 * 2) / (4 + 2)`</td><td>*是*</td><td>`/ (* 10 2) (+ 4 2)`</td></tr>
</table>

我们要制定一种语法来描述这个表示方法。我们可以从*用文字*描述开始，然后把我们所想的正式化。

要开始，我们可以观察到波兰表示法的表达式的开头总是操作符。后面跟着数字或者是其他在括号里的表达式。 也就是说我们可以说 "一个*程序*就是一个*操作符*后面跟着一个或者更多的*表达式 (expressions)*，"而 "一个 *表达式* 就是一个*数字*, 或者括号里一个*运算符*跟着一个或者更多的*表达式*"。

更加正式的。。。

<table class='table'>
  <tr><td>`Program`</td><td>*输入的开始*, 一个 `Operator`，一个或者更多 `Expression` 和输入的结尾</td></tr>
  <tr><td>`Expression`</td><td>一个 `Number` *或者* `'('`, 一个 `Operator`，一个或者更多 `Expression`, 以及一个 `')'`。</td></tr>
  <tr><td>`Operator`</td><td>`'+'`, `'-'`, `'*'`, 或者 `'/'`。</td></tr>
  <tr><td>`Number`</td><td>一个可选的 `-`，一个更多的字符介于 `0` 和 `9` 之间</td></tr>
</table>


正则表达式 (Regular Expressions)
-------------------

借助我们已经了解的东西，应该能写出上面大部分的规则，但是 *Number* 和 *Program* 可能会有一些问题。他们包含着我们还没有学过如何表达的构造。我们不知道如何表达输入的开始和结束，可选的字符，或者是一个范围的字符。

这些可以被表示，但是需要一个叫 *正则表达式 (Regular Expression)*的东西。正则表达式是描述一小段文本例如词或者数字的语法的方法。用正则表达式写的语法不能由符合规则构成，但是他们可以精确，简洁地控制什么是匹配的，什么是不匹配的。这里是一些写正则表达式的基本规则。

<table class='table'>
  <tr><td>`a`</td><td>匹配字符 `a` </td></tr>
  <tr><td>`[abcdef]`</td><td>匹配集合 `abcdef` 中的任一字符。</td></tr>
  <tr><td>`[a-f]`</td><td>匹配任一在范围 `a` 到 `f` 内的字符。</td></tr>
  <tr><td>`a?`</td><td>字符 `a` 是可选的。</td></tr>
  <tr><td>`a*`</td><td>匹配零个或者更多的字符 `a` 是。</td></tr>
  <tr><td>`a+`</td><td>匹配一个或者更多字符 `a` 。</td></tr>
  <tr><td>`^`</td><td>匹配输入的开始</td></tr>
  <tr><td>`$`</td><td>匹配输入的结束</td></tr>
</table>

这些就是我们现在需要的全部正则表达式了。[Whole books](http://regex.learncodethehardway.org/) 写了如何学习正则表达式。如果好奇的话更多的资源可以从这里在线获得。我们将在后面的章节用到它们，需要一些基础的知识，但是你不需要现在掌握它们。

在 `mpc` 的语法里我们将正则表达式写在斜杠 `/` 之间。用上面的指南我们的 *Number* 规则可以写成一个正则表达式，使用字符串 `/-?[0-9]+/`.


安装 mpc
--------------

在我们开始写这个语法之前我们首先需要 *包含* `mpc` 头，然后 *连接* `mpc` 库，就像我们在 Linux 和 Mac 下使用 `editline` 一样。 从你第四章的代码开始， 你可以将文件重命名为 `parsing.c` 并且从 [mpc repo](http://github.com/orangeduck/mpc) 下载 `mpc.h` 和 `mpc.c` 。把它们和你的源文件放到同一文件夹。

为了*包含*`mpc` 把 `#include "mpc.h"` 放在文件的顶部。为了*连接* `mpc` 把 `mpc.c` 文件夹放进编译命令。在 linux 下你还要通过添加标记 `-lm` 连接数学库。

在 **Linux** 和 **Mac** 上

```
cc -std=c99 -Wall parsing.c mpc.c -ledit -lm -o parsing
```

在 **Windows** 上

```
cc -std=c99 -Wall parsing.c mpc.c -o parsing
```

<div class="alert alert-warning">
  **等等，你的意思是不是 `#include <mpc.h>`?**

  实际上C语言有两种方法来包含文件。其中一种是用角括号 `>` ，另外一种是用引号 `""`.

  它们俩唯一的区别用角括号会先从系统路径搜寻头文件，而引号表示先从当前的文件夹搜索文件。因此系统头文件例如 `<stdio.h>` 通常放在角括号里，而私有的头文件例如 `"mpc.h"` 通常放在引号里。
</div>


波兰表示法的语法
-----------------------

进一步的形式化上面的语法，并且用一些正则表达式，我们可以像下面写出波兰表示法的最终语法。读下面的代码并且验证它是否和我们用文字写的波兰表示法的语法相同。

```c
/* 创造一些解析器 */
mpc_parser_t* Number   = mpc_new("number");
mpc_parser_t* Operator = mpc_new("operator");
mpc_parser_t* Expr     = mpc_new("expr");
mpc_parser_t* Lispy    = mpc_new("lispy");

/* 用下面的语言定义它们 */
mpca_lang(MPC_LANG_DEFAULT,
  "                                                     \
    number   : /-?[0-9]+/ ;                             \
    operator : '+' | '-' | '*' | '/' ;                  \
    expr     : <number> | '(' <operator> <expr>+ ')' ;  \
    lispy    : /^/ operator> expr>+ /$/ ;             \
  ",
  Number, Operator, Expr, Lispy);
```

我们需要把这些加到我们在第四章开始写的交互式提示符里。直接将这些代码放到 `main` 函数的开始处，在输出*版本*和*退出*信息之前。在程序的末尾我们也需要删除这些解析器。我们应该将下面的清除代码放在 `main` 返回前。

```c
/* 删除我们的解析器 */
mpc_cleanup(4, Number, Operator, Expr, Lispy);
```

<div class="alert alert-warning">
  **我得到一个错误: `undefined reference to `mpc_lang'`**

  那应该是 `mpca_lang`，有一个 `a` 在末尾!
</div>

解析用户输入
------------------

我们的代码为我们的*波兰表示法*语言创建了一个 `mpc` 解析器，但是我们实际上每次用户输入时都要*用*到它。我们需要修改我们的 `while` 循环而不是只是把用户的输入显示回去，它事实上将尝试用我们的解析器解析用户的输入。为了做到这一点，我们可以用下面的 `mpc` 代码替代调用 `printf` 函数，这用到了我们的解析器 `Lispy`.

```c
/* 尝试解析用户输入 */
mpc_result_t r;
if (mpc_parse("stdin>";, input, Lispy, &;r)) {
  /* On Success Print the AST */
  mpc_ast_print(r.output);
  mpc_ast_delete(r.output);
} else {
  /* 否则打印错误 */
  mpc_err_print(r.error);
  mpc_err_delete(r.error);
}
```

这个参数为我们的解析器 `Lispy` 和字符串 `input` 的函数叫做 `mpc_parse`。它将解析的结果拷贝进 `r` 并在成功时返回 `1`，失败时返回 `0` 。在我们将 `r` 传递给函数时我们对 `r` 使用取地址符号 `&`。这个操作符会在后面的章节中做详细的解释。

成功时一个内部结构体被复制进 `r` 的字段 `output` 里。我们可以使用 `mpc_ast_print` 打印这个结构体，并使用 `mpc_ast_delete` 删除它。

反之就会出现一个错误，它被复制进 `r` 的 `error` 字段里。 我们可以用 `mpc_err_print`打印它并且用 `mpc_err_delete` 删除它。

编译这些更新。尝试不同的输入并且看看系统如何反应。正确的表现应该看起来像下面这样。

```lispy
Lispy Version 0.0.0.0.2
Press Ctrl+c to Exit

lispy> + 5 (* 2 2)
>:
  regex:
  operator|char: '+'
  expr|number|regex: '5'
  expr|>:
    char: '('
    operator|char: '*'
    expr|number|regex: '2'
    expr|number|regex: '2'
    char: ')'
  regex:
lispy> hello
stdin>:0:0: error: expected '+', '-', '*' or '/' at 'h'
lispy> / 1dog & cat
stdin>:0:3: error: expected end of input at 'd'
lispy>
```

<div class="alert alert-warning">
  **我碰到一个错误 `stdin>:0:0: error: Parser Undefined!`.**

  这个错误是因为你提供给 `mpca_lang` 的语法有语法错误。看看你能不能找出哪一部分是错误的。你可以使用这一章提供的参考代码帮助你找出错误，并验证语法应该是什么样子。
</div>


参考
---------

<div class="panel-group alert alert-warning" id="accordion">
  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseOne">
          parsing.c
        </a>
      </h4>
    </div>
    <div id="collapseOne" class="panel-collapse collapse">
      <div class="panel-body">
```c
#include "mpc.h"

#ifdef _WIN32

static char buffer[2048];

char* readline(char* prompt) {
  fputs("lispy> ", stdout);
  fgets(buffer, 2048, stdin);
  char* cpy = malloc(strlen(buffer)+1);
  strcpy(cpy, buffer);
  cpy[strlen(cpy)-1] = '\0';
  return cpy;
}

void add_history(char* unused) {}

#else

#include editline/readline.h>
#include editline/history.h>

#endif

int main(int argc, char** argv) {

  /* Create Some Parsers */
  mpc_parser_t* Number   = mpc_new("number");
  mpc_parser_t* Operator = mpc_new("operator");
  mpc_parser_t* Expr     = mpc_new("expr");
  mpc_parser_t* Lispy    = mpc_new("lispy");

  /* Define them with the following Language */
  mpca_lang(MPC_LANG_DEFAULT,
    "                                                     \
      number   : /-?[0-9]+/ ;                             \
      operator : '+' | '-' | '*' | '/' ;                  \
      expr     : number> | '(' operator> expr>+ ')' ;  \
      lispy    : /^/ operator> expr>+ /$/ ;             \
    ",
    Number, Operator, Expr, Lispy);

  puts("Lispy Version 0.0.0.0.2");
  puts("Press Ctrl+c to Exit\n");

  while (1) {

    char* input = readline("lispy> ");
    add_history(input);

    /* Attempt to parse the user input */
    mpc_result_t r;
    if (mpc_parse("stdin>", input, Lispy, &r)) {
      /* On success print and delete the AST */
      mpc_ast_print(r.output);
      mpc_ast_delete(r.output);
    } else {
      /* Otherwise print and delete the Error */
      mpc_err_print(r.error);
      mpc_err_delete(r.error);
    }

    free(input);
  }

  /* Undefine and delete our parsers */
  mpc_cleanup(4, Number, Operator, Expr, Lispy);

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
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; 写一个正则表达式来匹配全部由 `a` 或则 `b` 组成的字符串例如 `aababa` 或者 `bbaa`。</li>
    <li class="list-group-item">&rsaquo; 写一个正则表达式来匹配由连续的 `a` 和 `b` 组成的字符串例如 `ababab` or `aba`。</li>
    <li class="list-group-item">&rsaquo; 写一个正则表达式来匹配 `pit`, `pot` 和 `respite` 但是*不匹配* `peat`, `spit`, 或者 `part`。</li>
    <li class="list-group-item">&rsaquo; 改变语法来添加一个新的运算符例如 `%`。</li>
    <li class="list-group-item">&rsaquo; 改变语法来识别文字格式的运算符例如 `add`, `sub`, `mul`, `div`。</li>
    <li class="list-group-item">&rsaquo; 改变语法来识别全部的十进制数字例如 `0.01`, `5.21`, or `10.2`。</li>
    <li class="list-group-item">&rsaquo; 改变语法使得运算符以传统的方式放在两个表达式之间。</li>
    <li class="list-group-item">&rsaquo; 使用前面章节的语法解析 `Doge`，你必须添加输入的*开始*和*结束*!</li>
  </ul>
</div>
