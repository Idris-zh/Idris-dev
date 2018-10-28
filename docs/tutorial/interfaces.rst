.. _sect-interfaces:

****
接口
****

.. **********
.. Interfaces
.. **********

.. We often want to define functions which work across several different
.. data types. For example, we would like arithmetic operators to work on
.. ``Int``, ``Integer`` and ``Double`` at the very least. We would like
.. ``==`` to work on the majority of data types. We would like to be able
.. to display different types in a uniform way.

我们经常想要定义能够跨多种不同数据类型工作的函数。例如，我们想要算术运算符至少可以作用于
``Int``、``Integer`` 和 ``Double``。我们想要 ``==`` 作用于大部分的数据类型。
我们想要以一种统一的方式来显示不同的类型。

.. To achieve this, we use *interfaces*, which are similar to type classes in
.. Haskell or traits in Rust. To define an interface, we provide a collection of
.. overloadable functions. A simple example is the ``Show``
.. interface, which is defined in the prelude and provides an interface for
.. converting values to ``String``:

为此，我们使用了 **接口（Interface）**，它类似于 Haskell 中的类型类（Typeclass）或 Rust
中的特性（Trait）。为了定义接口，我们提供了一组可重载的的函数。``Show`` 接口就是个简单的例子，
它在 Prelude 中定义，并提供了将值转换为 ``String`` 的接口：

.. code-block:: idris

    interface Show a where
        show : a -> String

.. This generates a function of the following type (which we call a
.. *method* of the ``Show`` interface):

它会生成一个类型如下的函数，我们称之为 ``Show`` 接口的 **方法（Method）**：

.. code-block:: idris

    show : Show a => a -> String

.. We can read this as: “under the constraint that ``a`` has an implementation
.. of ``Show``, take an input ``a`` and return a ``String``.” An implementation
.. of an interface is defined by giving definitions of the methods of the interface.
.. For example, the ``Show`` implementation for ``Nat`` could be defined as:

我们可以把它读作：「在 ``a`` 实现了 ``Show`` 的约束下，该函数接受一个输入  ``a``
并返回一个  ``String``。」我们可以通过为它定义接口的方法来实现该接口。例如，``Nat``
的 ``Show`` 实现可定义为：

.. code-block:: idris

    Show Nat where
        show Z = "Z"
        show (S k) = "s" ++ show k

::

    Idris> show (S (S (S Z)))
    "sssZ" : String

.. Only one implementation of an interface can be given for a type — implementations may
.. not overlap. Implementation declarations can themselves have constraints.
.. To help with resolution, the arguments of an implementation must be
.. constructors (either data or type constructors) or variables
.. (i.e. you cannot give an implementation for a function). For
.. example, to define a ``Show`` implementation for vectors, we need to know
.. that there is a ``Show`` implementation for the element type, because we are
.. going to use it to convert each element to a ``String``:

一个类型只能实现一次某个接口，也就是说，实现无法被覆盖。实现的声明自身可以包含约束。
为此，实现的参数必须为构造器（即数据或类型构造器）或变量（也就是说，你无法为函数赋予实现）。
例如，要为向量定义一个 ``Show`` 的实现，我们需要确认其元素实现了 ``Show``，
因为要用它来将每个元素都转换为 ``String``：

.. code-block:: idris

    Show a => Show (Vect n a) where
        show xs = "[" ++ show' xs ++ "]" where
            show' : Vect n a -> String
            show' Nil        = ""
            show' (x :: Nil) = show x
            show' (x :: xs)  = show x ++ ", " ++ show' xs

.. hint::

    译者更愿意将它读作：对于一个实现了 ``Show`` 的 ``a``， ``Vect n a`` 对
    ``Show`` 的实现为……



默认定义
========

.. Default Definitions
.. ===================

.. The library defines an ``Eq`` interface which provides methods for
.. comparing values for equality or inequality, with implementations for all of
.. the built-in types:

库中定义了 ``Eq`` 接口，它提供了比较值是否相等的方法，所有内建类型都实现了它：

.. code-block:: idris

    interface Eq a where
        (==) : a -> a -> Bool
        (/=) : a -> a -> Bool

.. To declare an implementation for a type, we have to give definitions of all
.. of the methods. For example, for an implementation of ``Eq`` for ``Nat``:

要为类型实现一个接口，我们必须给出所有方法的定义。例如，为 ``Nat`` 实现 ``Eq``：

.. code-block:: idris

    Eq Nat where
        Z     == Z     = True
        (S x) == (S y) = x == y
        Z     == (S y) = False
        (S x) == Z     = False

        x /= y = not (x == y)

.. It is hard to imagine many cases where the ``/=`` method will be
.. anything other than the negation of the result of applying the ``==``
.. method. It is therefore convenient to give a default definition for
.. each method in the interface declaration, in terms of the other method:

很难想象在哪些情况下 ``/=`` 方法不是应用了 ``==`` 方法的结果的否定。
因此，利用接口声明中的某个方法为其它方法提供默认的定义会很方便：

.. code-block:: idris

    interface Eq a where
        (==) : a -> a -> Bool
        (/=) : a -> a -> Bool

        x /= y = not (x == y)
        x == y = not (x /= y)

.. A minimal complete implementation of ``Eq`` requires either
.. ``==`` or ``/=`` to be defined, but does not require both. If a method
.. definition is missing, and there is a default definition for it, then
.. the default is used instead.

``Eq`` 的最小完整实现只需要定义 ``==`` 或 ``/=`` 二者之一，而不需要二者都定义。
若缺少某个方法定义，且存在它的默认定义，那么就会使用该默认定义。

扩展接口
========

.. Extending Interfaces
.. ====================

.. Interfaces can also be extended. A logical next step from an equality
.. relation ``Eq`` is to define an ordering relation ``Ord``. We can
.. define an ``Ord`` interface which inherits methods from ``Eq`` as well as
.. defining some of its own:

接口也可以扩展。逻辑上，相等关系 ``Eq`` 的下一步是定义排序关系 ``Ord``。
我们可以定义一个 ``Ord`` 接口，它除了继承 ``Eq`` 的方法外还定义了自己的方法：

.. code-block:: idris

    data Ordering = LT | EQ | GT

.. code-block:: idris

    interface Eq a => Ord a where
        compare : a -> a -> Ordering

        (<) : a -> a -> Bool
        (>) : a -> a -> Bool
        (<=) : a -> a -> Bool
        (>=) : a -> a -> Bool
        max : a -> a -> a
        min : a -> a -> a

.. The ``Ord`` interface allows us to compare two values and determine their
.. ordering. Only the ``compare`` method is required; every other method
.. has a default definition. Using this we can write functions such as
.. ``sort``, a function which sorts a list into increasing order,
.. provided that the element type of the list is in the ``Ord`` interface. We
.. give the constraints on the type variables left of the fat arrow
.. ``=>``, and the function type to the right of the fat arrow:

 ``Ord`` 接口允许我们比较两个值并确定二者的顺序。其中只有 ``compare`` 方法是必须的，
 其它方法都有默认定义。我们可以用它来编写 ``sort`` 之类的函数，它将一个列表按升序排列，
 只要列表中的元素类型实现了 ``Ord`` 接口即可。我们为宽箭头 ``=>`` 左侧的类型变量加上约束，
 为右侧加上函数的类型：

.. code-block:: idris

    sort : Ord a => List a -> List a

.. Functions, interfaces and implementations can have multiple
.. constraints. Multiple constraints are written in brackets in a comma
.. separated list, for example:

函数、接口和实现可拥有多个约束。多个约束的列表写在括号内，以逗号分隔，例如：

.. code-block:: idris

    sortAndShow : (Ord a, Show a) => List a -> String
    sortAndShow xs = show (sort xs)

注意：接口与 ``mutual`` 块
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Note: Interfaces and ``mutual`` blocks
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Idris is strictly "define before use", except in ``mutual`` blocks.
.. In a ``mutual`` block, Idris elaborates in two passes: types on the first
.. pass and definitions on the second. When the mutual block contains an
.. interface declaration, it elaborates the interface header but none of the
.. method types on the first pass, and elaborates the method types and any
.. default definitions on the second pass.

除 ``mutual`` 块外，Idris 严格遵循「先定义后使用」的规则。在 ``mutual`` 块中，
Idris 会分两趟进行繁释（elaborate）：第一趟为类型，第二趟为定义。当互用块中包含接口声明时，
第一趟会繁释接口的头部而不繁释方法类型；第二趟则繁释方法类型以及所有的默认定义。

函子与应用子
============

.. Functors and Applicatives
.. =========================

.. So far, we have seen single parameter interfaces, where the parameter
.. is of type ``Type``. In general, there can be any number of parameters
.. (even zero), and the parameters can have *any* type. If the type
.. of the parameter is not ``Type``, we need to give an explicit type
.. declaration. For example, the ``Functor`` interface is defined in the
.. prelude:

目前，我们已经见过形参类型为 ``Type`` 的单形参接口了。通常，形参的个数可为任意个
（甚至零个），而形参也可为 **任意** 类型。若形参的类型不为 ``Type``，
我们则需要提供显式的类型声明。例如，Prelude 中定义的函子接口 ``Functor`` 为：

.. code-block:: idris

    interface Functor (f : Type -> Type) where
        map : (m : a -> b) -> f a -> f b

.. A functor allows a function to be applied across a structure, for
.. example to apply a function to every element in a ``List``:

函子允许函数应用到结构上，例如将一个函数应用到 ``List`` 的每一个元素上：

.. code-block:: idris

    Functor List where
      map f []      = []
      map f (x::xs) = f x :: map f xs

::

    Idris> map (*2) [1..10]
    [2, 4, 6, 8, 10, 12, 14, 16, 18, 20] : List Integer

.. Having defined ``Functor``, we can define ``Applicative`` which
.. abstracts the notion of function application:

定义了 ``Functor`` 之后，我们就能定义应用子 ``Applicative`` 了，
它对函数应用的概念进行了抽象：

.. code-block:: idris

    infixl 2 <*>

    interface Functor f => Applicative (f : Type -> Type) where
        pure  : a -> f a
        (<*>) : f (a -> b) -> f a -> f b

.. _monadsdo:

单子与 ``do`` 记法
==================

.. Monads and ``do``-notation
.. ==========================

.. The ``Monad`` interface allows us to encapsulate binding and computation,
.. and is the basis of ``do``-notation introduced in Section
.. :ref:`sect-do`. It extends ``Applicative`` as defined above, and is
.. defined as follows:

单子接口 ``Monad`` 允许我们对绑定和计算进行封装，它也是 :ref:`sect-do` 一节中
``do`` 记法的基础。单子扩展了前面定义的 ``Applicative``，其定义如下：

.. code-block:: idris

    interface Applicative m => Monad (m : Type -> Type) where
        (>>=)  : m a -> (a -> m b) -> m b

.. Inside a ``do`` block, the following syntactic transformations are
.. applied:

.. - ``x <- v; e`` becomes ``v >>= (\x => e)``

.. - ``v; e`` becomes ``v >>= (\_ => e)``

.. - ``let x = v; e`` becomes ``let x = v in e``

在 ``do`` 块中会应用以下语法变换：

- ``x <- v; e`` 变为 ``v >>= (\x => e)``

- ``v; e`` 变为 ``v >>= (\_ => e)``

- ``let x = v; e`` 变为 ``let x = v in e``

.. ``IO`` has an implementation of ``Monad``, defined using primitive functions.
.. We can also define an implementation for ``Maybe``, as follows:

``IO`` 实现了 ``Monad``，它使用原语函数定义。我们也可以为 ``Maybe`` 定义实现，
其实现如下：

.. code-block:: idris

    Monad Maybe where
        Nothing  >>= k = Nothing
        (Just x) >>= k = k x

.. Using this we can, for example, define a function which adds two
.. ``Maybe Int``, using the monad to encapsulate the error handling:

.. .. code-block:: idris

..     m_add : Maybe Int -> Maybe Int -> Maybe Int
..     m_add x y = do x' <- x -- Extract value from x
..                    y' <- y -- Extract value from y
..                    pure (x' + y') -- Add them

通过它，我们可以定义一个将两个 ``Maybe Int`` 相加的函数，并用单子来封装错误处理：

.. code-block:: idris

    m_add : Maybe Int -> Maybe Int -> Maybe Int
    m_add x y = do x' <- x -- 从 x 中提取值
                   y' <- y -- 从 y 中提取值
                   pure (x' + y') -- 二者相加

.. This function will extract the values from ``x`` and ``y``, if they
.. are both available, or return ``Nothing`` if one or both are not ("fail fast"). Managing the
.. ``Nothing`` cases is achieved by the ``>>=`` operator, hidden by the
.. ``do`` notation.

若 ``x`` 和 ``y`` 均可用，该函数会从二者中提取出值；若其中一个或二者均不可用，
则返回 ``Nothing`` （「fail-fast 速错原则」）。``Nothing`` 的情况通过 ``>>=``
操作符来管理，由 ``do`` 记法来隐藏。

::

    *Interfaces> m_add (Just 20) (Just 22)
    Just 42 : Maybe Int
    *Interfaces> m_add (Just 20) Nothing
    Nothing : Maybe Int

模式匹配绑定
~~~~~~~~~~~~

.. Pattern Matching Bind
.. ~~~~~~~~~~~~~~~~~~~~~

.. Sometimes we want to pattern match immediately on the result of a function
.. in ``do`` notation. For example, let's say we have a function ``readNumber``
.. which reads a number from the console, returning a value of the form
.. ``Just x`` if the number is valid, or ``Nothing`` otherwise:

有时我们需要立即对 ``do`` 记法中某个函数的结果进行模式匹配。例如，假设我们有一个函数
``readNumber``，它从控制台读取一个数，若该数有效则返回 ``Just x`` 形式的值，否则返回
``Nothing`` ：

.. code-block:: idris

    readNumber : IO (Maybe Nat)
    readNumber = do
      input <- getLine
      if all isDigit (unpack input)
         then pure (Just (cast input))
         else pure Nothing

.. If we then use it to write a function to read two numbers, returning
.. ``Nothing`` if neither are valid, then we would like to pattern match
.. on the result of ``readNumber``:

如果接着用它来编写读取两个数，若二者均无效则返回 ``Nothing`` 的函数，
那么我们可能想要对 ``readNumber`` 进行模式匹配：

.. code-block:: idris

    readNumbers : IO (Maybe (Nat, Nat))
    readNumbers =
      do x <- readNumber
         case x of
              Nothing => pure Nothing
              Just x_ok => do y <- readNumber
                              case y of
                                   Nothing => pure Nothing
                                   Just y_ok => pure (Just (x_ok, y_ok))

.. If there's a lot of error handling, this could get deeply nested very quickly!
.. So instead, we can combine the bind and the pattern match in one line. For example,
.. we could try pattern matching on values of the form ``Just x_ok``:

如果有很多错误需要处理，它的嵌套层次很快就会变得非常深！我们不妨将绑定和模式匹配组合成一行。
例如，我们可以对 ``Just x_ok`` 的形式进行模式匹配：

.. code-block:: idris

    readNumbers : IO (Maybe (Nat, Nat))
    readNumbers =
      do Just x_ok <- readNumber
         Just y_ok <- readNumber
         pure (Just (x_ok, y_ok))

.. There is still a problem, however, because we've now omitted the case for
.. ``Nothing`` so ``readNumbers`` is no longer total! We can add the ``Nothing``
.. case back as follows:

然而问题仍然存在，我们现在忽略了 ``Nothing`` 的情况，所以该函数不再是完全的了！
我们可以把 ``Nothing`` 的情况添加回去：

.. code-block:: idris

    readNumbers : IO (Maybe (Nat, Nat))
    readNumbers =
      do Just x_ok <- readNumber | Nothing => pure Nothing
         Just y_ok <- readNumber | Nothing => pure Nothing
         pure (Just (x_ok, y_ok))

.. The effect of this version of ``readNumbers`` is identical to the first (in
.. fact, it is syntactic sugar for it and directly translated back into that form).
.. The first part of each statement (``Just x_ok <-`` and ``Just y_ok <-``) gives
.. the preferred binding - if this matches, execution will continue with the rest
.. of the ``do`` block. The second part gives the alternative bindings, of which
.. there may be more than one.

此版本的 ``readNumbers`` 效果与初版完全一样（实际上，它是初版的语法糖，
并且会直接被翻译回初版的形式）。每条语句的第一部分（``Just x_ok <-`` 和
``Just y_ok <-``）给出了首选的绑定：若能匹配，``do`` 块的剩余部分就会继续执行。
第二部分给出了选取的绑定，其中的绑定可以有不止一个。

``!`` 记法
~~~~~~~~~~

.. ``!``-notation
.. ~~~~~~~~~~~~~~

.. In many cases, using ``do``-notation can make programs unnecessarily
.. verbose, particularly in cases such as ``m_add`` above where the value
.. bound is used once, immediately. In these cases, we can use a
.. shorthand version, as follows:

在很多情况下，``do`` 记法会让程序不必要地啰嗦，在将值绑定一次就立即使用的情况下尤甚，
例如前面的 ``m_add``。此时我们可以使用更加简短的方式：

.. code-block:: idris

    m_add : Maybe Int -> Maybe Int -> Maybe Int
    m_add x y = pure (!x + !y)

.. The notation ``!expr`` means that the expression ``expr`` should be
.. evaluated and then implicitly bound. Conceptually, we can think of
.. ``!`` as being a prefix function with the following type:

``!expr`` 记法表示 ``expr`` 应当在求值后立即被隐式绑定。从概念上讲，我们可以把
``!`` 看做拥有以下类型的前缀函数：

.. code-block:: idris

    (!) : m a -> a

.. Note, however, that it is not really a function, merely syntax! In
.. practice, a subexpression ``!expr`` will lift ``expr`` as high as
.. possible within its current scope, bind it to a fresh name ``x``, and
.. replace ``!expr`` with ``x``. Expressions are lifted depth first, left
.. to right. In practice, ``!``-notation allows us to program in a more
.. direct style, while still giving a notational clue as to which
.. expressions are monadic.

然而请注意，它并不是一个真的函数，而只是一个语法！在实践中，子表达式 ``!expr``
会在 ``expr`` 的当前作用域内尽可能地提升，将它绑定到一个全新的名字 ``x``，
然后用它来代替 ``!expr``。首先表达式会按从左到右的顺序深度优先地上升。在实践中，``!``
记法允许我们以更直接的方式来编程，同时该记法也标出了哪些表达式为单子。

.. For example, the expression:

例如，表达式：

.. code-block:: idris

    let y = 42 in f !(g !(print y) !x)

.. is lifted to:

会被提升为：

.. code-block:: idris

    let y = 42 in do y' <- print y
                     x' <- x
                     g' <- g y' x'
                     f g'

单子推导式
~~~~~~~~~~

.. Monad comprehensions
.. ~~~~~~~~~~~~~~~~~~~~

.. The list comprehension notation we saw in Section
.. :ref:`sect-more-expr` is more general, and applies to anything which
.. has an implementation of both ``Monad`` and ``Alternative``:

我们之间在 :ref:`sect-more-expr` 一节中看到的列表推导记法其实更通用，
它可应用于任何实现了 ``Monad`` 和 ``Alternative`` 的东西：

.. code-block:: idris

    interface Applicative f => Alternative (f : Type -> Type) where
        empty : f a
        (<|>) : f a -> f a -> f a

.. In general, a comprehension takes the form ``[ exp | qual1, qual2, …,
.. qualn ]`` where ``quali`` can be one of:

.. - A generator ``x <- e``

.. - A *guard*, which is an expression of type ``Bool``

.. - A let binding ``let x = e``

通常，推导式形式为 ``[ exp | qual1, qual2, …, qualn ]`` 其中 ``quali`` 可以为：

- 一个生成式 ``x <- e``

- 一个 **守卫式（Guard）**，它是一个类型为 ``Bool`` 的表达式

- 一个 let 绑定 ``let x = e``

.. To translate a comprehension ``[exp | qual1, qual2, …, qualn]``, first
.. any qualifier ``qual`` which is a *guard* is translated to ``guard
.. qual``, using the following function:

要翻译一个推导式 ``[exp | qual1, qual2, …, qualn]``，首先任何作为 **守卫式** 的限定式
``qual`` 会使用以下函数翻译为 ``guard qual``：

.. code-block:: idris

    guard : Alternative f => Bool -> f ()

.. Then the comprehension is converted to ``do`` notation:

接着该推导式会被转换为 ``do`` 记法：

.. code-block:: idris

    do { qual1; qual2; ...; qualn; pure exp; }

.. Using monad comprehensions, an alternative definition for ``m_add``
.. would be:

使用单子推导式，``m_add`` 的选取（alternative）定义为：

.. code-block:: idris

    m_add : Maybe Int -> Maybe Int -> Maybe Int
    m_add x y = [ x' + y' | x' <- x, y' <- y ]

习语括号
========

.. Idiom brackets
.. ==============

.. While ``do`` notation gives an alternative meaning to sequencing,
.. idioms give an alternative meaning to *application*. The notation and
.. larger example in this section is inspired by Conor McBride and Ross
.. Paterson’s paper “Applicative Programming with Effects” [1]_.

``do`` 记法为串连提供了另一种写法，而惯用法则为 **应用** 提供了另一种写法。
本节中的记法以及大量的例子受到了 Conor McBride 和 Ross Paterson 的论文
「带作用的应用子编程」 [1]_ 的启发。

.. First, let us revisit ``m_add`` above. All it is really doing is
.. applying an operator to two values extracted from ``Maybe Int``. We
.. could abstract out the application:

首先，让我们回顾一下前面的 ``m_add``。它所做的只是将一个操作符应用到两个从
``Maybe Int`` 中提取的值。我们可以抽象出该应用：

.. code-block:: idris

    m_app : Maybe (a -> b) -> Maybe a -> Maybe b
    m_app (Just f) (Just a) = Just (f a)
    m_app _        _        = Nothing

.. Using this, we can write an alternative ``m_add`` which uses this
.. alternative notion of function application, with explicit calls to
.. ``m_app``:

我们可以用它来编写另一种 ``m_add``，它使用了函数应用的选取概念，带有显式的
``m_app`` 调用：

.. code-block:: idris

    m_add' : Maybe Int -> Maybe Int -> Maybe Int
    m_add' x y = m_app (m_app (Just (+)) x) y

.. Rather than having to insert ``m_app`` everywhere there is an
.. application, we can use idiom brackets to do the job for us.
.. To do this, we can give ``Maybe`` an implementation of ``Applicative``
.. as follows, where ``<*>`` is defined in the same way as ``m_app``
.. above (this is defined in the Idris library):

与其到处插入 ``m_app``，我们不如使用习语括号（Idiom Brackets）来做这件事。
为此，我们可以像下面这样为 ``Maybe`` 提供一个 ``Applicative`` 的实现，其中
``<*>`` 的定义方式与前面的 ``m_app`` 相同（它已在 Idris 库中定义）：

.. code-block:: idris

    Applicative Maybe where
        pure = Just

        (Just f) <*> (Just a) = Just (f a)
        _        <*> _        = Nothing

.. Using ``<*>`` we can use this implementation as follows, where a function
.. application ``[| f a1 …an |]`` is translated into ``pure f <*> a1 <*>
.. … <*> an``:

按照 ``<*>`` 的实现，我们可以像下面这样使用它，其中函数应用 ``[| f a1 … an |]``
会被翻译成 ``pure f <*> a1 <*> … <*> an``：

.. code-block:: idris

    m_add' : Maybe Int -> Maybe Int -> Maybe Int
    m_add' x y = [| x + y |]

错误处理解释器
~~~~~~~~~~~~~~

.. An error-handling interpreter
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Idiom notation is commonly useful when defining evaluators. McBride
.. and Paterson describe such an evaluator [1]_, for a language similar
.. to the following:

惯用记法在定义求值器时通常很有用。McBride 和 Paterson
就为下面这样的语言描述了求值器 [1]_ ：

.. .. code-block:: idris

..     data Expr = Var String      -- variables
..               | Val Int         -- values
..               | Add Expr Expr   -- addition

.. code-block:: idris

    data Expr = Var String      -- 变量
              | Val Int         -- 值
              | Add Expr Expr   -- 加法

.. Evaluation will take place relative to a context mapping variables
.. (represented as ``String``\s) to ``Int`` values, and can possibly fail.
.. We define a data type ``Eval`` to wrap an evaluator:

求值会被根据上下文将变量（表示为 ``String``）映射为 ``Int`` 值，且可能会失败。
我们定义了数据类型 ``Eval`` 来包装求值器：

.. code-block:: idris

    data Eval : Type -> Type where
         MkEval : (List (String, Int) -> Maybe a) -> Eval a

.. Wrapping the evaluator in a data type means we will be able to provide
.. implementations of interfaces for it later. We begin by defining a function to
.. retrieve values from the context during evaluation:

将求值器包装在数据类型中意味着我们之后可以为它提供接口的实现。我们首先定义一个函数，
它在求值过程中从上下文取出值。

.. code-block:: idris

    fetch : String -> Eval Int
    fetch x = MkEval (\e => fetchVal e) where
        fetchVal : List (String, Int) -> Maybe Int
        fetchVal [] = Nothing
        fetchVal ((v, val) :: xs) = if (x == v)
                                      then (Just val)
                                      else (fetchVal xs)

.. When defining an evaluator for the language, we will be applying functions in
.. the context of an ``Eval``, so it is natural to give ``Eval`` an implementation
.. of ``Applicative``. Before ``Eval`` can have an implementation of
.. ``Applicative`` it is necessary for ``Eval`` to have an implementation of
.. ``Functor``:

在定义该语言的求值器时，我们会应用 ``Eval`` 上下文中的函数，这样它自然会为
``Eval`` 提供 ``Applicative`` 的实现。在 ``Eval`` 能够实现 ``Applicative``
之前，我们必须为 ``Eval`` 实现 ``Functor``：

.. code-block:: idris

    Functor Eval where
        map f (MkEval g) = MkEval (\e => map f (g e))

    Applicative Eval where
        pure x = MkEval (\e => Just x)

        (<*>) (MkEval f) (MkEval g) = MkEval (\x => app (f x) (g x)) where
            app : Maybe (a -> b) -> Maybe a -> Maybe b
            app (Just fx) (Just gx) = Just (fx gx)
            app _         _         = Nothing

.. Evaluating an expression can now make use of the idiomatic application
.. to handle errors:

现在就可以在求值表达式时，通过应用的惯用法来处理错误了：

.. code-block:: idris

    eval : Expr -> Eval Int
    eval (Var x)   = fetch x
    eval (Val x)   = [| x |]
    eval (Add x y) = [| eval x + eval y |]

    runEval : List (String, Int) -> Expr -> Maybe Int
    runEval env e = case eval e of
        MkEval envFn => envFn env

命名实现
========

.. Named Implementations
.. =====================

.. It can be desirable to have multiple implementations of an interface for the
.. same type, for example to provide alternative methods for sorting or printing
.. values. To achieve this, implementations can be *named* as follows:

有时我们希望一个类型可以拥有一个接口的多个实现，例如为排序或打印提供另一种方法。
为此，实现可以像下面这样 **命名**：

.. code-block:: idris

    [myord] Ord Nat where
       compare Z (S n)     = GT
       compare (S n) Z     = LT
       compare Z Z         = EQ
       compare (S x) (S y) = compare @{myord} x y

.. This declares an implementation as normal, but with an explicit name,
.. ``myord``. The syntax ``compare @{myord}`` gives an explicit implementation to
.. ``compare``, otherwise it would use the default implementation for ``Nat``. We
.. can use this, for example, to sort a list of ``Nat`` in reverse.
.. Given the following list:

它像往常一样声明了一个实现，不过带有显式的名字 ``myord``。语法
``compare @{myord}`` 会为 ``compare`` 提供显式的实现，否则它会使用 ``Nat``
的默认实现。我们可以用它来反向排序一个 ``Nat`` 列表。给定以下列表：

.. code-block:: idris

    testList : List Nat
    testList = [3,4,1]

.. We can sort it using the default ``Ord`` implementation, then the named
.. implementation ``myord`` as follows, at the Idris prompt:

我们可以在 Idris 提示符中用默认的 ``Ord`` 实现来排序，之后使用命名的实现 ``myord``：

::

    *named_impl> show (sort testList)
    "[sO, sssO, ssssO]" : String
    *named_impl> show (sort @{myord} testList)
    "[ssssO, sssO, sO]" : String


.. Sometimes, we also need access to a named parent implementation. For example,
.. the prelude defines the following ``Semigroup`` interface:

有时，我们也需要访问命名的父级实现。例如 Prelude 中定义的半群 ``Semigroup`` 接口：

.. code-block:: idris

    interface Semigroup ty where
      (<+>) : ty -> ty -> ty

.. Then it defines ``Monoid``, which extends ``Semigroup`` with a “neutral”
.. value:

接着又定义了幺半群 ``Monoid``，它用「幺元」 ``neutral`` 扩展了 ``Semigroup``：

.. code-block:: idris

    interface Semigroup ty => Monoid ty where
      neutral : ty

.. We can define two different implementations of ``Semigroup`` and
.. ``Monoid`` for ``Nat``, one based on addition and one on multiplication:

我们可以为 ``Nat`` 定义 ``Semigroup`` 与 ``Monoid`` 的两种不同的实现，
一个基于加法，一个基于乘法：

.. code-block:: idris

    [PlusNatSemi] Semigroup Nat where
      (<+>) x y = x + y

    [MultNatSemi] Semigroup Nat where
      (<+>) x y = x * y

.. The neutral value for addition is ``0``, but the neutral value for multiplication
.. is ``1``. It's important, therefore, that when we define implementations
.. of ``Monoid`` they extend the correct ``Semigroup`` implementation. We can
.. do this with a ``using`` clause in the implementation as follows:

加法的幺元为 ``0``，而乘法的幺元为 ``1``。因此，我们在定义 ``Monoid`` 的实现时，
保证扩展了正确的 ``Semigroup`` 十分重要。我们可以通过 ``using`` 从句来做到这一点：

.. code-block:: idris

    [PlusNatMonoid] Monoid Nat using PlusNatSemi where
      neutral = 0

    [MultNatMonoid] Monoid Nat using MultNatSemi where
      neutral = 1

.. The ``using PlusNatSemi`` clause indicates that ``PlusNatMonoid`` should
.. extend ``PlusNatSemi`` specifically.

``using PlusNatSemi`` 从句指明 ``PlusNatMonoid`` 应当扩展 ``PlusNatSemi``。

确定形参
========

.. Determining Parameters
.. ======================

.. When an interface has more than one parameter, it can help resolution if the
.. parameters used to find an implementation are restricted. For example:

当接口的形参多于一个时，通过限定形参可帮助查找实现。例如：

.. code-block:: idris

    interface Monad m => MonadState s (m : Type -> Type) | m where
      get : m s
      put : s -> m ()

.. In this interface, only ``m`` needs to be known to find an implementation of
.. this interface, and ``s`` can then be determined from the implementation. This
.. is declared with the ``| m`` after the interface declaration. We call ``m`` a
.. *determining parameter* of the ``MonadState`` interface, because it is the
.. parameter used to find an implementation.

在此接口中，查找该接口的实现只需要知道 ``m`` 即可，而 ``s`` 可根据实现来确定。
它通过在接口声明之后添加 ``| m`` 来声明。我们将 ``m`` 称为 ``MonadState``
接口的 **确定形参（Determining Parameter）** ，因为它是用于查找实现的形参。


.. [1] Conor McBride and Ross Paterson. 2008. Applicative programming
       with effects. J. Funct. Program. 18, 1 (January 2008),
       1-13. DOI=10.1017/S0956796807006326
       http://dx.doi.org/10.1017/S0956796807006326
