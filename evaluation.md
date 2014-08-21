 求值(Evaluation)
==========


树(Tree)
-----

现在我们可以读取输入，并且在内部用结构体表示，但是我们仍然不能对其求值。在这个章节我们添加对结构体求值的代码，并且真正的执行里面的计算。

这个内部结构体就是我们在之前的章节看到的打印出来的。它叫做一个 *抽象语法树*，它表示了用户输入的程序的结构。在它的叶子上是数字和操作符 - 实际上要处理的数据。在它的分支上则是产生这部分树的规则 - 关于如何遍历和计算它的信息。

![tree](img/tree.png "Abstract Christmas Tree &bull; A seasonal variation")

在解决我们如何遍历之前，我们来看一看这个结构体在内部是如何定义的。如果我们看一看 `mpc.h` 的内容我们可以看到 `mpc_ast_t` 的定义， 这是我们从解析器得到的数据结构。


```c
typedef struct mpc_ast_t {
  char* tag;
  char* contents;
  int children_num;
  struct mpc_ast_t** children;
} mpc_ast_t;
```

这个结构体有几个我们可以使用的域。让我们一个一个的看看。

第一个域是 `tag`。当我们打印这个树的时候这是在节点内容钱的信息。这是一个字符串，包含一个由用来解析特定项的规则构成的列表，例如 `expr|number|regex`。

`tag` 域很重要因为它让我们看到节点的类型。

第二个节点是 `contents`。它会包含当前节点的内容例如 `'*'`, `'('` 或者 `'5'`。你会发现对于分支它是空的，但是对于叶我们可以用它来找到运算符或者数字。

最后我们看两个可以帮助我们遍历树的域。它们是 `children_num` 和 `children`。第一个域告诉我们一个节点有多少子节点，第二个是这些子节点组成的数组。

`children` 域的类型是 `mpc_ast_t**`。这是一个双重指针类型。它并不像看起来那么可怕并且会在后面的章节详细解释。现在你可以把它看做这个树的子节点组成的数组。

我们可以通过使用数组符号访问这个域来访问一个子节点。 只要在域的名字 `children` 后加上一对包含要访问的子节点序号的方括号就可以做到。例如访问第一个子节点我们可以使用 `children[0]`。注意 C 语言的数组序号从 0 开始计数。

因为 `mpc_ast_t*` 的类型是一个指向结构体的 *指针* ，所以访问它的域有一点不同。我们需要用一个箭头 `->` 而不是一个点 `.`。 采取这个符号没有什么原因，所以我们现在只要记住访问一个指针类型的域使用箭头。

```c
/* 从输出加载 AST(抽象语法树) */
mpc_ast_t* a = r.output;
printf("Tag: %s\n", a->tag);
printf("Contents: %s\n", a->contents);
printf("Number of children: %i\n", a->children_num);

/* 获取第一个子节点 */
mpc_ast_t* c0 = a->children[0];
printf("First Child Tag: %s\n", c0->tag);
printf("First Child Contents: %s\n", c0->contents);
printf("First Child Number of children: %i\n", c0->children_num);
```


递归(Recursion)
---------

树的结构很有趣。它指向他自己。他们的每一个子节点又是一个树，子节点的子节点又是树。就像我们的语言，重写规则，结构体的数据中又包含重复的子结构体，子结构体又和母节点一样。

![recursion](img/recursion.png "Recursion &bull; Dangerous in a fire")

这种重复子结构体的模式还能继续下去。如果我们想要一个函数能够对所有的树有效，我们不能只看几个节点，我们必须让它对任何深度的树都有效。

幸运的是我们可以做到这一点，通过利用子结构体重复的性质和一个叫做 *递归(recursion)* 的技术。

简单的 *递归函数(recursive function)* 就是它将自己作为运算的一部分调用。

一个函数自己定义自己听起来有些怪异。但是考虑到函数会根据不同的输入给出不同的输出。如果我们做成一些改变，或者给不同的输入给一个调用同样函数的递归，并且提供一个方法让这个函数在当前条件下不要再次调用自己，我们就更加肯定*递归函数*能够做一些有用的事情。

作为一个例子我们可以写出一个可以统计我们的树结构的节点数目的递归函数。

我们从搞清楚它在大多数简单的情况下如何表现开始，如果输入的树没有子节点。在这种情况下我们知道结果就是 1。现在我们可以继续定义更加复杂的情况，如果树有一个或者更多子节点。在这种情况下结果是 1 (节点本身)加上全部子节点的节点数目。

但是我们如何找出全部子节点的节点数目? 我们可以使用我们正在定义的函数! *是的，递归*

在 C 语言里我们可能把它写成类似下面的代码。

```c
int number_of_nodes(mpc_ast_t* t) {
  if (t->children_num == 0) { return 1; }
  if (t->children_num >= 1) {
    int total = 1;
    for (int i = 0; i < t->children_num; i++) {
      total = total + number_of_nodes(t->children[i]);
    }
    return total;
  }
}
```

递归函数看起来很怪异。首先我们需要假定我们已经有一些正确工作的函数，然后我们就要使用这个函数，来写我们假定已经有的最初代码!

像大多数事情一样，递归函数几乎总是以一个相似的方式结束。首先一个定义 *base case*。这是终止递归的情况，例如我们前面的例子中 `t->children_num == 0` 。然后定义 *recursive case* ，例如我们前面的例子中的 `t->children_num >= 1` 。它将计算过程分成更小的部分，然后递归的调用他自己来计算这一部分，最后将他们并在一起。

递归函数可能需要思考一会，所以暂停下来保证你在继续其他章节前理解了它们，因为我们在树下剩下的部分将要好好的利用他们。如果你仍然不确定，你可以挑一些本章的附加题作为联系。


求值(Evaluation)
----------

为了计算解析树我们将要写一个递归函数。但是在我们开始之前，让我们试试并看看作为输入的树结构有什么特点。试着用你上一章的程序打印一些表达式。看看有什么能注意的。

```lispy
lispy> + 1 (* 5 4)
>:
  regex:
  operator|char: '+'
  expr|number|regex: '1'
  expr|>:
    char: '('
    operator|char: '*'
    expr|number|regex: '5'
    expr|number|regex: '4'
    char: ')'
  regex:
```

其中一个可以观察到的是一个用 `number` 标记的节点总是一个数字，没有子节点，并且我们可以直接将它的内容转换成一个数字。就像我们的递归中的 *base case*。

如果一个节点被用 `expr` 标记，而且*并不是* `number`，我们需要看看第二个子节点(第一个子节点总是`'()'`) 看一看它是哪一个运算符。然后我们需要使用这个运算符来*计算*剩下的子节点，不包括总是 `')'` 的最后一个子节点。 这是我们的 *recursive case*。根节点也需要这样做。

当我们计算我们的树时, 就像统计节点的数目一样，我们需要累计结果。为了表示结果我们要使用 C 类型 `long`，这表示一个*长**整数*。

为了探测节点的标记，或者从节点获取一个数字，我们将要利用 `tag` 和 `contents` 域。 这些是*字符串*域，所以我们先要学一些字符串函数。

<table class='table'>
  <tr><td>`atoi`</td><td>将一个 `char*` 转换成 `long`.</td></tr>
  <tr><td>`strcmp`</td><td>比较两个 `char*` 若它们相等则返回 `0`。</td></tr>
  <tr><td>`strstr`</td><td>接受两个 `char*` 作为输入，并返回第二个字符串在第一个字符串中的位置，如果第二个字符串不是第一个字符串的子字符串则返回 `0` 。</td></tr>
</table>

我们可以使用 `strcmp` 来检查用的是哪一个运算符，用 `strstr` 来检查一个标记(tag)是否包含一个子字符串。总的来说，我们的递归函数看起来像这样。 

```c
long eval(mpc_ast_t* t) {

  /* 如果标记为 number 则直接返回它，否则为表达式。 */
  if (strstr(t->tag, "number")) { return atoi(t->contents); }

  /* 运算符总是第二个子节点。 */
  char* op = t->children[1]->contents;

  /* 我们将第三个子节点存储在 `x` 中 */
  long x = eval(t->children[2]);

  /* 迭代剩下的子节点，用我们的运算符将它们合并 */
  int i = 3;
  while (strstr(t->children[i]->tag, "expr")) {
    x = eval_op(x, op, eval(t->children[i]));
    i++;
  }

  return x;
}
```

我们可以像下面这样定义 `eval_op` 函数。它接受一个数字，一个运算符的字符串，和另外一个数字作为参数。它测试被传进来的是哪一个运算符，并执行对应的 C 运算符。

```c
/* 使用运算符字符串来看看要执行那个运算符 */
long eval_op(long x, char* op, long y) {
  if (strcmp(op, "+") == 0) { return x + y; }
  if (strcmp(op, "-") == 0) { return x - y; }
  if (strcmp(op, "*") == 0) { return x * y; }
  if (strcmp(op, "/") == 0) { return x / y; }
  return 0;
}
```


打印(Printing)
--------

我们现在希望能够打印计算的结果而不是树。因此我们需要将树传递给我们的 `eval` 函数，为了打印结果我们需要使用 `printf` 和指示符 `%li`，这用于 `long` 类型。

我们还要记得在完成计算后删除输出的树We also need to remember to delete the output tree after we are done evaluating it.

```c
long result = eval(r.output);
printf("%li\n", result);
mpc_ast_delete(r.output);
```

如果这些都成功了我们应该能够用我们的新语言做一些基础的数学运算了！

```lispy
Lispy Version 0.0.0.0.3
Press Ctrl+c to Exit

lispy> + 5 6
11
lispy> - (* 10 10) (+ 1 1 1)
97
lispy> - (/ 10 2) 20
-15
lispy>
```


参考
---------

<div class="panel-group alert alert-warning" id="accordion">
  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseOne">
          evaluation.c
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

#include <editline/readline.h>
#include <editline/history.h>

#endif

/* Use operator string to see which operation to perform */
long eval_op(long x, char* op, long y) {
  if (strcmp(op, "+") == 0) { return x + y; }
  if (strcmp(op, "-") == 0) { return x - y; }
  if (strcmp(op, "*") == 0) { return x * y; }
  if (strcmp(op, "/") == 0) { return x / y; }
  return 0;
}

long eval(mpc_ast_t* t) {

  /* If tagged as number return it directly, otherwise expression. */
  if (strstr(t->tag, "number")) { return atoi(t->contents); }

  /* The operator is always second child. */
  char* op = t->children[1]->contents;

  /* We store the third child in `x` */
  long x = eval(t->children[2]);

  /* Iterate the remaining children, combining using our operator */
  int i = 3;
  while (strstr(t->children[i]->tag, "expr")) {
    x = eval_op(x, op, eval(t->children[i]));
    i++;
  }

  return x;
}

int main(int argc, char** argv) {

  mpc_parser_t* Number = mpc_new("number");
  mpc_parser_t* Operator = mpc_new("operator");
  mpc_parser_t* Expr = mpc_new("expr");
  mpc_parser_t* Lispy = mpc_new("lispy");

  mpca_lang(MPC_LANG_DEFAULT,
    "                                                     \
      number   : /-?[0-9]+/ ;                             \
      operator : '+' | '-' | '*' | '/' ;                  \
      expr     : <number> | '(' <operator> <expr>+ ')' ;  \
      lispy    : /^/ <operator> <expr>+ /$/ ;             \
    ",
    Number, Operator, Expr, Lispy);

  puts("Lispy Version 0.0.0.0.3");
  puts("Press Ctrl+c to Exit\n");

  while (1) {

    char* input = readline("lispy> ");
    add_history(input);

    mpc_result_t r;
    if (mpc_parse("<stdin>", input, Lispy, &r)) {

      long result = eval(r.output);
      printf("%li\n", result);
      mpc_ast_delete(r.output);

    } else {
      mpc_err_print(r.error);
      mpc_err_delete(r.error);
    }

    free(input);

  }

  mpc_cleanup(4, Number, Operator, Expr, Lispy);

  return 0;
}<
```
      </div>
    </div>
  </div>
</div>

附加题
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; 写一个递归函数来计算树叶的数目。</li>
    <li class="list-group-item">&rsaquo; 写一个递归函数来计算树的分支的数目。</li>
    <li class="list-group-item">&rsaquo; 写一个递归函数来计算一个分支中子节点出现次数最多的数字。</li>
    <li class="list-group-item">&rsaquo; 你如何使用 `strstr` 来看一个节点是否被标记为 `expr`。</li>
    <li class="list-group-item">&rsaquo; 你如何使用 `strcmp` 来看一个节点的内容是否为 `'('` 或者 `')'`。</li>
    <li class="list-group-item">&rsaquo; 增加运算符 `%`, 返回除法的余数。例如 `% 10 6` 的结果是 `4`。</li>
    <li class="list-group-item">&rsaquo; 增加运算符 `^`, 进行次方运算。例如 `^ 4 2` 的结果是 `16`。</li>
    <li class="list-group-item">&rsaquo; 增加运算符 `min`, 返回最小的数字。例如 `min 1 5 3` 的结果是 `1`。</li>
    <li class="list-group-item">&rsaquo; 增加运算符 `max`, 返回最大的数字。例如 `max 1 5 3` 的结果是 `5`。</li>
    <li class="list-group-item">&rsaquo; 改变运算符 `-`，当它只接收到一个参数时对其取反。</li>
  </ul>
</div>
