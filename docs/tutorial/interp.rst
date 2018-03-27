.. _sect-interp:

********************
示例：良类型的解释器
********************

.. Example: The Well-Typed Interpreter

.. In this section, we’ll use the features we’ve seen so far to write a
.. larger example, an interpreter for a simple functional programming
.. language, with variables, function application, binary operators and
.. an ``if...then...else`` construct. We will use the dependent type
.. system to ensure that any programs which can be represented are
.. well-typed.

在本节中，我们使用已经见过的特性来编写一个更大的例子：一个简单的函数式语言解释器，
带有变量、函数应用、二元运算符和 ``if...then...else`` 构造。
我们用依赖类型系统来保证所有能够表示的程序都是良类型的。

语言的表示
==========

.. Representing Languages

.. First, let us define the types in the language. We have integers,
.. booleans, and functions, represented by ``Ty``:

首先，我们来定义语言中的类型。我们有整数、布尔和函数，用 ``Ty`` 来表示：

.. code-block:: idris

    data Ty = TyInt | TyBool | TyFun Ty Ty

.. We can write a function to translate these representations to a concrete
.. Idris type — remember that types are first class, so can be
.. calculated just like any other value:

我们可以写一个函数将上面的表示转换为具体的 Idris 类型——要注意类型是一等的，
所以可以像其它值一样参与计算：

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

我们将定义一种表示方法，它只能表示良类型的程序。我们通过表达式的类型 **与**
其局部变量的类型（上下文）来索引它。上下文可以用 ``Vect`` 数据类型来表达。
由于上下文需要被经常使用，因此它会被表示为隐式参数。为此，我们在 ``using``
块中定义了所有需要的东西。（注意后文中的代码需要缩进才能在 ``using`` 块中使用）：

.. code-block:: idris

    using (G:Vect n Ty)

.. Expressions are indexed by the types of the local variables, and the type of
.. the expression itself:

表达式通过局部变量的类型和表达式自身的类型来索引：

.. code-block:: idris

    data Expr : Vect n Ty -> Ty -> Type

.. The full representation of expressions is:

表达式的完整表示为：

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

上述代码使用了 Idris 标准库中的``Vect`` 和 ``Fin`` 类型。它们不在 Prelude 中，
因此我们需要导入它们：

.. code-block:: idris

    import Data.Vect
    import Data.Fin

.. Since expressions are indexed by their type, we can read the typing
.. rules of the language from the definitions of the constructors. Let us
.. look at each constructor in turn.

由于表达式通过其类型来索引，我们可以直接从构造器的定义中得出该语言的类型规则。
下面我们来逐一观察每个构造器。

.. We use a nameless representation for variables — they are *de Bruijn
.. indexed*. Variables are represented by a proof of their membership in
.. the context, ``HasType i G T``, which is a proof that variable ``i``
.. in context ``G`` has type ``T``. This is defined as follows:

我们为变量使用了不带名字的表示法 - 它们以 **de Bruijn** 法来索引。
变量以它们在上下文中从属关系的证明来表示： ``HasType i G T`` 是变量 ``i``
在上下文 ``G`` 中拥有类型 ``T`` 的证明。它的定义如下：

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

我们把 **Stop** 当做最近定义的变量有良类型的证明，而 **Pop n** 则证明了
如果第 ``n`` 个最近定义的变量是良类型的，那么第 ``n+1`` 个也是。在实践中，
这意味着我们通过 ``Var`` 构造器，使用 ``Stop`` 来引用最近定义的变量，
``Pop Stop`` 来引用下一个，以此类推。

.. code-block:: idris

    Var : HasType i G t -> Expr G t

.. So, in an expression ``\x,\y. x y``, the variable ``x`` would have a
.. de Bruijn index of 1, represented as ``Pop Stop``, and ``y 0``,
.. represented as ``Stop``. We find these by counting the number of
.. lambdas between the definition and the use.

所以，在表达式 ``\x. \y. x y`` 中，变量 ``x`` 的 de Bruijn 索引为 1，
可表示为 ``Pop Stop`` ；变量 ``y`` 的 de Bruijn 索引为 0，可表示为 ``Stop`` 。
我们通过计算定义和使用之间 λ 的数量来查找它们。

.. A value carries a concrete representation of an integer:

值携带了整数的具体表示：

.. code-block:: idris

    Val : (x : Integer) -> Expr G TyInt

.. A lambda creates a function. In the scope of a function of type ``a ->
.. t``, there is a new local variable of type ``a``, which is expressed
.. by the context index:

λ 用于创建函数。在类型为 ``a -> t`` 的函数的作用域内，有一个新的类型为
``a`` 的局部变量，它用上下文索引来表示：

.. code-block:: idris

    Lam : Expr (a :: G) t -> Expr G (TyFun a t)

.. Function application produces a value of type ``t`` given a function
.. from ``a`` to ``t`` and a value of type ``a``:

函数应用接受一个从 ``a`` 到 ``t`` 的函数和一个类型为 ``a`` 的值，产生一个类型为
``t`` 的值。

.. code-block:: idris

    App : Expr G (TyFun a t) -> Expr G a -> Expr G t

.. We allow arbitrary binary operators, where the type of the operator
.. informs what the types of the arguments must be:

我们允许任意的二元运算符，运算符的类型会告诉我们参数的类型：

.. code-block:: idris

    Op : (interpTy a -> interpTy b -> interpTy c) ->
         Expr G a -> Expr G b -> Expr G c

.. Finally, ``If`` expressions make a choice given a boolean. Each branch
.. must have the same type, and we will evaluate the branches lazily so
.. that only the branch which is taken need be evaluated:

最后，``If`` 表达式根据给定的布尔值来做出选择，每个分支必须有相同的类型。
我们将对分支使用惰性求值，只有被选择的分支才会被求值：

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

当我们对一个 ``Expr`` 求值时，我们需要其作用域内的值及其类型。 ``Env``
是一个根据其作用域中的类型来索引的环境。尽管环境和局部变量类型的向量有着很强联系，
它也只是列表的另一种形式。我们使用了一般的 ``::`` 和 ``Nil`` 构造器，
这样就能使用一般的列表语法了。给定一个变量在上下文中定义的证明，我们可以从环境中得到一个值：

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

有了上述内容，解释器就是一个根据特定环境将 ``Expr`` 转换成一个具体的 Idris 值的函数了：

.. code-block:: idris

    interp : Env G -> Expr G t -> interpTy t

.. The complete interpreter is defined as follows, for reference. For
.. each constructor, we translate it into the corresponding Idris value:

作为参考，解释器的完整定义如下。对于每一个构造器，我们把它转换成对应的 Idris 值：

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

我们来逐一观察每一种情况。对于一个变量，我们只要从环境中找出它：

.. code-block:: idris

    interp env (Var i) = lookup i env

.. To translate a value, we just return the concrete representation of the
.. value:

对于一个值，我们只需要返回其实际的表达方式：

.. code-block:: idris

    interp env (Val x) = x

.. Lambdas are more interesting. In this case, we construct a function
.. which interprets the scope of the lambda with a new value in the
.. environment. So, a function in the object language is translated to an
.. Idris function:

λ 则比较有意思。我们构造了一个解释 λ 内部作用域的函数，但其环境中带有一个新的值。
因此，目标语言的函数可以用如下的方法来转换成 Idris 函数：

.. code-block:: idris

    interp env (Lam sc) = \x => interp (x :: env) sc

.. For an application, we interpret the function and its argument and apply
.. it directly. We know that interpreting ``f`` must produce a function,
.. because of its type:

对于函数应用，我们解释函数及其参数，并直接将函数应用于参数。我们知道，根据其类型，
解释 ``f`` 必定会得到一个函数：

.. code-block:: idris

    interp env (App f s) = interp env f (interp env s)

.. Operators and conditionals are, again, direct translations into the
.. equivalent Idris constructs. For operators, we apply the function to
.. its operands directly, and for ``If``, we apply the Idris
.. ``if...then...else`` construct directly.

运算符和条件分支同样会转换成等价的 Idris 构造。对于运算符，我们将函数直接应用到
操作数上；对于 ``If``，我们直接使用 Idris 的 ``if...then...else`` 构造。

.. code-block:: idris

    interp env (Op op x y) = op (interp env x) (interp env y)
    interp env (If x t e)  = if interp env x then interp env t
                                             else interp env e

测试
====

.. Testing
.. =======

.. We can make some simple test functions. Firstly, adding two inputs
.. ``\x. \y. y + x`` is written as follows:

我们可以编写一个简单的测试函数。首先，将两个输入值加起来，``\x. \y. y + x``
可写成如下形式：

.. code-block:: idris

    add : Expr G (TyFun TyInt (TyFun TyInt TyInt))
    add = Lam (Lam (Op (+) (Var Stop) (Var (Pop Stop))))

.. More interestingly, a factorial function ``fact``
.. (e.g. ``\x. if (x == 0) then 1 else (fact (x-1) * x)``),
.. can be written as:

更有趣的是阶乘函数 ``fact``，如 ``\x. if (x == 0) then 1 else (fact (x-1) * x)``，
可写成如下形式：

.. code-block:: idris

    fact : Expr G (TyFun TyInt TyInt)
    fact = Lam (If (Op (==) (Var Stop) (Val 0))
                   (Val 1)
                   (Op (*) (App fact (Op (-) (Var Stop) (Val 1)))
                           (Var Stop)))

运行
====

.. Running
.. =======

.. To finish, we write a ``main`` program which interprets the factorial
.. function on user input:

作为结束，我们编写一个 ``main`` 程序，它根据用户的输入来解释阶乘函数：

.. code-block:: idris

    main : IO ()
    main = do putStr "Enter a number: "
              x <- getLine
              printLn (interp [] fact (cast x))

.. Here, ``cast`` is an overloaded function which converts a value from
.. one type to another if possible. Here, it converts a string to an
.. integer, giving 0 if the input is invalid. An example run of this
.. program at the Idris interactive environment is:

此处的 ``cast`` 是一个被重载的函数，在可能的情况下，它将值转为另一种类型。在这里，
它将字符串转换成了整数，在输入不合法时返回 0 。在 Idris 的交互式环境运行中这个程序
会产生如下结果：

.. _factrun:
.. literalinclude:: ../listing/idris-prompt-interp.txt


题外话：``cast``
----------------

.. Aside: ``cast``
.. ---------------

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

这是一个 **多参数** 接口，定义了转换的源类型和目标类型。在转换被应用的地方，
类型检查器必须能够推导出 **两个** 参数。原语类型间有意义的转换均已定义。
