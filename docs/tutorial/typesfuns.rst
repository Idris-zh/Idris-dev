.. _sect-typefuns:

**********
类型与函数
**********

.. *******************
.. Types and Functions
.. *******************

原语类型
========

.. Primitive Types
.. ===============

.. Idris defines several primitive types: ``Int``, ``Integer`` and
.. ``Double`` for numeric operations, ``Char`` and ``String`` for text
.. manipulation, and ``Ptr`` which represents foreign pointers. There are
.. also several data types declared in the library, including ``Bool``,
.. with values ``True`` and ``False``. We can declare some constants with
.. these types. Enter the following into a file ``Prims.idr`` and load it
.. into the Idris interactive environment by typing ``idris Prims.idr``:

Idris 定义了一些原语（Primitive）类型：``Int``、``Integer`` 和 ``Double`` 用于数值类型，
``Char`` 和 ``String`` 用于文本操作，``Ptr`` 则表示外部指针。库中还声明了一些数据类型，
包括 ``Bool`` 及其值 ``True`` 和 ``False``。我们可以用这些类型来声明常量。
将以下内容保存到文件 ``Prims.idr`` 中，在命令行中输入 ``idris Prims.idr``
以将其加载到 Idris 的交互式环境中：

.. code-block:: idris

    module Prims

    x : Int
    x = 42

    foo : String
    foo = "Sausage machine"

    bar : Char
    bar = 'Z'

    quux : Bool
    quux = False

.. An Idris file consists of an optional module declaration (here
.. ``module Prims``) followed by an optional list of imports and a
.. collection of declarations and definitions. In this example no imports
.. have been specified. However Idris programs can consist of several
.. modules and the definitions in each module each have their own
.. namespace. This is discussed further in Section
.. :ref:`sect-namespaces`. When writing Idris programs both the order in which
.. definitions are given and indentation are significant. Functions and
.. data types must be defined before use, incidentally each definition must
.. have a type declaration, for example see ``x : Int``, ``foo :
.. String``, from the above listing. New declarations must begin at the
.. same level of indentation as the preceding declaration.
.. Alternatively, a semicolon ``;`` can be used to terminate declarations.

.. hint:: 原语（Primitive）本意为不可分割的基本构件，原语类型即最基本的类型。

Idris 文件由一个可选的模块声明（此处为 ``module Prims``），一个可选的导入列表，
一组声明及其定义构成。在本例中并未指定导入。Idris 可由多个模块构成，
每个模块中的每个定义都有它自己的命名空间。这点会在 :ref:`sect-namespaces`
一节中进一步讨论。在编写 Idris 程序时，声明的顺序和缩进都很重要。
函数和数据类型必须先定义再使用，每个定义都必须有类型声明，例如之前列出的
``x : Int`` 和 ``foo : String``。新声明的缩进级别必须与之前的声明相同。
此外，分号 ``;`` 也可用于结束声明。

.. A library module ``prelude`` is automatically imported by every
.. Idris program, including facilities for IO, arithmetic, data
.. structures and various common functions. The prelude defines several
.. arithmetic and comparison operators, which we can use at the prompt.
.. Evaluating things at the prompt gives an answer, and the type of the
.. answer. For example:

库模块 ``Prelude`` 会在每个 Idris 程序中自动导入，包括 IO 功能，算术运算，
数据结构以及多种通用函数。Prelude 中定义了一些算术和比较运算符，
我们可以在提示符中使用它们。在提示符中进行求值会给出一个答案及其类型。例如：

::

    *prims> 6*6+6
    42 : Integer
    *prims> x == 6*6+6
    True : Bool

.. All of the usual arithmetic and comparison operators are defined for
.. the primitive types. They are overloaded using interfaces, as we
.. will discuss in Section :ref:`sect-interfaces` and can be extended to
.. work on user defined types. Boolean expressions can be tested with the
.. ``if...then...else`` construct, for example:

Idris 为原语类型定义了所有的普通算术和比较运算。它们通过接口进行了重载，
并可被扩展以适用于用户定义的类型，这点我们会在 :ref:`sect-interfaces`
一节中讨论。布尔表达式可通过 ``if...then...else`` 构造来测试，例如：

::

    *prims> if x == 6 * 6 + 6 then "The answer!" else "Not the answer"
    "The answer!" : String

数据类型
========

.. Data Types
.. ==========

.. Data types are declared in a similar way and with similar syntax to
.. Haskell. Natural numbers and lists, for example, can be declared as
.. follows:

.. .. code-block:: idris

..     data Nat    = Z   | S Nat           -- Natural numbers
..                                         -- (zero and successor)
..     data List a = Nil | (::) a (List a) -- Polymorphic lists

数据类型的声明方式和语法与 Haskell 非常相似。例如，自然数和列表的声明如下：

.. code-block:: idris

    data Nat    = Z   | S Nat           -- 自然数（零与后继）
    data List a = Nil | (::) a (List a) -- 多态列表

.. The above declarations are taken from the standard library. Unary
.. natural numbers can be either zero (``Z``), or the successor of
.. another natural number (``S k``). Lists can either be empty (``Nil``)
.. or a value added to the front of another list (``x :: xs``). In the
.. declaration for ``List``, we used an infix operator ``::``. New
.. operators such as this can be added using a fixity declaration, as
.. follows:

以上声明来自于标准库。一进制自然数要么为零（``Z``），
要么就是另一个自然数的后继（``S k``）；列表要么为空（``Nil``），
要么就是一个值被添加在另一个列表之前（``x :: xs``）。
在 ``List`` 的声明中，我们使用了中缀操作符 ``::``。
像这样的新操作符可通过缀序声明（Fixity Declaration）来添加，如下所示：

::

    infixr 10 ::

.. Functions, data constructors and type constructors may all be given
.. infix operators as names. They may be used in prefix form if enclosed
.. in brackets, e.g. ``(::)``. Infix operators can use any of the
.. symbols:

函数、数据构造器与类型构造器均可用名字作为中缀操作符。它们被括在括号中时，
也可作为前缀形式使用，例如 ``(::)``。中缀操作符可使用下面的任何符号：

::

    :+-*\/=.?|&><!@$%^~#

.. Some operators built from these symbols can't be user defined. These are
.. ``:``,  ``=>``,  ``->``,  ``<-``,  ``=``,  ``?=``,  ``|``,  ``**``,
.. ``==>``,  ``\``,  ``%``,  ``~``,  ``?``,  and ``!``.

有些以这些符号构成的操作符无法被用户定义。它们是
``:``、``=>``、``->``、``<-``、``=``、``?=``、``|``、``**``、
``==>``、``\``、``%``、``~``、``?`` 以及 ``!``。

函数
====

.. Functions
.. =========

.. Functions are implemented by pattern matching, again using a similar
.. syntax to Haskell. The main difference is that Idris requires type
.. declarations for all functions, using a single colon ``:`` (rather
.. than Haskell’s double colon ``::``). Some natural number arithmetic
.. functions can be defined as follows, again taken from the standard
.. library:

函数通过模式匹配（Pattern Matching）实现，语法同样和 Haskell 类似。
其主要的不同在于 Idris 的所有函数都需要类型声明，声明中使用单冒号 ``:``
（而非 Haskell 的双冒号 ``::`` ）。自然数的某些运算函数定义如下，同样来自标准库：

.. .. code-block:: idris

..     -- Unary addition
..     plus : Nat -> Nat -> Nat
..     plus Z     y = y
..     plus (S k) y = S (plus k y)

..     -- Unary multiplication
..     mult : Nat -> Nat -> Nat
..     mult Z     y = Z
..     mult (S k) y = plus y (mult k y)

.. code-block:: idris

    -- 一进制加法
    plus : Nat -> Nat -> Nat
    plus Z     y = y
    plus (S k) y = S (plus k y)

    -- 一进制乘法
    mult : Nat -> Nat -> Nat
    mult Z     y = Z
    mult (S k) y = plus y (mult k y)

.. The standard arithmetic operators ``+`` and ``*`` are also overloaded
.. for use by ``Nat``, and are implemented using the above functions.
.. Unlike Haskell, there is no restriction on whether types and function
.. names must begin with a capital letter or not. Function names
.. (``plus`` and ``mult`` above), data constructors (``Z``, ``S``,
.. ``Nil`` and ``::``) and type constructors (``Nat`` and ``List``) are
.. all part of the same namespace. By convention, however,
.. data types and constructor names typically begin with a capital letter.
.. We can test these functions at the Idris prompt:

标准算术运算符 ``+`` 和 ``*`` 同样根据 ``Nat`` 的需要进行了重载，
它们使用上面的函数来定义。和 Haskell 不同的是，类型和函数名的首字母并无大小写限制。
函数名（前面的 ``plus`` 和 ``mult`` ），数据构造器（``Z``、``S``、``Nil`` 和 ``::``）
以及类型构造器（``Nat`` 和 ``List``）均属同一命名空间。不过按照约定，
数据类型和构造器的名字通常以大写字母开头。我们可以在 Idris 提示符中测试这些函数：

::

    Idris> plus (S (S Z)) (S (S Z))
    4 : Nat
    Idris> mult (S (S (S Z))) (plus (S (S Z)) (S (S Z)))
    12 : Nat

.. .. note::

..    When displaying an element of ``Nat`` such as ``(S (S (S (S Z))))``,
..    Idris displays it as ``4``.
..    The result of ``plus (S (S Z)) (S (S Z))``
..    is actually ``(S (S (S (S Z))))``
..    which is the natural number ``4``.
..    This can be checked at the Idris prompt:

.. note::

   在显示一个 ``Nat`` 元素，如 ``(S (S (S (S Z))))`` 时，Idris 会将其显示为
   ``4``。 ``plus (S (S Z)) (S (S Z))`` 的结果实际上为 ``(S (S (S (S Z))))``，
   即自然数 ``4``。这点可在 Idris 提示符中验证：

::

    Idris> (S (S (S (S Z))))
    4 : Nat

.. Like arithmetic operations, integer literals are also overloaded using
.. interfaces, meaning that we can also test the functions as follows:

和算术运算符一样，整数字面也可通过接口重载，因此我们也能像下面这样测试函数：

::

    Idris> plus 2 2
    4 : Nat
    Idris> mult 3 (plus 2 2)
    12 : Nat

.. You may wonder, by the way, why we have unary natural numbers when our
.. computers have perfectly good integer arithmetic built in. The reason
.. is primarily that unary numbers have a very convenient structure which
.. is easy to reason about, and easy to relate to other data structures
.. as we will see later. Nevertheless, we do not want this convenience to
.. be at the expense of efficiency. Fortunately, Idris knows about
.. the relationship between ``Nat`` (and similarly structured types) and
.. numbers. This means it can optimise the representation, and functions
.. such as ``plus`` and ``mult``.

你可能会很好奇，既然计算机已经完美内建了整数运算，我们为何还需要一进制的自然数？
主要的原因在于一进制数的结构非常便于推理，而且易与其它数据结构建立联系，
我们之后就会看到。尽管如此，我们并不希望以牺牲效率为代价获得这种便捷。幸运的是，
Idris 知道 ``Nat`` （以及类似的结构化类型）和数之间的联系，
这意味着 Idris 可以优化它们的表示以及像 ``plus`` 和 ``mult`` 这样的函数。

``where`` 从句
--------------

.. ``where`` clauses
.. -----------------

.. Functions can also be defined *locally* using ``where`` clauses. For
.. example, to define a function which reverses a list, we can use an
.. auxiliary function which accumulates the new, reversed list, and which
.. does not need to be visible globally:

函数也可通过 ``where`` 从句来 **局部** 地定义。例如，要定义用来反转列表的函数，
我们可以使用辅助函数来累加新的，反转后的列表，并且它无需全局可见：

.. code-block:: idris

    reverse : List a -> List a
    reverse xs = revAcc [] xs where
      revAcc : List a -> List a -> List a
      revAcc acc [] = acc
      revAcc acc (x :: xs) = revAcc (x :: acc) xs

.. Indentation is significant — functions in the ``where`` block must be
.. indented further than the outer function.

缩进是十分重要的，``where`` 块中函数的缩进层次必须比外层函数更深。

.. .. note:: Scope

..     Any names which are visible in the outer scope are also visible in
..     the ``where`` clause (unless they have been redefined, such as ``xs``
..     here). A name which appears only in the type will be in scope in the
..     ``where`` clause if it is a *parameter* to one of the types, i.e. it
..     is fixed across the entire structure.

.. note:: 作用域

    任何外部作用域中可见，且没有被重新被定义过的名字，在 ``where`` 从句中也可见
    （这里的 ``xs`` 被重新定义了）。若某个名字是某个类型的
    **形参（Parameter）**，那么仅当它在类型中出现时才会在 ``where``
    从句的作用域中，即，它在整体结构中是固定不变的。

.. As well as functions, ``where`` blocks can include local data
.. declarations, such as the following where ``MyLT`` is not accessible
.. outside the definition of ``foo``:

除函数外，``where`` 块中也可包含局部数据声明，以下代码中的的 ``MyLT``
就无法在 ``foo`` 的定义之外访问。

.. code-block:: idris

    foo : Int -> Int
    foo x = case isLT of
                Yes => x*2
                No => x*4
        where
           data MyLT = Yes | No

           isLT : MyLT
           isLT = if x < 20 then Yes else No

.. In general, functions defined in a ``where`` clause need a type
.. declaration just like any top level function. However, the type
.. declaration for a function ``f`` *can* be omitted if:

.. - ``f`` appears in the right hand side of the top level definition

.. - The type of ``f`` can be completely determined from its first application

.. So, for example, the following definitions are legal:

通常，``where`` 从句中定义的函数和其它顶层函数一样，都需要类型声明。
然而，函数 ``f`` 的类型声明可在以下情况中省略：

- ``f`` 出现在顶层定义的右边

- ``f`` 的类型完全可以通过其首次应用来确定

因此，举例来说，以下定义是合法的：

.. code-block:: idris

    even : Nat -> Bool
    even Z = True
    even (S k) = odd k where
      odd Z = False
      odd (S k) = even k

    test : List Nat
    test = [c (S 1), c Z, d (S Z)]
      where c x = 42 + x
            d y = c (y + 1 + z y)
                  where z w = y + w

.. _sect-holes:

坑
--

.. Holes
.. -----

.. Idris programs can contain *holes* which stand for incomplete parts of
.. programs. For example, we could leave a hole for the greeting in our
.. "Hello world" program:

Idris 程序中可以挖 **坑（Hole）** 来表示未完成的部分。例如，我们可以在「Hello world」
程序中为问候语 ``greeting`` 挖一个坑：

.. code-block:: idris

    main : IO ()
    main = putStrLn ?greeting

.. The syntax ``?greeting`` introduces a hole, which stands for a part of
.. a program which is not yet written. This is a valid Idris program, and you
.. can check the type of ``greeting``:

语法 ``?greeting`` 挖了个坑，它表示程序中尚未写完的部分。这是个有效的 Idris
程序，你可以检查 ``greeting`` 的类型：

::

    *Hello> :t greeting
    --------------------------------------
    greeting : String

.. Checking the type of a hole also shows the types of any variables in scope.
.. For example, given an incomplete definition of ``even``:

检查坑的类型也会显示作用域中所有变量的类型。例如，给定一个未完成的 ``even`` 定义：

.. code-block:: idris

    even : Nat -> Bool
    even Z = True
    even (S k) = ?even_rhs

.. We can check the type of ``even_rhs`` and see the expected return type,
.. and the type of the variable ``k``:

我们可以检查 ``even_rhs`` 的类型，查看期望的返回类型，以及变量 ``k`` 的类型：

::

    *Even> :t even_rhs
      k : Nat
    --------------------------------------
    even_rhs : Bool

.. Holes are useful because they help us write functions *incrementally*.
.. Rather than writing an entire function in one go, we can leave some parts
.. unwritten and use Idris to tell us what is necessary to complete the
.. definition.

坑非常有用，因为它能帮助我们 **逐步地** 编写函数。我们无需一次写完整个函数，
而是留下一些尚未编写的部分，让 Idris 告诉我们如何完成其定义。

.. hint::

    lhs（left hand side） 与 rhs（right hand side）分别表示等式中等号的左边和右边，
    即左式和右式。

依赖类型
========

.. Dependent Types
.. ===============

.. _sect-fctypes:

一等类型
--------

.. First Class Types
.. -----------------

.. In Idris, types are first class, meaning that they can be computed and
.. manipulated (and passed to functions) just like any other language construct.
.. For example, we could write a function which computes a type:

在 Idris 中，类型是一等（First-Class）的，即它们可以像其它的语言构造那样被计算和操作
（以及传给函数）。例如，我们可以编写一个用来计算类型的函数：

.. code-block:: idris

    isSingleton : Bool -> Type
    isSingleton True = Nat
    isSingleton False = List Nat

.. This function calculates the appropriate type from a ``Bool`` which flags
.. whether the type should be a singleton or not. We can use this function
.. to calculate a type anywhere that a type can be used. For example, it
.. can be used to calculate a return type:

该函数可从一个 ``Bool`` 值计算出适当的类型，布尔值表示其类型是否为一个单例（Singleton）。
我们可以在任何能够使用类型的地方用该函数计算出一个类型。例如，它可用于计算返回类型：

.. code-block:: idris

    mkSingle : (x : Bool) -> isSingleton x
    mkSingle True = 0
    mkSingle False = []

.. Or it can be used to have varying input types. The following function
.. calculates either the sum of a list of ``Nat``, or returns the given
.. ``Nat``, depending on whether the singleton flag is true:

它也可拥有不同的输入类型。以下函数能够计算 ``Nat`` 列表之和，或者返回给定的
``Nat``，这取决于单例标记 ``single`` 是否为 ``True``：

.. code-block:: idris

    sum : (single : Bool) -> isSingleton single -> Nat
    sum True x = x
    sum False [] = 0
    sum False (x :: xs) = x + sum False xs

向量
----

.. Vectors
.. -------
.. A standard example of a dependent data type is the type of “lists with
.. length”, conventionally called vectors in the dependent type
.. literature. They are available as part of the Idris library, by
.. importing ``Data.Vect``, or we can declare them as follows:

依赖类型的一个范例就是「带长度的列表」类型，在依赖类型的文献中，
它通常被称作向量（Vector）。向量作为 Idris 库的一部分，可通过导入 ``Data.Vect``
来使用，当然我们也自己声明它：

.. code-block:: idris

    data Vect : Nat -> Type -> Type where
       Nil  : Vect Z a
       (::) : a -> Vect k a -> Vect (S k) a

.. Note that we have used the same constructor names as for ``List``.
.. Ad-hoc name overloading such as this is accepted by Idris,
.. provided that the names are declared in different namespaces (in
.. practice, normally in different modules). Ambiguous constructor names
.. can normally be resolved from context.

注意我们使用了与 ``List`` 相同的构造器名。只要名字声明在不同的命名空间内
（在实践中，通常在不同的模块内），Idris 就能接受像这样的特设（ad-hoc）名重载。
有歧义的构造器名称通常可根据上下文来解决。

.. This declares a family of types, and so the form of the declaration is
.. rather different from the simple type declarations above. We
.. explicitly state the type of the type constructor ``Vect`` — it takes
.. a ``Nat`` and a type as an argument, where ``Type`` stands for the
.. type of types. We say that ``Vect`` is *indexed* over ``Nat`` and
.. *parameterised* by ``Type``. Each constructor targets a different part
.. of the family of types. ``Nil`` can only be used to construct vectors
.. with zero length, and ``::`` to construct vectors with non-zero
.. length. In the type of ``::``, we state explicitly that an element of
.. type ``a`` and a tail of type ``Vect k a`` (i.e., a vector of length
.. ``k``) combine to make a vector of length ``S k``.

这里声明了一个类型族（Type Family），该声明的形式与之前的简单类型声明不太一样。
我们显式地描述了类型构造器 ``Vect`` 的类型，它接受一个 ``Nat``
和一个类型作为参数，其中 ``Type`` 表示类型的类型。我们说 ``Vect``
通过 ``Nat`` 来 **索引**，并被 ``Type`` **参数化** 。
每个构造器会产生该类型家族的不同部分。 ``Nil`` 只能用于构造零长度的向量，
而 ``::`` 用于构造非零长度的向量。在 ``::`` 的类型中，我们显式地指定了一个类型为
``a`` 的元素和一个类型为 ``Vect k a`` 的尾部（Tail）（即长度为 ``k`` 的向量），
二者构成了一个长度为 ``S k`` 的向量。

.. We can define functions on dependent types such as ``Vect`` in the same
.. way as on simple types such as ``List`` and ``Nat`` above, by pattern
.. matching. The type of a function over ``Vect`` will describe what
.. happens to the lengths of the vectors involved. For example, ``++``,
.. defined as follows, appends two ``Vect``:

同 ``List`` 以及 ``Nat`` 这类简单类型一样，我们可以通过模式匹配以同样的方式为
``Vect`` 这样的依赖类型定义函数。
作用于 ``Vect`` 的函数的类型能够描述所涉及向量的长度会如何变化。例如，下面定义的
``++`` 用于连接两个 ``Vect``：

.. code-block:: idris

    (++) : Vect n a -> Vect m a -> Vect (n + m) a
    (++) Nil       ys = ys
    (++) (x :: xs) ys = x :: xs ++ ys

.. The type of ``(++)`` states that the resulting vector’s length will be
.. the sum of the input lengths. If we get the definition wrong in such a
.. way that this does not hold, Idris will not accept the definition.
.. For example:

``(++)`` 的类型描述了结果向量的长度必须为输入向量的长度之和。
如果我们以某种方式给出了错误的定义使其不成立，那么 Idris 就不会接受该定义。
例如：

.. code-block:: idris

    (++) : Vect n a -> Vect m a -> Vect (n + m) a
    (++) Nil       ys = ys
    (++) (x :: xs) ys = x :: xs ++ xs -- 有误

.. When run through the Idris type checker, this results in the
.. following:

在经由 Idris 类型检查器检查时，它会给出以下结果：

::

    $ idris VBroken.idr --check
    VBroken.idr:9:23-25:
    When checking right hand side of Vect.++ with expected type
            Vect (S k + m) a

    When checking an application of constructor Vect.:::
            Type mismatch between
                    Vect (k + k) a (Type of xs ++ xs)
            and
                    Vect (plus k m) a (Expected type)

            Specifically:
                    Type mismatch between
                            plus k k
                    and
                            plus k m


.. This error message suggests that there is a length mismatch between
.. two vectors — we needed a vector of length ``k + m``, but provided a
.. vector of length ``k + k``.

该错误信息指出两个向量的长度不匹配：我们需要一个长度为 ``k + m`` 的向量，
而你提供了一个长度为 ``k + k`` 的向量。

有限集
------

.. The Finite Sets
.. ---------------

.. Finite sets, as the name suggests, are sets with a finite number of
.. elements. They are available as part of the Idris library, by
.. importing ``Data.Fin``, or can be declared as follows:

有限集，顾名思义，即元素有限的集合。它作为 Idris 库的一部分，可通过导入
``Data.Fin`` 来使用，当然也可以像下面这样声明它：

.. code-block:: idris

    data Fin : Nat -> Type where
       FZ : Fin (S k)
       FS : Fin k -> Fin (S k)

.. From the signature,  we can see that this is a type constructor that takes a ``Nat``, and produces a type.
.. So this is not a set in the sense of a collection that is a container of objects,
.. rather it is the canonical set of unnamed elements, as in "the set of 5 elements," for example.
.. Effectively, it is a type that captures integers that fall into the range of zero to ``(n - 1)`` where
.. ``n`` is the argument used to instantiate the ``Fin`` type.
.. For example, ``Fin 5`` can be thought of as the type of integers between 0 and 4.

从它的签名中，我们可以看出该类型构造器接受一个 ``Nat``，然后产生一个 **类型** 。
因此，它不是一个「对象的容器」意义上的集合，而是个拥有无名元素的一般集合。举例来说，
就是「存在一个包含五个元素的集合」的那种集合。实际上，它是一个捕获了所有落入零至
``(n - 1)`` 范围内的整数的类型，其中 ``n`` 是用于实例化 ``Fin`` 类型的参数。例如，
``Fin 5`` 可被视作从 0 到 4 之间的整数的类型。

.. Let us look at the constructors in greater detail.

我们来仔细地观察一下它的构造器。

.. ``FZ`` is the zeroth element of a finite set with ``S k`` elements;
.. ``FS n`` is the ``n+1``\ th element of a finite set with ``S k``
.. elements. ``Fin`` is indexed by a ``Nat``, which represents the number
.. of elements in the set. Since we can’t construct an element of an
.. empty set, neither constructor targets ``Fin Z``.

对于拥有 ``S k`` 个元素的有限集来说，``FZ`` 是它的第零个元素，
``FS n`` 则是它的第 ``n+1`` 个元素。 ``Fin`` 通过 ``Nat`` 来索引，
它表示该集合中元素的个数。由于我们无法构造出属于空集的元素，因此也就无法构造出
``Fin Z``。

.. As mentioned above, a useful application of the ``Fin`` family is to
.. represent bounded natural numbers. Since the first ``n`` natural
.. numbers form a finite set of ``n`` elements, we can treat ``Fin n`` as
.. the set of integers greater than or equal to zero and less than ``n``.

如之前提到的， ``Fin`` 家族的用途之一在于表示有界的自然数集。由于前 ``n``
个自然数构成了一个含有 ``n`` 个元素的有限集，我们可以将 ``Fin n``
视作大于等于零且小于 ``n`` 的整数集。

.. For example, the following function which looks up an element in a
.. ``Vect``, by a bounded index given as a ``Fin n``, is defined in the
.. prelude:

例如，下面的函数根据给定的有界索引 ``Fin n`` 找出 ``Vect`` 中的元素，
它在 Prelude 中定义为：

.. code-block:: idris

    index : Fin n -> Vect n a -> a
    index FZ     (x :: xs) = x
    index (FS k) (x :: xs) = index k xs

.. This function looks up a value at a given location in a vector. The
.. location is bounded by the length of the vector (``n`` in each case),
.. so there is no need for a run-time bounds check. The type checker
.. guarantees that the location is no larger than the length of the
.. vector, and of course no less than zero.

该函数从一个向量中找出给定位置的值。位置的边界由该向量的长度所界定
（每种情况下都是 ``n`` ），因此无需在运行时进行边界检查。类型检查器保证了
位置不会大于该向量的长度，当然也不会小于零。

.. Note also that there is no case for ``Nil`` here. This is because it
.. is impossible. Since there is no element of ``Fin Z``, and the
.. location is a ``Fin n``, then ``n`` can not be ``Z``. As a result,
.. attempting to look up an element in an empty vector would give a
.. compile time type error, since it would force ``n`` to be ``Z``.

注意这里也没有 ``Nil`` 的情况，因为这种情况不可能存在。
由于没有类型为 ``Fin Z`` 且位置为 ``Fin n`` 的元素，因此 ``n`` 不可能是 ``Z``。
因此，如果你试图在一个空向量中查找元素，就会得到一个编译时的类型错误，
因为这样做会强行令 ``n`` 为 ``Z``。

隐式参数
--------

.. Implicit Arguments
.. ------------------

.. Let us take a closer look at the type of ``index``:

我们再仔细观察一下 ``index`` 的类型：

.. code-block:: idris

    index : Fin n -> Vect n a -> a

.. It takes two arguments, an element of the finite set of ``n`` elements,
.. and a vector with ``n`` elements of type ``a``. But there are also two
.. names, ``n`` and ``a``, which are not declared explicitly. These are
.. *implicit* arguments to ``index``. We could also write the type of
.. ``index`` as:

它接受两个参数：一个类型为 ``n`` 元素有限集的元素，以及一个类型为 ``a`` 的 ``n``
元素向量。不过这里还有两个名字：``n`` 和 ``a``，它们未被显式地声明。``index``
包含了 **隐式** 参数。我们也可以将 ``index`` 的类型写作：

.. code-block:: idris

    index : {a:Type} -> {n:Nat} -> Fin n -> Vect n a -> a

.. Implicit arguments, given in braces ``{}`` in the type declaration,
.. are not given in applications of ``index``; their values can be
.. inferred from the types of the ``Fin n`` and ``Vect n a``
.. arguments. Any name beginning with a lower case letter which appears
.. as a parameter or index in a
.. type declaration, which is not applied to any arguments, will
.. *always* be automatically
.. bound as an implicit argument. Implicit arguments can still be given
.. explicitly in applications, using ``{a=value}`` and ``{n=value}``, for
.. example:

隐式参数在类型声明的大括号 ``{}`` 中给定，它们并没有在应用 ``index`` 时给出，
因为它们的值可以从 ``Fin n`` 和 ``Vect n a`` 的参数类型中推导出来。
任何以小写字母开头，在类型声明中作为形参和索引出现的名字都不会应用到任何实参上，
它们 **总是** 会作为隐式参数被自动绑定。隐式参数仍然可以在应用时通过 ``{a=value}``
和 ``{n=value}`` 来显式地给定，例如：

.. code-block:: idris

    index {a=Int} {n=2} FZ (2 :: 3 :: Nil)

.. In fact, any argument, implicit or explicit, may be given a name. We
.. could have declared the type of ``index`` as:

实际上，无论是隐式还是显式，任何参数都可以给定一个名称。我们可以将 ``index``
声明成这样：

.. code-block:: idris

    index : (i:Fin n) -> (xs:Vect n a) -> a

.. It is a matter of taste whether you want to do this — sometimes it can
.. help document a function by making the purpose of an argument more
.. clear.

写不写它纯属偏好问题，不过有时它能让参数更加明确，有助于函数文档的记录。

.. Furthermore, ``{}`` can be used to pattern match on the left hand side, i.e.
.. ``{var = pat}`` gets an implicit variable and attempts to pattern match on “pat”;
.. For example:

此外， ``{}`` 在等号左边时可用作模式匹配，即 ``{var = pat}`` 获取一个隐式变量，
并试图对「pat」进行模式匹配。例如：

.. code-block:: idris

    isEmpty : Vect n a -> Bool
    isEmpty {n = Z} _   = True
    isEmpty {n = S k} _ = False

「``using``」记法
-----------------

.. “``using``” notation
.. --------------------

.. Sometimes it is useful to provide types of implicit arguments,
.. particularly where there is a dependency ordering, or where the
.. implicit arguments themselves have dependencies. For example, we may
.. wish to state the types of the implicit arguments in the following
.. definition, which defines a predicate on vectors (this is also defined
.. in ``Data.Vect``, under the name ``Elem``):

有时为隐式参数提供类型会十分有用，特别是存在依赖顺序，或隐式参数本身含有依赖的情况下。
例如，我们可能希望在以下定义中指明隐式参数的类型，它为向量定义了前提（它也在
``Data.Vect`` 的 ``Elem`` 下定义）：

.. code-block:: idris

    data IsElem : a -> Vect n a -> Type where
       Here :  {x:a} ->   {xs:Vect n a} -> IsElem x (x :: xs)
       There : {x,y:a} -> {xs:Vect n a} -> IsElem x xs -> IsElem x (y :: xs)

.. An instance of ``IsElem x xs`` states that ``x`` is an element of
.. ``xs``.  We can construct such a predicate if the required element is
.. ``Here``, at the head of the vector, or ``There``, in the tail of the
.. vector. For example:

``IsElem x xs`` 的实例描述了 ``x`` 是 ``xs`` 中的一个元素。我们可以构造这样的谓词：
若所需的元素在向量的头部时为 ``Here``，在向量的尾部中时则为 ``There``。例如：

.. code-block:: idris

    testVec : Vect 4 Int
    testVec = 3 :: 4 :: 5 :: 6 :: Nil

    inVect : IsElem 5 Main.testVec
    inVect = There (There Here)

.. .. important:: Implicit Arguments and Scope

..     Within the type signature the typechecker will treat all variables
..     that start with an lowercase letter **and** are not applied to
..     something else as an implicit variable. To get the above code
..     example to compile you will need to provide a qualified name for
..     ``testVec``. In the example above, we have assumed that the code
..     lives within the ``Main`` module.

.. important:: 隐式参数与作用域

    在类型签名中，类型检查器会将所有以小写字母开头 **并且** 没有应用到别的东西上的变量
    视作隐式变量。要让上面的代码示例能够编译，你需要为 ``testVec`` 提供一个限定名。
    在前面的例子中，我们假设该代码处于 ``Main`` 模块内。

.. If the same implicit arguments are being used a lot, it can make a
.. definition difficult to read. To avoid this problem, a ``using`` block
.. gives the types and ordering of any implicit arguments which can
.. appear within the block:

如果大量使用相同的隐式参数，就会导致定义难以阅读。为避免此问题，可使用 ``using``
块来为任何在块中出现的隐式参数指定类型和顺序：

.. code-block:: idris

    using (x:a, y:a, xs:Vect n a)
      data IsElem : a -> Vect n a -> Type where
         Here  : IsElem x (x :: xs)
         There : IsElem x xs -> IsElem x (y :: xs)


注：声明顺序与 ``mutual`` 互用块
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Note: Declaration Order and ``mutual`` blocks
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. In general, functions and data types must be defined before use, since
.. dependent types allow functions to appear as part of types, and type
.. checking can rely on how particular functions are defined (though this
.. is only true of total functions; see Section :ref:`sect-totality`).
.. However, this restriction can be relaxed by using a ``mutual`` block,
.. which allows data types and functions to be defined simultaneously:

通常，函数与数据类型必须在使用前定义，因为依赖类型允许函数作为类型的一部分出现，
而类型检查会依赖于特定的函数如何定义（尽管这只对全函数成立，见 :ref:`sect-totality`）。
然而，此限制可通过使用 ``mutual`` 互用块来放宽，它允许数据类型和函数同时定义：

.. code-block:: idris

    mutual
      even : Nat -> Bool
      even Z = True
      even (S k) = odd k

      odd : Nat -> Bool
      odd Z = False
      odd (S k) = even k

.. In a ``mutual`` block, first all of the type declarations are added,
.. then the function bodies. As a result, none of the function types can
.. depend on the reduction behaviour of any of the functions in the
.. block.

在 ``mutual`` 块中，首先所有的类型声明会被添加，然后是函数体。
因此，没有一个函数类型可以依赖于块中任何函数的归约行为。

I/O
===

.. Computer programs are of little use if they do not interact with the
.. user or the system in some way. The difficulty in a pure language such
.. as Idris — that is, a language where expressions do not have
.. side-effects — is that I/O is inherently side-effecting. Therefore in
.. Idris, such interactions are encapsulated in the type ``IO``:

.. .. code-block:: idris

..     data IO a -- IO operation returning a value of type a

如果计算机程序不能通过某种方式与用户或系统进行交互，那么它基本上没什么用。在 Idris
这样纯粹（Pure）的语言中，表达式没有副作用（Side-Effect）。而 I/O
的难点在于它本质上是带有副作用的。因此在 Idris 中，这样的交互被封装在 ``IO`` 类型中：

.. code-block:: idris

    data IO a -- IO 操作返回一个类型为 a 的值

.. We’ll leave the definition of ``IO`` abstract, but effectively it
.. describes what the I/O operations to be executed are, rather than how
.. to execute them. The resulting operations are executed externally, by
.. the run-time system. We’ve already seen one IO program:

我们先给出 ``IO`` 抽象的定义，它本质上描述了被执行的 I/O 操作是什么，
而非如何去执行它们。最终操作则由运行时系统在外部执行。我们已经见过一个带 IO
的程序了：

.. code-block:: idris

    main : IO ()
    main = putStrLn "Hello world"

.. The type of ``putStrLn`` explains that it takes a string, and returns
.. an element of the unit type ``()`` via an I/O action. There is a
.. variant ``putStr`` which outputs a string without a newline:

``putStrLn`` 的类型描述了它接受一个字符串，然后通过 I/O 活动返回一个单元类型
``()`` 的元素。它还有一个变体 ``putStr`` 用来输出字符串但不换行。

.. code-block:: idris

    putStrLn : String -> IO ()
    putStr   : String -> IO ()

.. We can also read strings from user input:

我们可以从用户的输入中读取字符串：

.. code-block:: idris

    getLine : IO String

.. A number of other I/O operations are defined in the prelude, for
.. example for reading and writing files, including:

Prelude 中定义了很多 I/O 操作，例如为了读写文件，需要包含：

.. code-block:: idris

    data File -- abstract
    data Mode = Read | Write | ReadWrite

    openFile : (f : String) -> (m : Mode) -> IO (Either FileError File)
    closeFile : File -> IO ()

    fGetLine : (h : File) -> IO (Either FileError String)
    fPutStr : (h : File) -> (str : String) -> IO (Either FileError ())
    fEOF : File -> IO Bool

.. Note that several of these return ``Either``, since they may fail.

注意其中几个函数会返回 ``Either`` ，因为它们可能会失败。

.. _sect-do:

「``do``」记法
==============

.. “``do``” notation
.. =================

.. I/O programs will typically need to sequence actions, feeding the
.. output of one computation into the input of the next. ``IO`` is an
.. abstract type, however, so we can’t access the result of a computation
.. directly. Instead, we sequence operations with ``do`` notation:

I/O 程序通常需要串连起多个活动，将一个计算的输出送入下一个计算的输入中。
然而，``IO`` 是一个抽象类型，因此我们无法直接访问一个计算的结果。
因此，我们用 ``do`` 记法来串连起操作：

.. code-block:: idris

    greet : IO ()
    greet = do putStr "What is your name? "
               name <- getLine
               putStrLn ("Hello " ++ name)

.. The syntax ``x <- iovalue`` executes the I/O operation ``iovalue``, of
.. type ``IO a``, and puts the result, of type ``a`` into the variable
.. ``x``. In this case, ``getLine`` returns an ``IO String``, so ``name``
.. has type ``String``. Indentation is significant — each statement in
.. the do block must begin in the same column. The ``pure`` operation
.. allows us to inject a value directly into an IO operation:

语法 ``x <- iovalue`` 执行 ``IO a`` 类型的 I/O 操作 ``iovalue``，然后将类型为
``a`` 的结果送入变量 ``x`` 中。在这种情况下，``getLine`` 会返回一个
``IO String``，因此 ``name`` 的类型为 ``String``。缩进十分重要：
do 语句块中的每个语句都必须从同一列开始。``pure`` 操作允许我们将值直接注入到
IO 操作中：

.. code-block:: idris

    pure : a -> IO a

.. As we will see later, ``do`` notation is more general than this, and
.. can be overloaded.

后面我们会看到，``do`` 记法比这里的展示更加通用，并且可以被重载。

.. _sect-lazy:

惰性
====

.. Laziness
.. ========

.. Normally, arguments to functions are evaluated before the function
.. itself (that is, Idris uses *eager* evaluation). However, this is
.. not always the best approach. Consider the following function:

通常，函数的参数会在函数被调用前求值（也就是说，Idris 采用了 **及早（Eager）**
求值策略）。然而，这并不总是最佳的方式。考虑以下函数：

.. code-block:: idris

    ifThenElse : Bool -> a -> a -> a
    ifThenElse True  t e = t
    ifThenElse False t e = e

.. This function uses one of the ``t`` or ``e`` arguments, but not both
.. (in fact, this is used to implement the ``if...then...else`` construct
.. as we will see later). We would prefer if *only* the argument which was
.. used was evaluated. To achieve this, Idris provides a ``Lazy``
.. data type, which allows evaluation to be suspended:

该函数会使用参数 ``t`` 或 ``e`` 二者之一，而非二者都用（我们之后会看到其实它被用于实现
``if...then...else`` 构造）。我们更希望 **只有** 用到的参数才被求值。为此，
Idris 提供了 ``Lazy`` 数据类型，它允许暂缓求值：

.. code-block:: idris

    data Lazy : Type -> Type where
         Delay : (val : a) -> Lazy a

    Force : Lazy a -> a

.. A value of type ``Lazy a`` is unevaluated until it is forced by
.. ``Force``. The Idris type checker knows about the ``Lazy`` type,
.. and inserts conversions where necessary between ``Lazy a`` and ``a``,
.. and vice versa. We can therefore write ``ifThenElse`` as follows,
.. without any explicit use of ``Force`` or ``Delay``:

类型为 ``Lazy a`` 的值只有通过 ``Force`` 强制求值时才会被求值。Idris
类型检查器知道 ``Lazy`` 类型，并会在必要时在 ``Lazy a`` 和 ``a`` 之间插入转换，
反之亦同。因此我们可以将 ``ifThenElse`` 写成下面这样，无需任何 ``Force``
或 ``Delay`` 的显式使用：

.. code-block:: idris

    ifThenElse : Bool -> Lazy a -> Lazy a -> a
    ifThenElse True  t e = t
    ifThenElse False t e = e

余数据类型
==========

.. Codata Types
.. ============

.. Codata types allow us to define infinite data structures by marking recursive
.. arguments as potentially infinite. For
.. a codata type ``T``, each of its constructor arguments of type ``T`` are transformed
.. into an argument of type ``Inf T``. This makes each of the ``T`` arguments
.. lazy, and allows infinite data structures of type ``T`` to be built. One
.. example of a codata type is Stream, which is defined as follows.

我们可以通过余数据类型，将递归参数标记为潜在无穷来定义无穷数据结构。对于一个余数据类型
``T``，其每个构造器中类型为 ``T`` 的参数都会被转换成类型为 ``Inf T`` 的参数。
这会让每个 ``T`` 类型的参数惰性化，使得类型为 ``T`` 的无穷数据结构得以构建。
余数据类型的一个例子为 ``Stream``，其定义如下：

.. code-block:: idris

    codata Stream : Type -> Type where
      (::) : (e : a) -> Stream a -> Stream a

.. This gets translated into the following by the compiler.

它会被编译器翻译成下面这样：

.. code-block:: idris

    data Stream : Type -> Type where
      (::) : (e : a) -> Inf (Stream a) -> Stream a

.. The following is an example of how the codata type ``Stream`` can be used to
.. form an infinite data structure. In this case we are creating an infinite stream
.. of ones.

以下是如何用余数据类型 ``Stream`` 来构建无穷数据结构的例子。
在这里我们创建了一个 1 的无穷流：

.. code-block:: idris

    ones : Stream Nat
    ones = 1 :: ones

.. It is important to note that codata does not allow the creation of infinite
.. mutually recursive data structures. For example the following will create an
.. infinite loop and cause a stack overflow.

要重点注意：余数据类型不允许创建互用的无穷递归数据结构。
例如，以下代码会创建一个无穷循环并导致栈溢出：

.. code-block:: idris

    mutual
      codata Blue a = B a (Red a)
      codata Red a = R a (Blue a)

    mutual
      blue : Blue Nat
      blue = B 1 red

      red : Red Nat
      red = R 1 blue

    mutual
      findB : (a -> Bool) -> Blue a -> a
      findB f (B x r) = if f x then x else findR f r

      findR : (a -> Bool) -> Red a -> a
      findR f (R x b) = if f x then x else findB f b

    main : IO ()
    main = do printLn $ findB (== 1) blue

.. To fix this we must add explicit ``Inf`` declarations to the constructor
.. parameter types, since codata will not add it to constructor parameters of a
.. **different** type from the one being defined. For example, the following
.. outputs ``1``.

为了修复它，我们必须为构造器参数的类型显式地加上 ``Inf`` 声明，因为余数据类型
不会将它添加到和正在定义的构造器类型 **不同** 的构造器参数上。例如，以下程序输出「1」。

.. code-block:: idris

    mutual
      data Blue : Type -> Type where
       B : a -> Inf (Red a) -> Blue a

      data Red : Type -> Type where
       R : a -> Inf (Blue a) -> Red a

    mutual
      blue : Blue Nat
      blue = B 1 red

      red : Red Nat
      red = R 1 blue

    mutual
      findB : (a -> Bool) -> Blue a -> a
      findB f (B x r) = if f x then x else findR f r

      findR : (a -> Bool) -> Red a -> a
      findR f (R x b) = if f x then x else findB f b

    main : IO ()
    main = do printLn $ findB (== 1) blue

.. hint:: 「归纳数据类型」和「余归纳数据类型」

    余数据类型（Codata Type）的全称为余归纳数据类型（Coinductive Data Type），
    归纳数据类型和余归纳数据类型是对偶的关系。从语义上看，
    Inductive Type 描述了如何从更小的 term 构造出更大的 term；而
    Coinductive Type 则描述了如何从更大的 term 分解成更小的 term。
    二者即为塔斯基不动点定理中的最大不动点（对应余归纳）和最小不动点（对应归纳）。
    参考自 `Belleve <https://www.zhihu.com/question/60184579/answer/255291675>`_
    的回答。


常用数据类型
============

.. Useful Data Types
.. =================

.. Idris includes a number of useful data types and library functions
.. (see the ``libs/`` directory in the distribution, and the
.. `documentation <https://www.idris-lang.org/documentation/>`_). This section
.. describes a few of these. The functions described here are imported
.. automatically by every Idris program, as part of ``Prelude.idr``.

Idris 包含了很多常用的数据类型和库函数（见发行版中的 ``libs/`` 目录及
`文档 <https://www.idris-lang.org/documentation/>`_ ）。本节描述了其中的一部分。
作为 ``Prelude.idr`` 的一部分，下面描述的函数都会被每个 Idris 程序自动导入，

.. ``List`` and ``Vect``

``List`` 与 ``Vect``
---------------------

.. We have already seen the ``List`` and ``Vect`` data types:

我们已经见过 ``List`` 和 ``Vect`` 数据类型了：

.. code-block:: idris

    data List a = Nil | (::) a (List a)

    data Vect : Nat -> Type -> Type where
       Nil  : Vect Z a
       (::) : a -> Vect k a -> Vect (S k) a

.. Note that the constructor names are the same for each — constructor
.. names (in fact, names in general) can be overloaded, provided that
.. they are declared in different namespaces (see Section
.. :ref:`sect-namespaces`), and will typically be resolved according to
.. their type. As syntactic sugar, any type with the constructor names
.. ``Nil`` and ``::`` can be written in list form. For example:

注意它们的构造器名称是相同的：只要构造器名称在不同的命名空间中声明，
它们就可以被重载（其实一般的名字都可以），并且通常会根据其类型来确定。
作为一种语法糖，任何带有 ``Nil`` 和 ``::`` 构造其名的类型都可被写成列表的形式。
例如：

.. -  ``[]`` means ``Nil``

.. -  ``[1,2,3]`` means ``1 :: 2 :: 3 :: Nil``

-  ``[]`` 表示 ``Nil``

-  ``[1,2,3]`` 表示 ``1 :: 2 :: 3 :: Nil``

.. The library also defines a number of functions for manipulating these
.. types. ``map`` is overloaded both for ``List`` and ``Vect`` and
.. applies a function to every element of the list or vector.

库中还定义了一些用于操作这些类型的函数。 ``map`` 对 ``List`` 和 ``Vect``
都进行了重载，它将一个函数应用到列表或向量中的每个元素上。

.. code-block:: idris

    map : (a -> b) -> List a -> List b
    map f []        = []
    map f (x :: xs) = f x :: map f xs

    map : (a -> b) -> Vect n a -> Vect n b
    map f []        = []
    map f (x :: xs) = f x :: map f xs

.. For example, given the following vector of integers, and a function to
.. double an integer:

例如，给定以下整数向量，以及一个将整数乘以 2 的函数：

.. code-block:: idris

    intVec : Vect 5 Int
    intVec = [1, 2, 3, 4, 5]

    double : Int -> Int
    double x = x * 2

.. the function ``map`` can be used as follows to double every element in
.. the vector:

函数 ``map`` 可像下面这样将该向量中的每个元素乘以二：

::

    *UsefulTypes> show (map double intVec)
    "[2, 4, 6, 8, 10]" : String

.. For more details of the functions available on ``List`` and
.. ``Vect``, look in the library files:

更多可用于 ``List`` 和 ``Vect`` 的函数的详情请参阅以下库文件：

-  ``libs/prelude/Prelude/List.idr``

-  ``libs/base/Data/List.idr``

-  ``libs/base/Data/Vect.idr``

-  ``libs/base/Data/VectType.idr``

.. Functions include filtering, appending, reversing, and so on.

其中包括过滤、追加、反转等函数。


题外话：匿名函数与操作符段
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Aside: Anonymous functions and operator sections
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. There are actually neater ways to write the above expression. One way
.. would be to use an anonymous function:

上面的表达式其实还有更加利落的写法。其中一种就是使用匿名函数（Anonymous Function）：

::

    *UsefulTypes> show (map (\x => x * 2) intVec)
    "[2, 4, 6, 8, 10]" : String

.. The notation ``\x => val`` constructs an anonymous function which takes
.. one argument, ``x`` and returns the expression ``val``. Anonymous
.. functions may take several arguments, separated by commas,
.. e.g. ``\x, y, z => val``. Arguments may also be given explicit types,
.. e.g. ``\x : Int => x * 2``, and can pattern match,
.. e.g. ``\(x, y) => x + y``. We could also use an operator section:

记法 ``\x => val`` 构造了一个匿名函数，它接受一个参数 ``x`` 并返回表达式 ``val``。
匿名函数可以接受多个参数，它们以逗号分隔，如 ``\x, y, z => val``。
参数也可以显式地给定类型，如 ``\x : Int => x * 2``，也可使用模式匹配，如
``\(x, y) => x + y``。

::

    *UsefulTypes> show (map (* 2) intVec)
    "[2, 4, 6, 8, 10]" : String

.. ``(*2)`` is shorthand for a function which multiplies a number
.. by 2. It expands to ``\x => x * 2``. Similarly, ``(2*)`` would expand
.. to ``\x => 2 * x``.

``(*2)`` 是将数字乘以 2 的函数的简写，它会被展开为 ``\x => x * 2``。
同样，``(2*)`` 会被展开为 ``\x => 2 * x``。

.. hint:: 匿名函数在函数式编程中又称为 λ-表达式（Lambda Expression）。

Maybe
-----

.. ``Maybe`` describes an optional value. Either there is a value of the
.. given type, or there isn’t:

``Maybe`` 描述了可选值，表示给定类型的值是否存在：

.. code-block:: idris

    data Maybe a = Just a | Nothing

.. ``Maybe`` is one way of giving a type to an operation that may
.. fail. For example, looking something up in a ``List`` (rather than a
.. vector) may result in an out of bounds error:

``Maybe`` 是一种为可能失败的操作赋予类型的方式。例如，在 ``List``
（而非向量）中查找时可能会产生越界错误：

.. code-block:: idris

    list_lookup : Nat -> List a -> Maybe a
    list_lookup _     Nil         = Nothing
    list_lookup Z     (x :: xs) = Just x
    list_lookup (S k) (x :: xs) = list_lookup k xs

.. The ``maybe`` function is used to process values of type ``Maybe``,
.. either by applying a function to the value, if there is one, or by
.. providing a default value:

``maybe`` 函数用于处理 ``Maybe`` 类型的值，如果值存在就对其应用一个函数，
否则提供一个默认值：

.. code-block:: idris

    maybe : Lazy b -> Lazy (a -> b) -> Maybe a -> b

.. Note that the types of the first two arguments are wrapped in
.. ``Lazy``. Since only one of the two arguments will actually be used,
.. we mark them as ``Lazy`` in case they are large expressions where it
.. would be wasteful to compute and then discard them.

注意前两个参数的类型被封装在 ``Lazy`` 内。由于二者只有其一会被使用，
而计算完大型表达式之后就丢弃会造成浪费，因此我们将它们标记为 ``Lazy``。

元组
----

.. Tuples
.. ------

.. Values can be paired with the following built-in data type:

值可以通过以下内建的数据类型构成序对（Pair）：

.. code-block:: idris

    data Pair a b = MkPair a b

.. As syntactic sugar, we can write ``(a, b)`` which, according to
.. context, means either ``Pair a b`` or ``MkPair a b``. Tuples can
.. contain an arbitrary number of values, represented as nested pairs:

序对的语法糖可以写成 ``(a, b)``，根据上下文，其意思为 ``Pair a b``
或 ``MkPair a b``。元组（Tuple）可包含任意个数的值，它通过嵌套的序对来表示：

.. code-block:: idris

    fred : (String, Int)
    fred = ("Fred", 42)

    jim : (String, Int, String)
    jim = ("Jim", 25, "Cambridge")

::

    *UsefulTypes> fst jim
    "Jim" : String
    *UsefulTypes> snd jim
    (25, "Cambridge") : (Int, String)
    *UsefulTypes> jim == ("Jim", (25, "Cambridge"))
    True : Bool

依赖序对
--------

.. Dependent Pairs
.. ---------------

.. Dependent pairs allow the type of the second element of a pair to depend
.. on the value of the first element:

依赖序对允许序对第二个元素的类型依赖于第一个元素的值。

.. code-block:: idris

    data DPair : (a : Type) -> (P : a -> Type) -> Type where
       MkDPair : {P : a -> Type} -> (x : a) -> P x -> DPair a P

.. Again, there is syntactic sugar for this. ``(a : A ** P)`` is the type
.. of a pair of A and P, where the name ``a`` can occur inside ``P``.
.. ``( a ** p )`` constructs a value of this type. For example, we can
.. pair a number with a ``Vect`` of a particular length:

同样，它也有语法糖。``(a : A ** P)`` 表示由 A 和 P 构成的序对的类型，其中名字
``a`` 可出现在 ``P`` 中。``( a ** p )`` 会构造一个该类型的值。例如，
我们可以将一个数和一个特定长度的 ``Vect`` 构造成一个序对：

.. code-block:: idris

    vec : (n : Nat ** Vect n Int)
    vec = (2 ** [3, 4])
.. If you like, you can write it out the long way, the two are precisely
.. equivalent:

如果你喜欢，也可以把它写成较长的形式，二者完全等价：

.. code-block:: idris

    vec : DPair Nat (\n => Vect n Int)
    vec = MkDPair 2 [3, 4]

.. The type checker could of course infer the value of the first element
.. from the length of the vector. We can write an underscore ``_`` in
.. place of values which we expect the type checker to fill in, so the
.. above definition could also be written as:

当然，类型检查器可以根据向量的的长度推断出第一个元素的值。
我们可以在希望类型检查器填写值的地方写一个下划线 ``_``，这样上面的定义也可以写作：

.. code-block:: idris

    vec : (n : Nat ** Vect n Int)
    vec = (_ ** [3, 4])

.. We might also prefer to omit the type of the first element of the
.. pair, since, again, it can be inferred:

有时我们也更倾向于省略该序对第一个元素的类型，同样，它也可以被推断出来：

.. code-block:: idris

    vec : (n ** Vect n Int)
    vec = (_ ** [3, 4])

.. One use for dependent pairs is to return values of dependent types
.. where the index is not necessarily known in advance. For example, if
.. we filter elements out of a ``Vect`` according to some predicate, we
.. will not know in advance what the length of the resulting vector will
.. be:

依赖序对的一个用处就是返回依赖类型的值，其中的索引未必事先知道。例如，
若按照某谓词过滤出 ``Vect`` 中的元素，我们不会事先知道结果向量的长度：

.. code-block:: idris

    filter : (a -> Bool) -> Vect n a -> (p ** Vect p a)

.. If the ``Vect`` is empty, the result is easy:

如果 ``Vect`` 为空，结果很简单：

.. code-block:: idris

    filter p Nil = (_ ** [])

.. In the ``::`` case, we need to inspect the result of a recursive call
.. to ``filter`` to extract the length and the vector from the result. To
.. do this, we use ``with`` notation, which allows pattern matching on
.. intermediate values:

在 ``::`` 的情况下，我们需要检查 ``filter`` 递归调用的结果来提取结果的长度和向量。
为此，我们使用 ``with`` 记法，它允许我们对中间值进行模式匹配：

.. code-block:: idris

    filter p (x :: xs) with (filter p xs)
      | ( _ ** xs' ) = if (p x) then ( _ ** x :: xs' ) else ( _ ** xs' )

.. We will see more on ``with`` notation later.

我们之后会看到 ``with`` 的更多详情。

.. Dependent pairs are sometimes referred to as “Sigma types”.

依赖序对有时被称作「Sigma 类型」。

记录
----

.. Records
.. -------

.. *Records* are data types which collect several values (the record's
.. *fields*) together. Idris provides syntax for defining records and
.. automatically generating field access and update functions. Unlike
.. the syntax used for data structures, records in Idris follow a
.. different syntax to that seen with Haskell. For example, we can
.. represent a person’s name and age in a record:

**记录（Record）** 数据类型将多个值（记录的 **字段（Field）**）收集在一起。
Idris 提供了定义记录的语法，它也会自动生成用于访问和更新字段的函数。
和数据结构的语法不同，Idris 中的记录遵循与 Haskell 中看起来不同的语法。
例如，我们可以将一个人的名字和年龄用记录表示：

.. code-block:: idris

    record Person where
        constructor MkPerson
        firstName, middleName, lastName : String
        age : Int

    fred : Person
    fred = MkPerson "Fred" "Joe" "Bloggs" 30

.. The constructor name is provided using the ``constructor`` keyword,
.. and the *fields* are then given which are in an indented block
.. following the `where` keyword (here, ``firstName``, ``middleName``,
.. ``lastName``, and ``age``). You can declare multiple fields on a
.. single line, provided that they have the same type. The field names
.. can be used to access the field values:

构造器名称由 ``constructor`` 关键字确定，**字段** 在 ``where``
关键字之后的缩进块中给定（此处为 ``firstName``、``middleName``、``lastName``
以及 ``age``）。

::

    *Record> firstName fred
    "Fred" : String
    *Record> age fred
    30 : Int
    *Record> :t firstName
    firstName : Person -> String

.. We can also use the field names to update a record (or, more
.. precisely, produce a copy of the record with the given fields
.. updated):

我们也可以用字段名来更新一个记录（确切来说，会产生一个更新了给定字段的记录的副本）：

.. code-block:: bash

    *Record> record { firstName = "Jim" } fred
    MkPerson "Jim" "Joe" "Bloggs" 30 : Person
    *Record> record { firstName = "Jim", age $= (+ 1) } fred
    MkPerson "Jim" "Joe" "Bloggs" 31 : Person

.. The syntax ``record { field = val, ... }`` generates a function which
.. updates the given fields in a record. ``=`` assigns a new value to a field,
.. and ``$=`` applies a function to update its value.

语法 ``record { field = val, ... }`` 会生成一个更新了记录中给定字段的函数。``=``
为字段赋予新值，而 ``$=`` 通过应用一个函数来更新其值。

.. Each record is defined in its own namespace, which means that field names
.. can be reused in multiple records.

每个记录在它自己的命名空间中定义，这意味着字段名可在多个记录中重用。

.. Records, and fields within records, can have dependent types. Updates
.. are allowed to change the type of a field, provided that the result is
.. well-typed.

记录以及记录中的字段可拥有依赖类型。更新允许更改字段的类型，前提是结果是良类型的。

.. code-block:: idris

    record Class where
        constructor ClassInfo
        students : Vect n Person
        className : String

.. It is safe to update the ``students`` field to a vector of a different
.. length because it will not affect the type of the record:

将 ``students`` 的字段更新为不同长度的向量是安全的，因为它不会影响该记录的类型：

.. code-block:: idris

    addStudent : Person -> Class -> Class
    addStudent p c = record { students = p :: students c } c

::

    *Record> addStudent fred (ClassInfo [] "CS")
    ClassInfo [MkPerson "Fred" "Joe" "Bloggs" 30] "CS" : Class

.. We could also use ``$=`` to define ``addStudent`` more concisely:

我们也可以用 ``$=`` 来更简洁地定义 ``addStudent``：

.. code-block:: idris

    addStudent' : Person -> Class -> Class
    addStudent' p c = record { students $= (p ::) } c

嵌套记录的更新
~~~~~~~~~~~~~~

.. Nested record update
.. ~~~~~~~~~~~~~~~~~~~~

.. Idris also provides a convenient syntax for accessing and updating
.. nested records. For example, if a field is accessible with the
.. expression ``c (b (a x))``, it can be updated using the following
.. syntax:

Idris 也提供了便于访问和更新嵌套记录的语法。例如，若一个字段可以通过表达式
``c (b (a x))`` 访问，那么它就可通过以下语法更新：

.. code-block:: idris

    record { a->b->c = val } x

.. This returns a new record, with the field accessed by the path
.. ``a->b->c`` set to ``val``. The syntax is first class, i.e. ``record {
.. a->b->c = val }`` itself has a function type. Symmetrically, the field
.. can also be accessed with the following syntax:

这会返回一个新的记录，通过路径 ``a->b->c`` 访问的字段会被设置为 ``val``。
该语法是一等的，即 ``record { a->b->c = val }`` 本身拥有一个函数类型。
与此对应，你也可以使用以下语法访问该字段：

.. code-block:: idris

    record { a->b->c } x

.. The ``$=`` notation is also valid for nested record updates.

``$=`` 记法对嵌套记录的更新亦有效。

依赖记录
--------

.. Dependent Records
.. -----------------

.. Records can also be dependent on values. Records have *parameters*, which
.. cannot be updated like the other fields. The parameters appear as arguments
.. to the resulting type, and are written following the record type
.. name. For example, a pair type could be defined as follows:

记录也可依赖于值。记录拥有 **形参**，它无法像其它字段那样更新。
形参作为结果类型的参数出现，写在记录类型名之后。例如，一个序对类型可定义如下：

.. code-block:: idris

    record Prod a b where
        constructor Times
        fst : a
        snd : b

.. Using the ``class`` record from earlier, the size of the class can be
.. restricted using a ``Vect`` and the size included in the type by parameterising
.. the record with the size.  For example:

使用前面的 ``class`` 记录，班级的大小可用 ``Vect`` 及通过 size
参数化该记录的大小来限制其类型。例如：

.. code-block:: idris

    record SizedClass (size : Nat) where
        constructor SizedClassInfo
        students : Vect size Person
        className : String

.. **Note** that it is no longer possible to use the ``addStudent``
.. function from earlier, since that would change the size of the class. A
.. function to add a student must now specify in the type that the
.. size of the class has been increased by one. As the size is specified
.. using natural numbers, the new value can be incremented using the
.. ``S`` constructor:

**注意** 它无法再使用之前的 ``addStudent`` 函数了，因为这会改变班级的大小。
现在用于添加学生的函数必须在类型中指定班级的大小加一。由于其大小用自然数指定，
新的值可使用 ``S`` 构造器递增：

.. code-block:: idris

    addStudent : Person -> SizedClass n -> SizedClass (S n)
    addStudent p c =  SizedClassInfo (p :: students c) (className c)

.. _sect-more-expr:

更多表达式
==========

.. More Expressions
.. ================

``let`` 绑定
------------

.. ``let`` bindings
.. ----------------


.. Intermediate values can be calculated using ``let`` bindings:

中间值可使用 ``let`` 绑定来计算：

.. code-block:: idris

   mirror : List a -> List a
   mirror xs = let xs' = reverse xs in
                   xs ++ xs'

.. We can do simple pattern matching in ``let`` bindings too. For
.. example, we can extract fields from a record as follows, as well as by
.. pattern matching at the top level:

我们也可以在 ``let`` 绑定中进行简单的模式匹配。例如，我们可以按如下方式从记录中提取字段，
也可以通过顶层的模式匹配来提取字段：

.. code-block:: idris

    data Person = MkPerson String Int

    showPerson : Person -> String
    showPerson p = let MkPerson name age = p in
                       name ++ " is " ++ show age ++ " years old"

列表推导
--------

.. List comprehensions
.. -------------------

.. Idris provides *comprehension* notation as a convenient shorthand
.. for building lists. The general form is:

Idris 提供了 **推导** 记法作为构建列表的简便写法。一般形式为：

::

    [ expression | qualifiers ]

.. This generates the list of values produced by evaluating the
.. ``expression``, according to the conditions given by the comma
.. separated ``qualifiers``. For example, we can build a list of
.. Pythagorean triples as follows:

它会根据逗号分隔的限定式 ``qualifiers`` 给定的条件，通过求值表达式 ``expression``
产生的值来生成列表。例如，我们可以按如下方式构建构建勾股三角的列表：

.. code-block:: idris

    pythag : Int -> List (Int, Int, Int)
    pythag n = [ (x, y, z) | z <- [1..n], y <- [1..z], x <- [1..y],
                             x*x + y*y == z*z ]

.. The ``[a..b]`` notation is another shorthand which builds a list of
.. numbers between ``a`` and ``b``. Alternatively ``[a,b..c]`` builds a
.. list of numbers between ``a`` and ``c`` with the increment specified
.. by the difference between ``a`` and ``b``. This works for type ``Nat``,
.. ``Int`` and ``Integer``, using the ``enumFromTo`` and ``enumFromThenTo``
.. function from the prelude.

``[a..b]`` 是另一种构建从 ``a`` 到 ``b`` 的数列的简便记法。此外，
``[a,b..c]`` 以 ``a``，``b`` 之差为增量，构建从 ``a`` 到 ``c`` 的数列。
它可作用于 ``Nat``、``Int`` 以及 ``Integer``，它们使用了 Prelude 中的 ``enumFromTo``
与 ``enumFromThenTo`` 函数。

.. hint:: 推导式

    推导式（Comprehension）来源于集合的构建方法，即 ``{x|x∈X⋀Φ(x)}``，其中的 ``Φ(x)``
    即为限定式（Qualifier）。详情参见
    `维基百科 <https://en.wikipedia.org/wiki/Set-builder_notation#Parallels_in_programming_languages>`_ 。


``case`` 表达式
---------------

.. ``case`` expressions
.. --------------------

.. Another way of inspecting intermediate values of *simple* types is to
.. use a ``case`` expression. The following function, for example, splits
.. a string into two at a given character:

另一种检查 **简单** 类型的中间值的方法是使用 ``case`` 表达式。例如，
以下函数在给定的字符处将字符串分为两部分：

.. code-block:: idris

    splitAt : Char -> String -> (String, String)
    splitAt c x = case break (== c) x of
                      (x, y) => (x, strTail y)

.. ``break`` is a library function which breaks a string into a pair of
.. strings at the point where the given function returns true. We then
.. deconstruct the pair it returns, and remove the first character of the
.. second string.

``break`` 是个库函数，它从给定的函数返回 true 的位置将字符串分为一个字符串的序对。
我们接着析构它返回的序对， 并移除第二个字符串的第一个字符。

.. A ``case`` expression can match several cases, for example, to inspect
.. an intermediate value of type ``Maybe a``. Recall ``list_lookup``
.. which looks up an index in a list, returning ``Nothing`` if the index
.. is out of bounds. We can use this to write ``lookup_default``, which
.. looks up an index and returns a default value if the index is out of
.. bounds:

一个 ``case`` 表达式可匹配多种情况，例如去检查一个类型为 ``Maybe a`` 的中间值。
回想 ``list_lookup``，它按索引查找列表中的元素，若索引越界则返回 ``Nothing``。
我们可以用它来编写 ``lookup_default``，该函数按索引查找元素，若索引越界则返回默认值：

.. code-block:: idris

    lookup_default : Nat -> List a -> a -> a
    lookup_default i xs def = case list_lookup i xs of
                                  Nothing => def
                                  Just x => x

.. If the index is in bounds, we get the value at that index, otherwise
.. we get a default value:

若索引在界内，我们就会获得该索引对应的值，否则就会获得默认值：

::

    *UsefulTypes> lookup_default 2 [3,4,5,6] (-1)
    5 : Integer
    *UsefulTypes> lookup_default 4 [3,4,5,6] (-1)
    -1 : Integer

.. **Restrictions:** The ``case`` construct is intended for simple
.. analysis of intermediate expressions to avoid the need to write
.. auxiliary functions, and is also used internally to implement pattern
.. matching ``let`` and lambda bindings. It will *only* work if:

**限制：** ``case`` 构造用于对中间表达式进行简单的分析，以此避免编写辅助函数，
它也在内部用于实现 ``let`` 和 λ-绑定的模式匹配。它 **仅** 在以下情况中可用：

.. - Each branch *matches* a value of the same type, and *returns* a
..   value of the same type.

.. - The type of the result is “known”. i.e. the type of the expression
..   can be determined *without* type checking the ``case``-expression
..   itself.

- 每个分支 **匹配** 一个相同类型的值，并 **返回** 一个相同类型的值。

- 结果的类型是「已知」的，即表达式的类型无需对该 ``case``
  表达式进行类型检查就能确定。

完全性
======

.. Totality
.. ========

.. Idris distinguishes between *total* and *partial* functions.
.. A total function is a function that either:

Idris 区分 **完全（全，Total）** 函数与 **部分（偏，Partial）** 函数。
全函数满足以下情况之一：

.. + Terminates for all possible inputs, or
.. + Produces a non-empty, finite, prefix of a possibly infinite result

+ 对于所有可能的输入都会终止，或
+ 产生一个非空的，有限的，可能为无限结果的前缀

.. If a function is total, we can consider its type a precise description of what
.. that function will do. For example, if we have a function with a return
.. type of ``String`` we know something different, depending on whether or not
.. it's total:

若一个函数是完全的，我们可以认为其类型精确描述了该函数会做什么。例如，
若我们有一个返回类型为 ``String`` 的函数，根据它是否完全，我们能知道的东西会有所不同：

.. + If it's total, it will return a value of type ``String`` in finite time;
.. + If it's partial, then as long as it doesn't crash or enter an infinite loop,
..   it will return a ``String``.

+ 若它是全函数，就会在有限的时间内返回一个类型为 ``String`` 的值；
+ 若它是偏函数，那么只要它不崩溃或进入无限循环，就会返回一个 ``String``。

.. Idris makes this distinction so that it knows which functions are safe to
.. evaluate while type checking (as we've seen with :ref:`sect-fctypes`). After all,
.. if it tries to evaluate a function during type checking which doesn't
.. terminate, then type checking won't terminate!
.. Therefore, only total functions will be evaluated during type checking.
.. Partial functions can still be used in types, but will not be evaluated
.. further.

Idris 对此作了区分，因此它知道在进行类型检查（正如我们在 :ref:`sect-fctypes`
一节所见）的时候，哪些函数可以安全地求值。毕竟，若它在类型检查时试图对一个不终止的函数求值，
那么类型检查将无法终止！因此，在类型检查时只有全函数才会被求值。偏函数仍然可在类型中使用，
但它们不会被进一步求值。
