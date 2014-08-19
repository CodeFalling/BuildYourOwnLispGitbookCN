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

To begin we work out how it will act in the most simple case, if the input tree has no children. In this case we know the result is simply one. Now we can go on to define the more complex case, if the tree has one or more children. In this case the result will be one (for the node itself), plus the number of nodes in all of those children.

But how do we find the number of nodes in all of the children? Well we can use the function we are in the process defining! *Yeah, Recursion.*

In C we might write it something like this.

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

Recursive functions are weird because they require an odd leap of faith. First we have to assume we have some function which does something correctly already, and then we have to go about using this function, to write the initial function we assumed we had!

Like most things, recursive functions almost always end up following a similar pattern. First a *base case* is defined. This is the case that ends the recursion, such as `t->children_num == 0` in our previous example. After this the *recursive case* is defined, such as `t->children_num >= 1` in our previous example, which breaks down a computation into smaller parts, and calls itself recursively to compute those parts, before combining them together.

Recursive functions can take some thought, so pause now and ensure you understand them before continuing onto other chapters because we'll be making good use of them in the rest of the book. If you are still uncertain, you can check out some of the bonus marks for this chapter to practice.


Evaluation
----------

To evaluate the parse tree we are going to write a recursive function. But before we get started, let us try and see what observations we can make about the structure of the tree we get as input. Try printing out some expressions using your program from the previous chapter. See what you can notice.

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

One observation is that if a node is tagged with `number` it is always a number, has no children, and we can just convert the contents to an integer. This will act as the *base case* in our recursion.

If a node is tagged with `expr`, and is *not* a `number`, we need to look at its second child (the first child is always `'('`) and see which operator it is. Then we need to apply this operator to the *evaluation* of the remaining children, excluding the final child which is always `')'`. This is our *recursive case*. This also needs to be done for the root node.

When we evaluate our tree, just like when counting the nodes, we'll need to accumulate the result. To represent this result we'll use the C type `long` which means a *long* *integer*.

To detect the tag of a node, or to get a number from a node, we will need to make use of the `tag` and `contents` fields. These are *string* fields, so we are going to have to learn a couple of string functions first.

<table class='table'>
  <tr><td>`atoi`</td><td>Converts a `char*` to a `long`.</td></tr>
  <tr><td>`strcmp`</td><td>Takes as input two `char*` and if they are equal it returns `0`.</td></tr>
  <tr><td>`strstr`</td><td>Takes as input two `char*` and returns a pointer to the location of the second in the first, or `0` if the second is not a sub-string of the first.</td></tr>
</table>

We can use `strcmp` to check which operator to use, and `strstr` to check if a tag contains some substring. Altogether our recursive evaluation function looks like this.

```c
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
```

We can define the `eval_op` function as follows. It takes in a number, an operator string, and another number. It tests for which operator is passed in, an performs the corresponding C operation on the inputs.

```c
/* Use operator string to see which operation to perform */
long eval_op(long x, char* op, long y) {
  if (strcmp(op, "+") == 0) { return x + y; }
  if (strcmp(op, "-") == 0) { return x - y; }
  if (strcmp(op, "*") == 0) { return x * y; }
  if (strcmp(op, "/") == 0) { return x / y; }
  return 0;
}
```


Printing
--------

Instead of printing the tree we now want to print the result of the evaluation. Therefore we need to pass the tree into our `eval` function, and print the result we get using `printf` and the specifier `%li`, which is used for `long` type.

We also need to remember to delete the output tree after we are done evaluating it.

```c
long result = eval(r.output);
printf("%li\n", result);
mpc_ast_delete(r.output);
```

If all of this is successful we should be able to do some basic maths with our new programming language!

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


Reference
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

Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Write a recursive function to compute the number of leaves of a tree.</li>
    <li class="list-group-item">&rsaquo; Write a recursive function to compute the number of branches of a tree.</li>
    <li class="list-group-item">&rsaquo; Write a recursive function to compute the most number of children spanning from one branch of a tree.</li>
    <li class="list-group-item">&rsaquo; How would you use `strstr` to see if a node was tagged as an `expr`.</li>
    <li class="list-group-item">&rsaquo; How would you use `strcmp` to see if a node had the contents `'('` or `')'`.</li>
    <li class="list-group-item">&rsaquo; Add the operator `%`, which returns the remainder of division. For example `% 10 6` is `4`.</li>
    <li class="list-group-item">&rsaquo; Add the operator `^`, which raises one number to another. For example `^ 4 2` is `16`.</li>
    <li class="list-group-item">&rsaquo; Add the function `min`, which returns the smallest number. For example `min 1 5 3` is `1`.</li>
    <li class="list-group-item">&rsaquo; Add the function `max`, which returns the biggest number. For example `max 1 5 3` is `5`.</li>
    <li class="list-group-item">&rsaquo; Change the minus operator `-` so that when it receives one argument it negates it.</li>
  </ul>
</div>
