.. _sect-syntax:

********
语法扩展
********

.. *****************
.. Syntax Extensions
.. *****************

.. Idris supports the implementation of *Embedded Domain Specific
.. Languages* (EDSLs) in several ways [1]_. One way, as we have already
.. seen, is through extending ``do`` notation. Another important way is
.. to allow extension of the core syntax. In this section we describe two
.. ways of extending the syntax: ``syntax`` rules and ``dsl`` notation.

Idris 支持以多种方式实现 **嵌入式领域特定语言（Embedded Domain Specific Language, EDSL）** [1]_ 。
我们见过的一种方式是扩展 ``do`` 记法。另一种重要的方式就是对核心语法进行扩展。
在本节中，我们描述了两种扩展语法的方式：``syntax`` 规则与 ``dsl`` 记法。

``syntax`` 规则
===============

.. ``syntax`` rules
.. ================

.. We have seen ``if...then...else`` expressions, but these are not built
.. in. Instead, we can define a function in the prelude as follows (we
.. have already seen this function in Section :ref:`sect-lazy`):

我们已经见过 ``if...then...else`` 表达式了，然而它并不是内建的。同样，
我们可以定义一个 Prelude 中的函数（它在 :ref:`sect-lazy` 一节中出现过了）：

.. code-block:: idris

    ifThenElse : (x:Bool) -> Lazy a -> Lazy a -> a;
    ifThenElse True  t e = t;
    ifThenElse False t e = e;

.. and then extend the core syntax with a ``syntax`` declaration:

接着用 ``syntax`` 声明来扩展核心语法：

.. code-block:: idris

    syntax if [test] then [t] else [e] = ifThenElse test t e;

.. The left hand side of a ``syntax`` declaration describes the syntax
.. rule, and the right hand side describes its expansion. The syntax rule
.. itself consists of:

.. -  **Keywords** — here, ``if``, ``then`` and ``else``, which must be
..    valid identifiers

.. -  **Non-terminals** — included in square brackets, ``[test]``, ``[t]``
..    and ``[e]`` here, which stand for arbitrary expressions. To avoid
..    parsing ambiguities, these expressions cannot use syntax extensions
..    at the top level (though they can be used in parentheses).

.. -  **Names** — included in braces, which stand for names which may be
..    bound on the right hand side.

.. -  **Symbols** — included in quotations marks, e.g. ``:=``. This can
..    also be used to include reserved words in syntax rules, such as
..    ``let`` or ``in``.

``syntax`` 声明的左式描述了语法规则，而右式描述了其展开式。语法规则的构成为：

-  **关键字** — 在这里为 ``if``、``then`` 和 ``else``，它必须是有效的标识符。

-  **非终止符（Non-terminal）** — 位于方括号内，此处为 ``[test]``、``[t]`` 和
   ``[e]``，它们表示任意表达式。为避免解析歧义，这些表达式不能在顶层使用语法扩展
   （也就是说你可以在括号中使用）。

-  **名称** — 位于大括号内，它表示可在右侧被绑定的名字。

-  **符号** — 位于引号内，例如 ``":="``。它也可在语法规则中包含保留字，例如
   ``"let"`` 或 ``"in"``。

.. The limitations on the form of a syntax rule are that it must include
.. at least one symbol or keyword, and there must be no repeated
.. variables standing for non-terminals. Any expression can be used, but
.. if there are two non-terminals in a row in a rule, only simple
.. expressions may be used (that is, variables, constants, or bracketed
.. expressions). Rules can use previously defined rules, but may not be
.. recursive. The following syntax extensions would therefore be valid:

语法规则形式的限制在于它必须包含至少一个符号或关键字，且表示非终止符的变量不能重复。
任何表达式都可以使用，不过如果在一个规则的同一行内有两个非终止符，那么只有简单的表达式会被使用
（即，变量、常量或方括号括起的表达式）。规则可在其定义之前使用，但无法递归地使用。
因此以下语法扩展是有效的：

.. code-block:: idris

    syntax [var] ":=" [val]                = Assign var val;
    syntax [test] "?" [t] ":" [e]          = if test then t else e;
    syntax select [x] from [t] "where" [w] = SelectWhere x t w;
    syntax select [x] from [t]             = Select x t;

.. Syntax macros can be further restricted to apply only in patterns (i.e.,
.. only on the left hand side of a pattern match clause) or only in terms
.. (i.e. everywhere but the left hand side of a pattern match clause) by
.. being marked as ``pattern`` or ``term`` syntax rules. For example, we
.. might define an interval as follows, with a static check that the lower
.. bound is below the upper bound using ``so``:

语法宏可被进一步限制为只能在模式（即，只能在模式匹配从句的左侧）中或只能在被标为
``pattern`` 或 ``term`` 语法规则的项（即除了模式匹配从句左侧的任何地方）中应用。
例如，假设我们定义了一个区间 ``Interval``，用 ``So`` 静态检查以保证下界小于上界：

.. code-block:: idris

    data Interval : Type where
       MkInterval : (lower : Double) -> (upper : Double) ->
                    So (lower < upper) -> Interval

.. We can define a syntax which, in patterns, always matches ``Oh`` for
.. the proof argument, and in terms requires a proof term to be provided:

我们可以用 ``pattern`` 定义一个语法，它总是匹配 ``Oh`` 作为证明论据，用 ``term``
请求提供一个证明项：

.. code-block:: idris

    pattern syntax "[" [x] "..." [y] "]" = MkInterval x y Oh
    term    syntax "[" [x] "..." [y] "]" = MkInterval x y ?bounds_lemma

.. In terms, the syntax ``[x...y]`` will generate a proof obligation
.. ``bounds_lemma`` (possibly renamed).

在 ``term`` 中，语法 ``[x...y]`` 会生成一个证明义务 ``bounds_lemma`` （可能被重命名）。

.. Finally, syntax rules may be used to introduce alternative binding
.. forms. For example, a ``for`` loop binds a variable on each iteration:

最后，语法规则可引入另一种绑定形式。例如，``for`` 循环在每次迭代中绑定一个参数：

.. code-block:: idris

    syntax for {x} "in" [xs] ":" [body] = forLoop xs (\x => body)

    main : IO ()
    main = do for x in [1..10]:
                  putStrLn ("Number " ++ show x)
              putStrLn "Done!"

.. Note that we have used the ``{x}`` form to state that ``x`` represents
.. a bound variable, substituted on the right hand side. We have also put
.. ``in`` in quotation marks since it is already a reserved word.

注意，我们用形式 ``{x}`` 指明 ``x`` 表示一个已绑定的变量，它在右侧会被替换。
我们还把 ``in`` 放在了引号中，因为它是保留字。

``dsl`` 记法
============

.. ``dsl`` notation
.. ================

.. The well-typed interpreter in Section :ref:`sect-interp` is a simple
.. example of a common programming pattern with dependent types. Namely:
.. describe an *object language* and its type system with dependent types
.. to guarantee that only well-typed programs can be represented, then
.. program using that representation. Using this approach we can, for
.. example, write programs for serialising binary data [2]_ or running
.. concurrent processes safely [3]_.

:ref:`sect-interp` 一节中的良类型解释器是个依赖类型编程模式的简单例子。也就是说：
先用依赖类型描述一个 **目标语言** 及其类型系统，保证只有良类型的程序可被表示，
然后再通过这种方式表示程序。通过这种方式，我们可以编写序列化二进制数据 [2]_
或安全运行并发过程 [3]_ 的程序。

.. Unfortunately, the form of object language programs makes it rather
.. hard to program this way in practice. Recall the factorial program in
.. ``Expr`` for example:

然而，目标语言的形式使其难以在实践中编程。回想一下用 ``Expr`` 编写的阶乘程序：

.. code-block:: idris

    fact : Expr G (TyFun TyInt TyInt)
    fact = Lam (If (Op (==) (Var Stop) (Val 0))
                   (Val 1) (Op (*) (App fact (Op (-) (Var Stop) (Val 1)))
                                   (Var Stop)))

.. Since this is a particularly useful pattern, Idris provides syntax
.. overloading [1]_ to make it easier to program in such object
.. languages:

由于这是一种特别有用的模式，因此 Idris 提供了语法重载 [1]_
使其在这种目标语言中更易于编程：

.. code-block:: idris

    mkLam : TTName -> Expr (t::g) t' -> Expr g (TyFun t t')
    mkLam _ body = Lam body

    dsl expr
        variable    = Var
        index_first = Stop
        index_next  = Pop
        lambda      = mkLam

.. A ``dsl`` block describes how each syntactic construct is represented
.. .. in an object language. Here, in the ``expr`` language, any variable is
.. .. translated to the ``Var`` constructor, using ``Pop`` and ``Stop`` to
.. .. construct the de Bruijn index (i.e., to count how many bindings since
.. .. the variable itself was bound); and any lambda is translated to a
.. .. ``Lam`` constructor. The ``mkLam`` function simply ignores its first
.. .. argument, which is the name that the user chose for the variable. It
.. .. is also possible to overload ``let`` and dependent function syntax
.. .. (``pi``) in this way. We can now write ``fact`` as follows:

``dsl`` 块描述了每个语法构造是如何在目标语言中表示的。在这里的 ``expr`` 语言中，
任何变量都会被翻译为 ``Var`` 构造器，使用 ``Pop`` 和 ``Stop`` 来构造 de Bruijn
索引（即，由于变量本身被绑定，所以要统计有多少个绑定）；而任何 λ-表达式都会被翻译为
``Lam`` 构造器。``mkLam`` 函数会简单地忽略其第一个参数，它是用户为变量选择的名字。
我们也可以通过这种方式来重载 ``let`` 与依赖函数的语法。现在可以将 ``fact`` 写成下面这样了：

.. code-block:: idris

    fact : Expr G (TyFun TyInt TyInt)
    fact = expr (\x => If (Op (==) x (Val 0))
                          (Val 1) (Op (*) (app fact (Op (-) x (Val 1))) x))

.. In this new version, ``expr`` declares that the next expression will
.. be overloaded. We can take this further, using idiom brackets, by
.. declaring:

在这个新的版本中，``expr`` 声明了下一个要被重载的表达式。我们可以利用惯用括号，
通过以下声明再进一步：

.. code-block:: idris

    (<*>) : (f : Lazy (Expr G (TyFun a t))) -> Expr G a -> Expr G t
    (<*>) f a = App f a

    pure : Expr G a -> Expr G a
    pure = id

.. Note that there is no need for these to be part of an implementation of
.. ``Applicative``, since idiom bracket notation translates directly to
.. the names ``<*>`` and ``pure``, and ad-hoc type-directed overloading
.. is allowed. We can now say:

注意，它无需成为 ``Applicative`` 实现的一部分，因为惯用括号记法会直接被翻译为
名字 ``<*>`` 和 ``pure``，针对特设（ad-hoc）类型的重载也被允许。现在我们可以写成：

.. code-block:: idris

    fact : Expr G (TyFun TyInt TyInt)
    fact = expr (\x => If (Op (==) x (Val 0))
                          (Val 1) (Op (*) [| fact (Op (-) x (Val 1)) |] x))

.. With some more ad-hoc overloading and use of interfaces, and a new
.. syntax rule, we can even go as far as:

使用更加特设的重载、接口，以及新的语法规则，我们甚至可以再进一步：

.. code-block:: idris

    syntax "IF" [x] "THEN" [t] "ELSE" [e] = If x t e

    fact : Expr G (TyFun TyInt TyInt)
    fact = expr (\x => IF x == 0 THEN 1 ELSE [| fact (x - 1) |] * x)


.. [1] Edwin Brady and Kevin Hammond. 2012. Resource-Safe systems
       programming with embedded domain specific languages. In
       Proceedings of the 14th international conference on Practical
       Aspects of Declarative Languages (PADL'12), Claudio Russo and
       Neng-Fa Zhou (Eds.). Springer-Verlag, Berlin, Heidelberg,
       242-257. DOI=10.1007/978-3-642-27694-1_18
       http://dx.doi.org/10.1007/978-3-642-27694-1_18

.. [2] Edwin C. Brady. 2011. IDRIS ---: systems programming meets full
       dependent types. In Proceedings of the 5th ACM workshop on
       Programming languages meets program verification (PLPV
       '11). ACM, New York, NY, USA,
       43-54. DOI=10.1145/1929529.1929536
       http://doi.acm.org/10.1145/1929529.1929536

.. [3] Edwin Brady and Kevin Hammond. 2010. Correct-by-Construction
       Concurrency: Using Dependent Types to Verify Implementations of
       Effectful Resource Usage Protocols. Fundam. Inf. 102, 2 (April
       2010), 145-176. http://dl.acm.org/citation.cfm?id=1883636
