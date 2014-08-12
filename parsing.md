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


Polish Notation Grammar
-----------------------

Formalising the above rules further, and using some regular expressions, we can write a final grammar for the language of polish notation as follows. Read the below code and verify that it matches what we had written textually, and our ideas of polish notation.

```c
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
    expr     : <number> | '(' <operator> <expr>+ ')' ;  \
    lispy    : /^/ operator> expr>+ /$/ ;             \
  ",
  Number, Operator, Expr, Lispy);
```

We need to add this to the interactive prompt we started on in chapter 4. Put this code right at the beginning of the `main` function before we print the *Version* and *Exit* information. At the end of our program we also need to delete the parsers when we are done with them. Right before `main` returns we should place the following clean-up code.

```c
/* Undefine and Delete our Parsers */
mpc_cleanup(4, Number, Operator, Expr, Lispy);
```

<div class="alert alert-warning">
  **I'm getting an error `undefined reference to `mpc_lang'`**

  That should be `mpca_lang`, with an `a` at the end!
</div>

Parsing User Input
------------------

Our new code creates a `mpc` parser for our *Polish Notation* language, but we still need to actually *use* it on the user input supplied each time from the prompt. We need to edit our `while` loop so that rather than just echoing user input back, it actually attempts to parse the input using our parser. We can do this by replacing the call to `printf` with the following `mpc` code, that makes use of our program parser `Lispy`.

```c
/* Attempt to Parse the user Input */
mpc_result_t r;
if (mpc_parse("stdin>";, input, Lispy, &;r)) {
  /* On Success Print the AST */
  mpc_ast_print(r.output);
  mpc_ast_delete(r.output);
} else {
  /* Otherwise Print the Error */
  mpc_err_print(r.error);
  mpc_err_delete(r.error);
}
```

This code calls the `mpc_parse` function with our parser `Lispy`, and the input string `input`. It copies the result of the parse into `r` and returns `1` on success and `0` on failure. We use the address of operator `&` on `r` when we pass it to the function. This operator will be explained in more detail in later chapters.

On success a internal structure is copied into `r`, in the field `output`. We can print out this structure using `mpc_ast_print` and delete it using `mpc_ast_delete`.

Otherwise there has been an error, which is copied into `r` in the field `error`. We can print it out using `mpc_err_print` and delete it using `mpc_err_delete`.

Compile these updates, and take this program for a spin. Try out different inputs and see how the system reacts. Correct behaviour should look like the following.

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
  **I'm getting an error `stdin>:0:0: error: Parser Undefined!`.**

  This error is due to the syntax for your grammar supplied to `mpca_lang` being incorrect. See if you can work out what part of the grammar is incorrect. You can use the reference code for this chapter to help you find this, and verify how the grammar should look.
</div>


Reference
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


Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Write a regular expression matching strings of all `a` or `b` such as `aababa` or `bbaa`.</li>
    <li class="list-group-item">&rsaquo; Write a regular expression matching strings of consecutive `a` and `b` such as `ababab` or `aba`.</li>
    <li class="list-group-item">&rsaquo; Write a regular expression matching `pit`, `pot` and `respite` but *not* `peat`, `spit`, or `part`.</li>
    <li class="list-group-item">&rsaquo; Change the grammar to add a new operator such as `%`.</li>
    <li class="list-group-item">&rsaquo; Change the grammar to recognize operators written in textual format `add`, `sub`, `mul`, `div`.</li>
    <li class="list-group-item">&rsaquo; Change the grammar to recognize decimal numbers such as `0.01`, `5.21`, or `10.2`.</li>
    <li class="list-group-item">&rsaquo; Change the grammar to make the operators written conventionally, between two expressions.</li>
    <li class="list-group-item">&rsaquo; Use the grammar from the previous chapter to parse `Doge`. You must add *start* and *end* of input!</li>
  </ul>
</div>
