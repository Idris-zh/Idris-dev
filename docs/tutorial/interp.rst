.. _sect-interp:

.. Example: The Well-Typed Interpreter

********************
例子：良类型的解释器
********************

.. In this section, we’ll use the features we’ve seen so far to write a
.. larger example, an interpreter for a simple functional programming
.. language, with variables, function application, binary operators and
.. an ``if...then...else`` construct. We will use the dependent type
.. system to ensure that any programs which can be represented are
.. well-typed.

在本节中，我们使用我们所见过的功能来做一个大一点的例子——一个简单的函数式语言解释器，
带有变量、函数运用、二元运算符和 ``if...then...else`` 构造。我们用依赖类型系统来
保证所有能可以被表达的程序都是良类型的。

.. Representing Languages

表达一种语言
============

.. First, let us define the types in the language. We have integers,
.. booleans, and functions, represented by ``Ty``:

首先，我们来定义语言中的类型。我们有整数、布尔和函数，用 ``Ty`` 来表示。

.. code-block:: idris

    data Ty = TyInt | TyBool | TyFun Ty Ty

.. We can write a function to translate these representations to a concrete
.. Idris type — remember that types are first class, so can be
.. calculated just like any other value:

我们可以写一个函数来把上面的表述方法转换成一个 Idris 类型——要注意类型是一等的，所以可以像
其他的值一样参与计算。

.. code-block:: idris

    interpTy : Ty -> Type
    interpTy TyInt       = Integer
    interpTy TyBool      = Bool
    interpTy (TyFun A T) = interpTy A -> interpTy T

.. We're going to define a representation of our language in such a way
.. that only well-typed programs can be represented. We'll index the
.. representations of expressions by their type, **and** the types of
.. local variables (the context). The context can be represented using
.. the ``Vect`` data type, and as it will be used regularly it will be
.. represented as an implicit argument. To do so we define everything in
.. a ``using`` block (keep in mind that everything after this point needs
.. to be indented so as to be inside the ``using`` block):

我们将定义一种表达方法，这种方法将只能表达良类型的程序。
我们用表达式的类型和其局部变量的类型（上下文）来索引一个表达式。这样的上下文可以用
``Vect`` 数据类型来表达。因为上下文需要被经常使用，所以这将会成为一个隐式参数。
为了达成这个目的，我们需要使用一个 ``using`` 块来定义所有需要的东西。（注意下文的
代码需要被缩进才能在 ``using`` 块中使用）

.. code-block:: idris

    using (G:Vect n Ty)

.. Expressions are indexed by the types of the local variables, and the type of
.. the expression itself:

表达式用局部变量的类型和表达式自身类型来索引：

.. code-block:: idris

    data Expr : Vect n Ty -> Ty -> Type

.. The full representation of expressions is:

表达式的完全表达方法是：

.. code-block:: idris

    data HasType : (i : Fin n) -> Vect n Ty -> Ty -> Type where
        Stop : HasType FZ (t :: G) t
        Pop  : HasType k G t -> HasType (FS k) (u :: G) t

    data Expr : Vect n Ty -> Ty -> Type where
        Var : HasType i G t -> Expr G t
        Val : (x : Integer) -> Expr G TyInt
        Lam : Expr (a :: G) t -> Expr G (TyFun a t)
        App : Expr G (TyFun a t) -> Expr G a -> Expr G t
        Op  : (interpTy a -> interpTy b -> interpTy c) ->
              Expr G a -> Expr G b -> Expr G c
        If  : Expr G TyBool ->
              Lazy (Expr G a) ->
              Lazy (Expr G a) -> Expr G a

.. The code above makes use of the ``Vect`` and ``Fin`` types from the
.. Idris standard library. We import them because they are not provided
.. in the prelude:

上述代码使用了 Idris 标准库 ``Vect`` 和 ``Fin`` 类型。因为它们不在 prelude 库中，
所以我么需要导入它们。

.. code-block:: idris

    import Data.Vect
    import Data.Fin

.. Since expressions are indexed by their type, we can read the typing
.. rules of the language from the definitions of the constructors. Let us
.. look at each constructor in turn.

由于表达式由用其自身的类型来索引，我们可以直接从其构造器的定义中得出这个语言的类型规则。
下面我们来逐一观察每个构造器。

.. We use a nameless representation for variables — they are *de Bruijn
.. indexed*. Variables are represented by a proof of their membership in
.. the context, ``HasType i G T``, which is a proof that variable ``i``
.. in context ``G`` has type ``T``. This is defined as follows:

我们使用一个不带名字的变量表示方法 - 它们是以 **德布鲁因** 法来索引。变量以其在上下文中
从属关系的证明来表达： ``HasType i G T`` 是变量 ``i`` 在上下文 ``G`` 中有类型
``T`` 的证明。这是以如下形式定义的。

.. code-block:: idris

    data HasType : (i : Fin n) -> Vect n Ty -> Ty -> Type where
        Stop : HasType FZ (t :: G) t
        Pop  : HasType k G t -> HasType (FS k) (u :: G) t

.. We can treat *Stop* as a proof that the most recently defined variable
.. is well-typed, and *Pop n* as a proof that, if the ``n``\ th most
.. recently defined variable is well-typed, so is the ``n+1``\ th. In
.. practice, this means we use ``Stop`` to refer to the most recently
.. defined variable, ``Pop Stop`` to refer to the next, and so on, via
.. the ``Var`` constructor:

我们把 **Stop** 当做最近一个定义的变量有良类型的证明，而 **Pop n** 则证明了
如果第 ``n`` 个最近定义的变量有良类型，那么第 ``n+1`` 个也是。在实际中，这意味着
我们通过 ``Var`` 构造器，使用 ``Stop`` 来引用最近定义的变量，
``Pop Stop`` 来引用下一个，以此类推。

.. code-block:: idris

    Var : HasType i G t -> Expr G t

.. So, in an expression ``\x,\y. x y``, the variable ``x`` would have a
.. de Bruijn index of 1, represented as ``Pop Stop``, and ``y 0``,
.. represented as ``Stop``. We find these by counting the number of
.. lambdas between the definition and the use.

**FZ: Is it a typo? Shouldn't it be** ``\x. \y. x y`` **? This is inconsistent
to Testing section.**

所以，在表达式 ``\x,\y. x y`` 中，变量 ``x`` 有德布鲁因索引 1，可以表达为
``Pop Stop``；变量 ``y`` 有德布鲁因索引 0，可以表达为 ``Stop``。我们通过数定义与使用之间
lambda 的个数来决定德布鲁因索引。

.. A value carries a concrete representation of an integer:

一个值可以表示一个实际的整数。

.. code-block:: idris

    Val : (x : Integer) -> Expr G TyInt

.. A lambda creates a function. In the scope of a function of type ``a ->
.. t``, there is a new local variable of type ``a``, which is expressed
.. by the context index:

一个 lambda 创建一个函数。在这个类型是 ``a -> t`` 的函数的作用域内，有一个类型为
``a`` 的局部变量，用上下文索引来表示。

.. code-block:: idris

    Lam : Expr (a :: G) t -> Expr G (TyFun a t)

.. Function application produces a value of type ``t`` given a function
.. from ``a`` to ``t`` and a value of type ``a``:

函数运用取一个从 ``a`` 到 ``t`` 的函数和一个类型为 ``a`` 的值，创造一个类型为 ``t``
的值。

.. code-block:: idris

    App : Expr G (TyFun a t) -> Expr G a -> Expr G t

.. We allow arbitrary binary operators, where the type of the operator
.. informs what the types of the arguments must be:

我们允许任意的二元运算符，运算符的类型告诉我们其参数的类型：

.. code-block:: idris

    Op : (interpTy a -> interpTy b -> interpTy c) ->
         Expr G a -> Expr G b -> Expr G c

.. Finally, ``If`` expressions make a choice given a boolean. Each branch
.. must have the same type, and we will evaluate the branches lazily so
.. that only the branch which is taken need be evaluated:

最后，``If`` 表达式表示一个取决于给定布尔值的选择。每个分支必须有相同的类型。我们将对于分支
使用惰性求值。只有被选择的分支才会被求值。

.. code-block:: idris

    If : Expr G TyBool ->
         Lazy (Expr G a) ->
         Lazy (Expr G a) ->
         Expr G a

.. Writing the Interpreter

编写解释器
==========

.. When we evaluate an ``Expr``, we'll need to know the values in scope,
.. as well as their types. ``Env`` is an environment, indexed over the
.. types in scope. Since an environment is just another form of list,
.. albeit with a strongly specified connection to the vector of local
.. variable types, we use the usual ``::`` and ``Nil`` constructors so
.. that we can use the usual list syntax. Given a proof that a variable
.. is defined in the context, we can then produce a value from the
.. environment:

当我们求一个 ``Expr`` 求值时，我们需要它作用域内的值和其值的类型。 ``Env`` 是由其
作用域来索引的一个环境。尽管它和局部变量的向量有着很强联系，环境只是列表的另一个形式。
我们用惯用的 ``::`` 和 ``Nil`` 构造器，这样我们可以使用惯用的列表语法。给定一个变量
在上下文中定义的证明，我们可以从环境中得到一个值：

.. code-block:: idris

    data Env : Vect n Ty -> Type where
        Nil  : Env Nil
        (::) : interpTy a -> Env G -> Env (a :: G)

    lookup : HasType i G t -> Env G -> interpTy t
    lookup Stop    (x :: xs) = x
    lookup (Pop k) (x :: xs) = lookup k xs

.. Given this, an interpreter is a function which
.. translates an ``Expr`` into a concrete Idris value with respect to a
.. specific environment:

有了上述内容，一个解释器就是一个把 ``Expr`` 根据特定的环境转换成一个实际的 Idris 值的函数了。

.. code-block:: idris

    interp : Env G -> Expr G t -> interpTy t

.. The complete interpreter is defined as follows, for reference. For
.. each constructor, we translate it into the corresponding Idris value:

作为范例，完整的解释器如下定义。对于每一个构造器，我们把它转换成对应的 Idris 值。

.. code-block:: idris

    interp env (Var i)     = lookup i env
    interp env (Val x)     = x
    interp env (Lam sc)    = \x => interp (x :: env) sc
    interp env (App f s)   = interp env f (interp env s)
    interp env (Op op x y) = op (interp env x) (interp env y)
    interp env (If x t e)  = if interp env x then interp env t
                                             else interp env e

.. Let us look at each case in turn.  To translate a variable, we simply look it
.. up in the environment:

我们来逐一观察每一种情况。对于一个变量，我们只要从环境中找出它。

.. code-block:: idris

    interp env (Var i) = lookup i env

.. To translate a value, we just return the concrete representation of the
.. value:

对于一个值，我们只需要返回其实际的表达方式。

.. code-block:: idris

    interp env (Val x) = x

.. Lambdas are more interesting. In this case, we construct a function
.. which interprets the scope of the lambda with a new value in the
.. environment. So, a function in the object language is translated to an
.. Idris function:

Lambda 则比较有意思。我们创建一个解释 lambda 内部作用域的函数，但其环境中带有一个新的值。
因此，一个目标语言的函数可以用如下的方法来转换成 Idris 函数：

.. code-block:: idris

    interp env (Lam sc) = \x => interp (x :: env) sc

.. For an application, we interpret the function and its argument and apply
.. it directly. We know that interpreting ``f`` must produce a function,
.. because of its type:

对于函数应用，我们解释函数及其参数，并直接地应用函数于参数之上。我们知道，根据其类型，
解释 ``f`` 必定会得到一个函数。

.. code-block:: idris

    interp env (App f s) = interp env f (interp env s)

.. Operators and conditionals are, again, direct translations into the
.. equivalent Idris constructs. For operators, we apply the function to
.. its operands directly, and for ``If``, we apply the Idris
.. ``if...then...else`` construct directly.

运算符和条件分支亦是转换成等价的 Idris 构造。对于运算符，我们将函数直接应用于其（operand）
之上；对于 ``If``，我们直接使用 Idris 的 ``if...then...else`` 构造。

.. code-block:: idris

    interp env (Op op x y) = op (interp env x) (interp env y)
    interp env (If x t e)  = if interp env x then interp env t
                                             else interp env e

.. Testing

测试
====

.. We can make some simple test functions. Firstly, adding two inputs
.. ``\x. \y. y + x`` is written as follows:

我们可以编写一个简单的测试函数。首先，将两个输入值加起来，``\x. \y. y + x`` 可以被
写成如下形式：

.. code-block:: idris

    add : Expr G (TyFun TyInt (TyFun TyInt TyInt))
    add = Lam (Lam (Op (+) (Var Stop) (Var (Pop Stop))))

.. More interestingly, a factorial function ``fact``
.. (e.g. ``\x. if (x == 0) then 1 else (fact (x-1) * x)``),
.. can be written as:

更有趣的是阶乘函数 ``fact``，如 ``\x. if (x == 0) then 1 else (fact (x-1) * x)``，
可以被写成如下形式：

.. code-block:: idris

    fact : Expr G (TyFun TyInt TyInt)
    fact = Lam (If (Op (==) (Var Stop) (Val 0))
                   (Val 1)
                   (Op (*) (App fact (Op (-) (Var Stop) (Val 1)))
                           (Var Stop)))

.. Running

运行
=======

.. To finish, we write a ``main`` program which interprets the factorial
.. function on user input:

作为结束，我们写一个 ``main`` 程序来根据用户的输入来解释阶乘函数：

.. code-block:: idris

    main : IO ()
    main = do putStr "Enter a number: "
              x <- getLine
              printLn (interp [] fact (cast x))

.. Here, ``cast`` is an overloaded function which converts a value from
.. one type to another if possible. Here, it converts a string to an
.. integer, giving 0 if the input is invalid. An example run of this
.. program at the Idris interactive environment is:

此处，``cast`` 是一个被重载的函数，用于在可能的情况下，将一个值转成另外一个类型。在这里，
它将一个字符串转换成一个整数，在输入不合法时返回 0 。用交互式 Idris 环境运行这个程序可以
有以下结果：

.. _factrun:
.. literalinclude:: ../listing/idris-prompt-interp.txt


.. Aside: ``cast``

额外：``cast``
--------------

.. The prelude defines an interface ``Cast`` which allows conversion
.. between types:

Prelude 中定义了一个 ``Cast`` 接口来实现类型之间的转换：

.. code-block:: idris

    interface Cast from to where
        cast : from -> to

.. It is a *multi-parameter* interface, defining the source type and
.. object type of the cast. It must be possible for the type checker to
.. infer *both* parameters at the point where the cast is applied. There
.. are casts defined between all of the primitive types, as far as they
.. make sense.

这是一个 **多参数** 接口，定义了转换的源类型和目标类型。在转换被应用的地方，类型检查器必须
能够推导出 **两个** 参数。原语类型之间有意义的转换已经被提前定义好。
