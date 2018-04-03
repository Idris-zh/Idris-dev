********************
常见问题解答（FA♂Q）
********************

.. **************************
.. Frequently Asked Questions
.. **************************

.. 反正也没人看，随便敲巴敲巴得了= =||

Agda 跟 Idris 有啥不一样啊？
============================

.. What are the differences between Agda and Idris?
.. ================================================

.. Like Idris, Agda is a functional language with dependent types, supporting
.. dependent pattern matching. Both can be used for writing programs and proofs.
.. However, Idris has been designed from the start to emphasise general purpose
.. programming rather than theorem proving. As such, it supports interoperability
.. with systems libraries and C programs, and language constructs for
.. domain specific language implementation. It also includes higher level
.. programming constructs such as interfaces (similar to type classes) and do notation.

和 Idris 一样，Agda 也是个带有依赖类型的函数式语言，并且支持依赖模式匹配。
它们都能编写程序和证明。不过，Idris 一开始就侧重于为通用编程而设计，而非 Agda
侧重的定理证明。因此，它支持与系统库和 C 程序的互操作性，以及用于实现 EDSL
（领域特定语言）的语言构造。它还包括更高级的编程构造，例如接口（类似于类型类）和
``do`` 记法。

.. Idris supports multiple back ends (C and JavaScript by default, with the
.. ability to add more via plugins) and has a reference run time system, written
.. in C, with a garbage collector and built-in message passing concurrency.

Idris 支持多种后端（默认为 C 和 JavaScript，可以通过插件添加更多），拥有一个 C
编写的，带有垃圾收集器和内建并发消息传递的参考运行时系统。


Idris 能拿来干活不？
====================

.. Is Idris production ready?
.. ==========================

.. Idris is primarily a research tool for exploring the possibilities of software
.. development with dependent types, meaning that the primary goal is not (yet) to
.. make a system which could be used in production. As such, there are a few rough
.. corners, and lots of missing libraries. Nobody is working on Idris full time,
.. and we don't have the resources at the moment to polish the system on our own.
.. Therefore, we don't recommend building your business around it!

Idris 主要是个研究工具，用来探索依赖类型软件开发的可能性，也就是说它的主要目的（还）
不是开发可用于生产的系统。因此，某些方面上它还有些粗糙，也缺少很多库。现在还没人全职搞
Idris，我们目前也没有资源来独立完善该系统。因此，我们不推荐你围绕它来开展业务！

.. Having said that, contributions which help towards making Idris suitable
.. for use in production would be very welcome - this includes (but is not
.. limited to) extra library support, polishing the run-time system (and ensuring
.. it is robust), providing and maintaining a JVM back end, etc.

话虽如此，我们非常欢迎帮助推进 Idris 适应生产的贡献，包括（但不限于）额外的库支持、
完善运行时系统（并确保其健壮性）、提供并维护 JVM 后端等等。


为啥 Idris 用了及早求值，不用惰性求值啊？
=========================================

.. Why does Idris use eager evaluation rather than lazy?
.. =====================================================

.. Idris uses eager evaluation for more predictable performance, in particular
.. because one of the longer term goals is to be able to write efficient and
.. verified low level code such as device drivers and network infrastructure.
.. Furthermore, the Idris type system allows us to state precisely the type
.. of each value, and therefore the run-time form of each value. In a lazy
.. language, consider a value of type ``Int``:

Idris 采用及早求值主要是为了能更好地预测性能，特别是其长期目标之一就是能够编写高效
且经过验证的底层代码，例如设备驱动以及网络设施。此外，Idris 的类型系统能让我们精确地
指明每个值的类型，因此也就能确定每个值在运行时的形式。在惰性求值的语言中，考虑一个
``Int`` 类型的值：

.. code-block:: idris

    thing : Int

.. What is the representation of ``thing`` at run-time? Is it a bit pattern
.. representing an integer, or is it a pointer to some code which will compute
.. an integer? In Idris, we have decided that we would like to make this
.. distinction precise, in the type:

``thing`` 在运行时是如何表示的？它是一个按照位模式来表示的整数，
还一个是指向了能计算出整数的代码的指针？在 Idris 中，我们决定按照类型来精确地区分它们：

.. code-block:: idris

    thing_val : Int
    thing_comp : Lazy Int

.. Here, it is clear from the type that ``thing_val`` is guaranteed to be a
.. concrete ``Int``, whereas ``thing_comp`` is a computation which will produce an
.. ``Int``.

这样，``thing_val`` 就能确保为具体的 ``Int``，而 ``thing_comp`` 则是会产生 ``Int``
的计算。


俺咋弄个惰性控制结构呐？
========================

.. How can I make lazy control structures?
.. =======================================

.. You can make control structures using the special Lazy type. For
.. example, ``if...then...else...`` in Idris expands to an application of
.. a function named ``ifThenElse``. The default implementation for
.. Booleans is defined as follows in the library:

你可以用特殊的 ``Lazy`` 类型来创建这种控制结构。例如，Idris 中的
``if...then...else...`` 会被扩展为 ``ifThenElse`` 函数的应用。
它在库中对布尔值的默认实现如下：

.. code-block:: idris

    ifThenElse : Bool -> (t : Lazy a) -> (e : Lazy a) -> a
    ifThenElse True  t e = t
    ifThenElse False t e = e

.. The type ``Lazy a`` for ``t`` and ``e`` indicates that those arguments will
.. only be evaluated if they are used, that is, they are evaluated lazily.

``t`` 和 ``e`` 的类型 ``Lazy a`` 指出它们只会在需要时被求值，也就是说，它们是惰性求值的。


REPL 里头的求值行为跟咱想的不一样啊，这咋回事儿= =？
====================================================

.. Evaluation at the REPL doesn't behave as I expect. What's going on?
.. ===================================================================

.. Being a fully dependently typed language, Idris has two phases where it
.. evaluates things, compile-time and run-time. At compile-time it will only
.. evaluate things which it knows to be total (i.e. terminating and covering all
.. possible inputs) in order to keep type checking decidable. The compile-time
.. evaluator is part of the Idris kernel, and is implemented in Haskell using a
.. HOAS (higher order abstract syntax) style representation of values. Since
.. everything is known to have a normal form here, the evaluation strategy doesn't
.. actually matter because either way it will get the same answer, and in practice
.. it will do whatever the Haskell run-time system chooses to do.

作为一个完全依赖类型的语言，Idris 的求值分为两个阶段，编译时与运行时。在编译时，
它只会对已知完全（即保证会终止且覆盖了所有可能输入）的东西求值，
以此来保持类型检查的可判定性。编译时的求值器属于 Idris 核心的一部分，它用 Haskell
编写，按照值的 HOAS（Higher Order Abstract Syntax 高阶抽象语法）风格的表示来实现。
由于已知所有东西都有规范的形式，选用哪种求值策略实际上也就无关紧要了，
因为它们都会得到相同的答案，而在实践中，它会执行 Haskell 运行时系统选择的任何事情。

.. The REPL, for convenience, uses the compile-time notion of evaluation. As well
.. as being easier to implement (because we have the evaluator available) this can
.. be very useful to show how terms evaluate in the type checker. So you can see
.. the difference between:

按照约定，REPL 使用编译时求值的概念。除了更容易实现外（因为我们可以用求值器），
它在展示项（Term）如何在类型检查器中求值时也非常有用。

.. code-block:: idris

    Idris> \n, m => (S n) + m
    \n => \m => S (plus n m) : Nat -> Nat -> Nat

    Idris> \n, m => n + (S m)
    \n => \m => plus n (S m) : Nat -> Nat -> Nat


为啥咱不能在类型里用没参数的函数呐？
====================================

.. Why can't I use a function with no arguments in a type?
.. =======================================================

.. If you use a name in a type which begins with a lower case letter, and which is
.. not applied to any arguments, then Idris will treat it as an implicitly
.. bound argument. For example:

如果你在类型中使用小写字母开头的名字，且它们没用被应用于任何参数，那么 Idris
会把它视为隐式绑定的参数。例如：

.. code-block:: idris

    append : Vect n ty -> Vect m ty -> Vect (n + m) ty

.. Here, ``n``, ``m``, and ``ty`` are implicitly bound. This rule applies even
.. if there are functions defined elsewhere with any of these names. For example,
.. you may also have:

这里的 ``n``、``m`` 和 ``ty`` 是隐式绑定的。此规则同样适用于在别处定义的，
带这类名字的函数。例如，你或许还有：

.. code-block:: idris

    ty : Type
    ty = String

.. Even in this case, ``ty`` is still considered implicitly bound in the definition
.. of ``append``, rather than making the type of ``append`` equivalent to...

即便如此，``ty`` 还是会被视作 ``append`` 定义中隐式绑定的参数，而不会让 ``append``
的类型等价于你预想的：

.. code-block:: idris

    append : Vect n String -> Vect m String -> Vect (n + m) String

.. ...which is probably not what was intended!  The reason for this rule is so
.. that it is clear just from looking at the type of ``append``, and no other
.. context, what the implicitly bound names are.

采用了此规则后，你无需其它上下文，只要查看 ``append`` 的类型就能明白哪些是\
隐式绑定的名字。

.. If you want to use an unapplied name in a type, you have two options. You
.. can either explicitly qualify it, for example, if ``ty`` is defined in the
.. namespace ``Main`` you can do the following:

如果你想在类型中使用不被应用的名字，那么有两种选择：你可以显式地限定它，例如，
若 ``ty`` 在 ``Main`` 的命名空间中定义，你可以这样做：

.. code-block:: idris

    append : Vect n Main.ty -> Vect m Main.ty -> Vect (n + m) Main.ty

.. Alternatively, you can use a name which does not begin with a lower case
.. letter, which will never be implicitly bound:

此外，你还可以使用不以小写字母开头的名字，它决不会被隐式绑定：

.. code-block:: idris

    Ty : Type
    Ty = String

    append : Vect n Ty -> Vect m Ty -> Vect (n + m) Ty

.. As a convention, if a name is intended to be used as a type synonym, it is
.. best for it to begin with a capital letter to avoid this restriction.

按照约定，如果你打算将一个名字用作类型同义，那么最好以大写字母开头来避免此限制。


这破程序明显能停，为啥 Idris 还说它有可能不完全？
=================================================

.. I have an obviously terminating program, but Idris says it possibly isn't total. Why is that?
.. =============================================================================================

.. Idris can't decide in general whether a program is terminating due to
.. the undecidability of the `Halting Problem
.. <https://en.wikipedia.org/wiki/Halting_problem>`_. It is possible, however,
.. to identify some programs which are definitely terminating. Idris does this
.. using "size change termination" which looks for recursive paths from a
.. function back to itself. On such a path, there must be at least one
.. argument which converges to a base case.

由于\ `停机问题 <https://en.wikipedia.org/wiki/Halting_problem>`_\ 的不可判定性，
Idris 通常无法判定一个程序是否会停止。然而，我们可以找出某些确定可以终止的程序。
Idris 使用「大小改变终止（size change termination）」来寻找从函数返回到自身的递归路径。
在此路径上，至少必有一个参数会收敛到基本情况。

.. - Mutually recursive functions are supported
.. - However, all functions on the path must be fully applied. In particular,
..   higher order applications are not supported
.. - Idris identifies arguments which converge to a base case by looking for
..   recursive calls to syntactically smaller arguments of inputs. e.g.
..   ``k`` is syntactically smaller than ``S (S k)`` because ``k`` is a
..   subterm of ``S (S k)``, but ``(k, k)`` is
..   not syntactically smaller than ``(S k, S k)``.

- Idris 支持相互递归的函数
- 不过，递归路径上的所有函数必须被完整地应用。此外，Idris 不支持高阶应用。
- 在递归调用的过程中，Idris 会查找语法上更小的输入参数，以此来识别能收敛到基本情况的参数。
  例如，``k`` 在语法上小于 ``S (S k)``，因为 ``k`` 是 ``S (S k)`` 的子项（subterm），
  然而 ``(k, k)`` 在语法上却不小于 ``(S k, S k)``。

.. If you have a function which you believe to be terminating, but Idris does
.. not, you can either restructure the program, or use the ``assert_total``
.. function.

如果你有个确信会终止的函数，但 Idris 不信，那么你可以调整程序的结构，或者使用
``assert_total`` 函数。


Idris 啥时候能自举啊？
======================

.. When will Idris be self-hosting?
.. ================================

.. It’s not a priority, though not a bad idea in the long run. It would be a
.. worthwhile effort in the short term to implement libraries to support
.. self-hosting, such as a good parsing library.

这事不急，虽说从长远来看这主意不错。就目前来说，实现支持自举的库是一项很有价值的工作,
比如说实现良好的解析库。


Idris 有全域多态不? ``Type`` 是啥类型的？
=========================================

.. Does Idris have universe polymorphism? What is the type of ``Type``?
.. ====================================================================

.. Rather than universe polymorphism, Idris has a cumulative hierarchy of
.. universes; ``Type : Type 1``, ``Type 1 : Type 2``, etc.
.. Cumulativity means that if ``x : Type n`` and ``n <= m``, then
.. ``x : Type m``. Universe levels are always inferred by Idris, and
.. cannot be specified explicitly. The REPL command ``:type Type 1`` will
.. result in an error, as will attempting to specify the universe level
.. of any type.

Idris 并没有全域多态（Universe Polymorphism），而是拥有全域的积累层级
（Cumulative Hierarchy），如 ``Type : Type 1``、``Type 1 : Type 2`` 等等。
积累性的意思是，若 ``x : Type n`` 且 ``n <= m``，则 ``x : Type m``。
全域的级别总是由 Idris 推导，且无法被显式地指定。执行 REPL 命令 ``:type Type 1``
以及试图为任何类型指定全域级别时，都会产生一个错误。


为啥 Idris 用 ``Double`` 不用 ``Float64``？
===========================================

.. Why does Idris use ``Double`` instead of ``Float64``?
.. =====================================================

.. Historically the C language and many other languages have used the
.. names ``Float`` and ``Double`` to represent floating point numbers of
.. size 32 and 64 respectively.  Newer languages such as Rust and Julia
.. have begun to follow the naming scheme described in `IEEE Standard for
.. Floating-Point Arithmetic (IEEE 754)
.. <https://en.wikipedia.org/wiki/IEEE_floating_point>`_. This describes
.. single and double precision numbers as ``Float32`` and ``Float64``;
.. the size is described in the type name.

历史上 C 和很多语言都用分别用 ``Float`` 和 ``Double`` 来表示 32 位和
64 位的浮点数。较新的语言，如 Rust 和 Julia 都开始遵循 `IEEE 浮点运算标准
(IEEE 754) <https://en.wikipedia.org/wiki/IEEE_floating_point>`_ 的命名规范了。
它将单精度和双精度的数描述为 ``Float32`` 和 ``Float64``，其大小在类型名中描述。

.. Due to developer familiarity with the older naming convention, and
.. choice by the developers of Idris, Idris uses the C style convention.
.. That is, the name ``Double`` is used to describe double precision
.. numbers, and Idris does not support 32 bit floats at present.

由于开发者更熟悉旧有的命名约定，而 Idris 的开发者也选择了它，因此 Idris 采用了
C 风格的约定。也就是说名称 ``Double`` 用于描述双精度浮点数，而 Idris 现在还不支持
32 位浮点数。

``-ffreestanding`` 是啥？
=========================

.. What is -ffreestanding?
.. =======================

.. The freestanding flag is used to build Idris binaries which have their
.. libs and compiler in a relative path. This is useful for building binaries
.. where the install directory is unknown at build time. When passing this
.. flag, the IDRIS_LIB_DIR environment variable needs to be set to the path
.. where the Idris libs reside relative to the idris executable. The
.. IDRIS_TOOLCHAIN_DIR environment variable is optional, if that is set,
.. Idris will use that path to find the C compiler.

在相对路径中拥有自己的库和编译器时，可使用 ``freestanding`` 命令行参数来构建
Idris 二进制文件。当构建过程中的安装目录未知时，它对于构建二进制文件来说非常有用。
当传入此参数时，``IDRIS_LIB_DIR`` 环境变量需要设置为相对与 ``idris`` 可执行文件所在的
Idris 库的路径。 ``IDRIS_TOOLCHAIN_DIR`` 环境变量是可选的，如果设置了它，Idris
就会在该路径下寻找 C 编译器。

.. .. Example::

   .. IDRIS_LIB_DIR="./libs" IDRIS_TOOLCHAIN_DIR="./mingw/bin" CABALFLAGS="-fffi -ffreestanding -frelease" make

例如：

::

   IDRIS_LIB_DIR="./libs" \
   IDRIS_TOOLCHAIN_DIR="./mingw/bin" \
   CABALFLAGS="-fffi -ffreestanding -frelease" \
   make


话说「Idris」是个啥名儿 O_O？
=============================

.. What does the name ‘Idris’ mean?
.. ================================

.. British people of a certain age may be familiar with this
.. `singing dragon <https://www.youtube.com/watch?v=G5ZMNyscPcg>`_. If
.. that doesn’t help, maybe you can invent a suitable acronym :-) .

有一定年龄的英国人可能很熟悉\ `唱歌贼好听的小火龙妹砸 <https://www.youtube.com/watch?v=G5ZMNyscPcg>`_。
你要是不满意，自己想个好词儿啊（手动微笑 :-)


还能有 Unicode 操作符不？
=========================

.. Will there be support for Unicode characters for operators?
.. ===========================================================

.. There are several reasons why we should not support Unicode operators:

.. - It's hard to type (this is important if you're using someone else's code, for
..   example). Various editors have their own input methods, but you have to know
..   what they are.
.. - Not every piece of software easily supports it. Rendering issues have been
..   noted on some mobile email clients, terminal-based IRC clients, web browsers,
..   etc. There are ways to resolve these rendering issues but they provide a
..   barrier to entry to using Idris.
.. - Even if we leave it out of the standard library (which we will in any case!)
..   as soon as people start using it in their library code, others have to deal
..   with it.
.. - Too many characters look too similar. We had enough trouble with confusion
..   between 0 and O without worrying about all the different kinds of colons and
..   brackets.
.. - There seems to be a tendency to go over the top with use of Unicode. For
..   example, using sharp and flat for delay and force (or is it the other way
..   around?) in Agda seems gratuitous. We don't want to encourage this sort of
..   thing, when words are often better.

下面是为什么我们不应该支持 Unicode 操作符的原因：

 - 它难以输入（如果你在使用别人的代码，这点就很重要）。很多编辑器都有它自己的输入法，
   不过你必须知道怎么输入。
 - 并不是任何软件都能轻松支持它。在一些移动 Email 客户端、基于终端的 IRC 客户端、
   以及 Web 浏览器等软件中都会出现渲染问题。
 - 即便我们不在标准库中使用它（绝对不会！），然而只要有人在他们的库代码中用了它，
   别人就得去处理它。
 - 有太多字符看起来太像了。单是分不清 0 和 O 就会造成很多麻烦，更不说各式各样的冒号和括号了。

.. 他们肯定不认识一只叫 O0 的中国猫= =||

 - Unicode 似乎有被滥用的趋势。比如说，Agda 用升调♯和降调♭符号来表示推迟和强制求值
   （还是啥来着？）这看起来很没道理嘛。当用单词更好时，我们不想鼓励这种事情。

.. With care, Unicode operators can make things look pretty but so can ``lhs2TeX``.
.. Perhaps in a few years time things will be different and software will cope
.. better and it will make sense to revisit this. For now, however, Idris will not
.. be offering arbitrary Unicode symbols in operators.

如果使用得当，Unicode 操作符能让代码看起来更漂亮，然而 ``lhs2TeX`` 也能。
也许几年后情况有变，软件能更好地应对它，到时候重新审视它才有意义。然而目前，
Idris 不会为操作符提供任何 Unicode 符号。

.. This seems like an instance of `Wadler's
.. Law <http://www.haskell.org/haskellwiki/Wadler%27s_Law>`__ in action.

这似乎是个 `Wadler 定律 <http://www.haskell.org/haskellwiki/Wadler%27s_Law>`_
在工作中的实例。

.. 鸡毛蒜皮定律（Law of triviality）了解一下？

.. This answer is based on Edwin Brady's response in the following
.. `pull request <https://github.com/idris-lang/Idris-dev/pull/694#issuecomment-29559291>`__.

本答案基于 Edwin Brady 对此
`推送请求 <https://github.com/idris-lang/Idris-dev/pull/694#issuecomment-29559291>`_
的回应。

Idris 有社区准则不？
====================

.. Where can I find the community standards for the Idris community?
.. ==================================================================

.. The Idris Community Standards are stated `here
.. <https://www.idris-lang.org/documentation/community-standards/>`_ .

`这里 <https://www.idris-lang.org/documentation/community-standards/>`_ 是Idris 社区规范的声明。

还有哪儿能找到解答啊？
======================

.. Where can I find more answers?
.. ==============================

.. There is an `Unofficial FAQ
.. <https://github.com/idris-lang/Idris-dev/wiki/Unofficial-FAQ>`_ on the wiki on
.. GitHub which answers more technical questions and may be updated more often.

Github 的维基上还有个\ `非官方 FAQ
<https://github.com/Idris-zh/Idris-dev/wiki/Unofficial-FAQ>`_，
其中解答了更多技术问题，而且经常更新。
