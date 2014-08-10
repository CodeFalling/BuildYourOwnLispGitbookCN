语言
=========


什么是编程语言?
-------------------------------

一个编程语言和真实的语言是很相似的。它背后有一个结构和一些规则。当我们阅读和书写自然语言时，我们不知不觉便学到了这些规则， 编程语言也是这样的。我们可以利用规则来理解他人，说话或者写出代码。

20世纪50年代语言学家 *Noam Chomsky* 形式化了大量关于语言的 [重要观察](http://en.wikipedia.org/wiki/Chomsky_hierarchy)。这些中的许多构成了我们今天的语言理解的基础。其中的一个就是观察自然语言的递归建立了递归和重复的子结构。

![cat](img/cat.png "Cat &bull; cannot speak or program")

作为一个例子，我们可以看看下面这个短语。

&rsaquo; `The cat walked on the carpet.`

用英语的规则，名词 `cat` 可以被两个由 `and` 连接的名词替代。

&rsaquo; <code>The <strong>cat and dog</strong> walked on the carpet.</code>

每一个新名词也可以被再次取代。我们可以用和之前一样的规则，用两个用 `and` 连接的词替代 `cat` 。 或者我们可以用一个不同的规则，为了给他们增加一点描述，用一个有形容词和一个名词来取代每一个名词。

&rsaquo; <code>The <strong>cat and mouse</strong> and dog walked on the carpet.</code>

&rsaquo; <code>The <strong>white cat</strong> and <strong>black dog</strong> walked on the carpet.</code>

这只是两个例子，但是英语对于单词种类的变化，操作和替换有很多不同的规则。

我们注意到这一点也精确的表现在编程语言中。在C语言里，`if` 语句里包含着一系列新的语句。每一个新语句也可以是另外一个 `if` 语句。这个重复的结构和替换都反应在语言的各个部位。 这些叫做 *(重写规则)re-write rules*，因为它们告诉你一个东西怎样能被另外一些东西 *重写* 。

&rsaquo; `if (x > 5) { return x; }`

&rsaquo; <code>if (x > 5) { <strong>if (x > 10) { return x; }</strong> }</code>

*Noam Chomsky* 观察的意义是重要的。它意味着虽然我们可以用特定的语言说或者写无限多的东西,并且可以用有限的重写规则来处理和理解它们。这些重写规则的集合叫做一个 *(语法)grammar*.

我们有几个方法来描述重写规则。其中一个方法就是原文。我们可能说一些像, "一个*句子*必需包含一个*动词短语*"，或者"*动词短语* 可以是一个*动词*或者一个*副词*和一个*动词*"。这个方法对人类来说不错，但是对于计算机来说理解起来太糢糊。编程时我们需要写更加正式的语法说明。

为了写一个编程语言例如我们的 Lisp ，我们需要理解它的语法。为了阅读用户的输入我们需要写一个*语法*来描述它。然后我们就可以在用户输入的时候使用它，来判断输入是否有效。我们还可以用它构建结构化的内部表达， 这将让*理解*它，*计算*它，在内部表达计算机编码都更加简单。

于是一个叫 `mpc` 的库便有了用武之地。


解析器组合子(Parser Combinators)
------------------

`mpc` 是一个可以自己写 *解析器组合子* 库。这意味着这个库可以让你构建能够理解和处理特定语言的程序。 这些叫做*(解析器)parsers*. 有很多不同的方法可以构造解析器，用一个*解析器组合子*库的优点是你构建*解析器*的过程将变得简单，只需要指定*语法* ... 之类的。

很多*解析器组合子*库实际上只是让你写一些*看起来有点像*一个语法的代码，而不是真正的直接指定一个语法。在很多情况下这是可以的，但是很多情况下可能会显得笨重而复杂。幸运的是 `mpc` 让我们写*看起来像*一个语法的代码， *或者* 我们可以用特殊的符号来直接写一个语法!


代码的语法
---------------

那么，什么是*看起来像*一个语法的代码*? 让我们看一看 `mpc`，通过试着为一个语法写一些代码，这种语法能够识别 [the language of Shiba Inu](http://knowyourmeme.com/memes/doge). 一种如同 *Doge* 一样的口语。这种语言我们可以通过下面的代码定义。

&rsaquo; 一个 *形容词(Adjective)* 是  *"wow"*, *"many"*, *"so"* 或者 *"such"*.
&rsaquo; 一个 *名词(Noun)* 是 *"lisp"*, *"language"*, *"c"*, *"book"* 或者 *"build"*.
&rsaquo; 一个 *短语(Phrase)* 是一个 *形容词* 跟着一个*名词*.
&rsaquo; 一个 *Doge* 是零个或者更多的 *短语*.

我们可以从尝试定义*形容词*和*名词*开始。为了做这些我们要创造两个新的解析器， 表示为 `mpc_parser_t*` 类型，我们将它存储在变量 `Adjective` 和 `Noun` 中。我们用函数 `mpc_or` 来创造一个解析器，其中一个选项应该被用到，并且用函数 `mpc_sym` 来包装我们的初始字符串。

如果你眯着眼睛你可以试着阅读下面这段代码，就像它是我们在上面提过的规则一样。

```c
/* 构造一个新解析器 'Adjective' 来识别描述 */
mpc_parser_t* Adjective = mpc_or(4,
  mpc_sym("wow"), mpc_sym("many"),
  mpc_sym("so"),  mpc_sym("such")
);

/* 构造新解析器 'Noun' 来识别东西 */
mpc_parser_t* Noun = mpc_or(5,
  mpc_sym("lisp"), mpc_sym("language"),
  mpc_sym("c"),    mpc_sym("book"),
  mpc_sym("build")
);
```

<div class="alert alert-warning">
  **我如何使用 `mpc` 的函数?**

  目前不用担心或者运行这一章的任何实例代码。只要阅读它并且看看你能不能理解语法背后的原理。下一章我们将会安装 `mpc` 并且使用它。
</div>

为了定义 `短语(Phrase)` 我们可以参考我们已经存在的短语。我们需要使用函数 `mpc_and`，这指定了一个东西是必要的然后再加上其他东西。作为输入我们传递 `Adjective` 和 `Noun` 给它，我们早先定义的解析器。这个函数也可以接受 `mpcf_strfold` 和 `free` 做为参数，这些表明如何加入或删除这些解析器的结果。目前忽略这些函数。


```c
mpc_parser_t* Phrase = mpc_and(2, mpcf_strfold, Adjective, Noun, free);
```

为了定义一个 *Doge* 我们必需指定 *零个或者更多* 的一些解析器是必需的。为了这一点我们需要使用函数 `mpc_many`。正如前面所说的，这个函数要求特定的变量 `mpcf_strfold` 来表明这些结果如何连接在一起，我们可以忽略它。

```c
mpc_parser_t* Doge = mpc_many(mpcf_strfold, Phrase);
```

通过创造一个解析器来寻找*一个或者更多*其他的解析器时一件很酷的事情发生了。我们的 `Doge` 解析器接受任意长度的输入。这意味这它的语言是*无限的*。 这里有一些 `Doge` 可以接受的字符串的例子。就像我们在这一章的第一节发现的，我们可以用有限数目的规则创造一个无限的语言。

```c
"wow book such language many lisp"
"so c such build such language"
"many build wow c"
""
"wow lisp wow c many language"
"so c"
```

如果我们用更多的 `mpc` 函数，我们可以慢慢地建立解析器来解析越来越复杂的语言。The code we use *sort of* reads like a grammar, but becomes much more messy with added complexity. 因此，采取这种方法就不一定是一个简单的任务了。有一些有帮助的函数的文档都在[mpc repository](http://github.com/orangeduck/mpc)，它们建立在简单的结构上，可以让繁杂的任务变得简单。这对于复杂的语言是一个简单的方法，它提供了一个细粒度很好的控制，但对我们的需求来说并不是必需的。 


自然语法
----------------

`mpc` 也允许我们写形式更加自然的语法。我们可以在一个长字符串里定义所有的事而不是用C函数这种看起来更加不像。在使用这种方法时我们不需要担心如何通过像 `mpcf_strfold`, 或者 `free` 这样的函数添加或废弃输入。 这些都被自动完成了！

下面是我们如何用这种方法重建上面的例子。

```c
mpc_parser_t* Adjective = mpc_new("adjective");
mpc_parser_t* Noun      = mpc_new("noun");
mpc_parser_t* Phrase    = mpc_new("phrase");
mpc_parser_t* Doge      = mpc_new("doge");

mpca_lang(MPC_LANG_DEFAULT,
  "                                                                     \
    adjective : \"wow\" | \"many\" | \"so\" | \"such\";                 \
    noun      : \"lisp\" | \"language\" | \"c\" | \"book\" | \"build\"; \
    phrase    : <adjective> <noun>;                                     \
    doge      : <phrase>*;                                              \
  ",
  Adjective, Noun, Phrase, Doge);

/* Do some parsing here... */m

mpc_cleanup(4, Adjective, Noun, Phrase, Doge);
```

在真正理解这个长字符串的语法之前，这种形式的语法显然*更清晰*。如果我们学习了所有这些特殊符号的意思，我们就可以眯着眼睛将它们看作一体了。

需要注意的另外一件事是，现在这个过程分为两个步骤。首先我们用 `mpc_new` 创造和命名了一些规则，然后我们用 `mpca_lang` 来定义它们。

`mpca_lang` 的第一个参数是选项标志。对于这个我们只需要用默认的就可以了。第二个参数是一个很长的多行字符串。 这是 *(语法)grammar* 规范。它由一些*重写规则*组成。 每一个规则都有一个名字在它们的左边，一个冒号 `:`，在右边有一个分号 `;` 表示定义结束。

右手边定义这些规则的特殊符号像下面这样工作。

<table class='table'>
  <tr><td>`"ab"`</td><td>字符串 `ab` 是必需的</td></tr>
  <tr><td>`'a'`</td><td>字符 `a` 是必需的</td></tr>
  <tr><td>`'a' 'b'`</td><td>首先 `'a'` 是必需的，然后 `'b'` 是必需的</td></tr>
  <tr><td>`'a' | 'b'`</td><td>`a` 或者 `b` 是必需的</td></tr>
  <tr><td>`'a'*`</td><td>零个或者更多 `'a'` 是必需的</td></tr>
  <tr><td>`'a'+`</td><td>一个或者更多 `'a'` 是必需的</td></tr>
  <tr><td>`<abba>`</td><td>叫做 `abba` 的规则是必需的</td></tr>
</table>

<div class="alert alert-warning">
  **听起来有些熟悉...**

  你有注意到这些作为 `mpca_lang` 输入字符串的描述听起来像我正在指定一个语法么？ 这是因为它就是! `mpc` 使用它自己内部解析这些给 `mpca_lang` 的输入。为了做到这一点，它通过用之前的函数来指定一个语法。多么的干净利落啊。
</div>

用上面描述的确认我上面写的和我们之前用代码指定的是等价的。

这种指定语法的方法是我们下一章将要用到的。它一开始看起来可能铺天盖地的。语法看起来不能马上被理解。但是随着我们的继续你会对如何创建和编辑它们越来越熟悉。随意创造一些符号或者符号的概念，让它们写起来更加简单。一些附加题

这一章是关于理论的，所以如果你要尝试附加题，不用太担心正确性。用正确的心态思考更加重要。一些附加题可能需要循环和递归的语法结构，所以如果你用到这些不要担心。


参考
---------

<div class="panel-group alert alert-warning" id="accordion">

  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseOne">
          doge_code.c
        </a>
      </h4>
    </div>
    <div id="collapseOne" class="panel-collapse collapse">
      <div class="panel-body">
```c
#include "mpc.h"

int main(int argc, char** argv) {

  /* Build a new parser 'Adjective' to recognize descriptions */
  mpc_parser_t* Adjective = mpc_or(4,
    mpc_sym("wow"), mpc_sym("many"),
    mpc_sym("so"),  mpc_sym("such")
  );

  /* Build a new parser 'Noun' to recognize things */
  mpc_parser_t* Noun = mpc_or(5,
    mpc_sym("lisp"), mpc_sym("language"),
    mpc_sym("c"),    mpc_sym("book"),
    mpc_sym("build")
  );

  mpc_parser_t* Phrase = mpc_and(2, mpcf_strfold, Adjective, Noun, free);

  mpc_parser_t* Doge = mpc_many(mpcf_strfold, Phrase);

  /* Do some parsing here... */

  mpc_delete(Doge);

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
          doge_grammar.c
        </a>
      </h4>
    </div>
    <div id="collapseTwo" class="panel-collapse collapse">
      <div class="panel-body">
```c
#include "mpc.h"

int main(int argc, char** argv) {

  mpc_parser_t* Adjective = mpc_new("adjective");
  mpc_parser_t* Noun      = mpc_new("noun");
  mpc_parser_t* Phrase    = mpc_new("phrase");
  mpc_parser_t* Doge      = mpc_new("doge");

  mpca_lang(MPC_LANG_DEFAULT,
    "                                                                     \
      adjective : \"wow\" | \"many\" | \"so\" | \"such\";                 \
      noun      : \"lisp\" | \"language\" | \"c\" | \"book\" | \"build\"; \
      phrase    : <adjective> <noun>;                                     \
      doge      : <phrase>*;                                              \
    ",
    Adjective, Noun, Phrase, Doge);

  /* Do some parsing here... */

  mpc_cleanup(4, Adjective, Noun, Phrase, Doge);

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
    <li class="list-group-item">&rsaquo; 尝试写出更多 `Doge` 语言的字符串</li>
    <li class="list-group-item">&rsaquo; 为什么语法中在引号 `"` 的前面会有反斜线 `\` ?</li>
	<li class="list-group-item">&rsaquo; 为什么语法中在行的结尾有反斜线 `\` ?</li>
    <li class="list-group-item">&rsaquo; 用文字描述十进制例如 `0.01` 或者 `52.221` 的语法</li>
    <li class="list-group-item">&rsaquo; 用文字描述网址例如 `http://www.buildyourownlisp.com` 的语法</li>
    <li class="list-group-item">&rsaquo; 用文字描述简单的英语句子例如 `the cat sat on the mat` 的语法</li>
    <li class="list-group-item">&rsaquo; 用更正式的方法描述上面的语法。用 `|`, `*` 或者任何你发明的符号</li>
    <li class="list-group-item">&rsaquo; 如果你熟悉 JSON，用文字来描述它的语法。</li>
  </ul>
</div>
