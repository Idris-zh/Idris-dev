.. _sect-namespaces:

**************
模块与命名空间
**************

.. **********************
.. Modules and Namespaces
.. **********************

.. An Idris program consists of a collection of modules. Each module
.. includes an optional ``module`` declaration giving the name of the
.. module, a list of ``import`` statements giving the other modules which
.. are to be imported, and a collection of declarations and definitions of
.. types, interfaces and functions. For example, the listing below gives a
.. module which defines a binary tree type ``BTree`` (in a file
.. ``Btree.idr``):

Idris 程序由一组模块构成。每个模块包含一个可选的 ``module`` 声明来命名模块，
一系列 ``import`` 语句用于导入其它模块，还有一堆类型、接口和函数的声明与定义。
例如，以下模块定义了二叉树类型 ``BTree`` （在 ``Btree.idr`` 文件中）：

.. code-block:: idris

    module Btree

    public export
    data BTree a = Leaf
                 | Node (BTree a) a (BTree a)

    export
    insert : Ord a => a -> BTree a -> BTree a
    insert x Leaf = Node Leaf x Leaf
    insert x (Node l v r) = if (x < v) then (Node (insert x l) v r)
                                       else (Node l v (insert x r))

    export
    toList : BTree a -> List a
    toList Leaf = []
    toList (Node l v r) = Btree.toList l ++ (v :: Btree.toList r)

    export
    toTree : Ord a => List a -> BTree a
    toTree [] = Leaf
    toTree (x :: xs) = insert x (toTree xs)

.. The modifiers ``export`` and ``public export`` say which names are visible
.. from other modules. These are explained further below.

修饰符 ``export`` 和 ``public export`` 说明了哪些名称在其它模块中可见。
后面会有进一步的解释。

.. Then, this gives a main program (in a file
.. ``bmain.idr``) which uses the ``Btree`` module to sort a list:

这里给出了一个 main 程序（在 ``bmain.idr`` 文件中），它利用 ``Btree`` 模块对列表排序：

.. code-block:: idris

    module Main

    import Btree

    main : IO ()
    main = do let t = toTree [1,8,2,7,9,3]
              print (Btree.toList t)

.. The same names can be defined in multiple modules: names are *qualified* with
.. the name of the module.  The names defined in the ``Btree`` module are, in
.. full:

同一名称可以在多个模块中定义：名称可以通过模块名来字 **限定**。
``Btree`` 模块中定义的全部名称如下：

+ ``Btree.BTree``
+ ``Btree.Leaf``
+ ``Btree.Node``
+ ``Btree.insert``
+ ``Btree.toList``
+ ``Btree.toTree``

.. If names are otherwise unambiguous, there is no need to give the fully
.. qualified name. Names can be disambiguated either by giving an explicit
.. qualification, or according to their type.

如果名称可以区分，就没必要给出完整的限定名。通过明确的限定，或者根据它们的类型，
可以消除名称歧义。

.. There is no formal link between the module name and its filename,
.. although it is generally advisable to use the same name for each. An
.. ``import`` statement refers to a filename, using dots to separate
.. directories. For example, ``import foo.bar`` would import the file
.. ``foo/bar.idr``, which would conventionally have the module declaration
.. ``module foo.bar``. The only requirement for module names is that the
.. main module, with the ``main`` function, must be called
.. ``Main``—although its filename need not be ``Main.idr``.

模块名和文件名没有正式的联系，尽管通常会建议使用同一名字。引用文件名的 ``import``
语句通过 ``.`` 来分割路径。例如，``import foo.bar`` 会导入文件 ``foo/bar.idr``，
其中通常会有模块声明 ``module foo.bar``。对模块名字的唯一要求是，包含 ``main``
函数的主模块必须命名为 ``Main``，然而文件名无需为 ``Main.idr``。

导出修饰符
=============

.. Export Modifiers
.. ================

.. Idris allows for fine-grained control over the visibility of a
.. module's contents. By default, all names defined in a module are kept
.. private.  This aides in specification of a minimal interface and for
.. internal details to be left hidden.  Idris allows for functions,
.. types, and interfaces to be marked as: ``private``, ``export``, or
.. ``public export``.  Their general meaning is as follows:

Idris 允许对模块内容的可见性进行细粒度的控制。默认情况下，模块中定义的所有名字
都是私有的。这有助于最小化接口并隐藏内部细节。Idris 允许将函数、类型和接口标记为：
``private``、``export`` 或 ``public export``。它们的含义通常如下:

.. - ``private`` meaning that it's not exported at all. This is the
..   default.

.. - ``export`` meaning that its top level type is exported.

.. - ``public export`` meaning that the entire definition is exported.

- ``private`` 表示完全不被导出。这是默认的。
-
- ``export`` 表示顶级类型是导出的。
-
- ``public export`` 表示整个定义都是导出的。

.. A further restriction in modifying the visibility is that definitions
.. must not refer to anything within a lower level of visibility. For
.. example, ``public export`` definitions cannot use private names, and
.. ``export`` types cannot use private names. This is to prevent private
.. names leaking into a module's interface.

更改可见性的另一限制是，定义无法引用可见性更低的名称。例如，
``public export`` 的定义无法使用私有名称, 而 ``export`` 的类型也无法使用私有名称。
这是为了避免私有名称泄漏到模块的接口中。

对于函数的意义
--------------

.. Meaning for Functions
.. ---------------------

.. - ``export`` the type is exported

.. - ``public export`` the type and definition are exported, and the
..   definition can be used after it is imported. In other words, the
..   definition itself is considered part of the module's interface. The
..   long name ``public export`` is intended to make you think twice
..   about doing this.

- ``export`` 的类型是导出的

- ``public export`` 的类型和定义是导出的, 在导出之后定义可被使用。
  换言之, 定义本身被认为是模块接口的一部分。名字 ``public export``
  比较长是为了让你三思而后行。

.. .. note::

..   Type synonyms in Idris are created by writing a function. When
..   setting the visibility for a module, it might be a good idea to
..   ``public export`` all type synonyms if they are to be used outside
..   the module. Otherwise, Idris won't know what the synonym is a
..   synonym for.

.. note::

   Idris 中的同义类型可通过编写函数来创建。在为模块设置可见性时，
   如果同义类型要在模块外部使用，那么将它们声明为 ``public export`` 可能会更好。
   否则，Idris 会无法获知该类型与谁同义。

.. Since ``public export`` means that a function's definition is exported,
.. this effectively makes the function definition part of the module's API.
.. Therefore, it's generally a good idea to avoid using ``public export`` for
.. functions unless you really mean to export the full definition.

既然 ``public export`` 意味着函数的定义是导出的，那么该函数的定义实际上也就成为了模块
API 的一部分。因此，除非你确实想要导出函数的完整定义，否则最好避免将其声明为
``public export`` 。

对于数据类型的涵义
------------------

.. Meaning for Data Types
.. ----------------------

.. For data types, the meanings are:

.. - ``export``  the type constructor is exported

.. - ``public export`` the type constructor and data constructors are
..   exported

对于数据类型，其涵义如下：

- ``export`` 的类型构造器是导出的

- ``public export`` 的类型构造器和数据构造器是导出的


对于接口的涵义
--------------

.. Meaning for Interfaces
.. ----------------------

.. For interfaces, the meanings are:

.. - ``export`` the interface name is exported

.. - ``public export`` the interface name, method names and default
..   definitions are exported

对于接口，其涵义如下：

- ``export`` 的接口名字是导出的

- ``public export`` 的接口名、方法名和默认定义都是导出的

``%access`` 指令
----------------------

.. ``%access`` Directive
.. ----------------------

.. The default export mode can be changed with the ``%access``
.. directive, for example:

默认的导出模式可以通过 ``%access`` 指令更改，例如：

.. code-block:: idris

    module Btree

    %access export

    public export
    data BTree a = Leaf
                 | Node (BTree a) a (BTree a)

    insert : Ord a => a -> BTree a -> BTree a
    insert x Leaf = Node Leaf x Leaf
    insert x (Node l v r) = if (x < v) then (Node (insert x l) v r)
                                       else (Node l v (insert x r))

    toList : BTree a -> List a
    toList Leaf = []
    toList (Node l v r) = Btree.toList l ++ (v :: Btree.toList r)

    toTree : Ord a => List a -> BTree a
    toTree [] = Leaf
    toTree (x :: xs) = insert x (toTree xs)

.. In this case, any function with no access modifier will be exported as
.. ``export``, rather than left ``private``.

在这种情况下，没有访问修饰符的任何函数均可以导出为 ``export``，而不是 ``private``。

内部模块 API 的传递
-------------------

.. Propagating Inner Module API's
.. -------------------------------

.. Additionally, a module can re-export a module it has imported, by using
.. the ``public`` modifier on an ``import``. For example:

另外，通过对 ``import`` 使用 ``public`` 修饰符，可将模块内导入的模块再次导出。
例如：

.. code-block:: idris

    module A

    import B
    import public C

.. The module ``A`` will export the name ``a``, as well as any public or
.. abstract names in module ``C``, but will not re-export anything from
.. module ``B``.

模块 ``A`` 会导出名字 ``a``, 以及模块 ``C`` 中所有公共或抽象的名称，
但无法再从模块 ``B`` 中导出任何东西。

显式命名空间
============

.. Explicit Namespaces
.. ===================

.. Defining a module also defines a namespace implicitly. However,
.. namespaces can also be given *explicitly*. This is most useful if you
.. wish to overload names within the same module:

在定义模块的同时，也会隐式定义一个命名空间。然而，该命名空间也可以 **显式**
给出。当你想在同一模块内重载名称时，它会非常有用：

.. code-block:: idris

    module Foo

    namespace x
      test : Int -> Int
      test x = x * 2

    namespace y
      test : String -> String
      test x = x ++ x

.. This (admittedly contrived) module defines two functions with fully
.. qualified names ``Foo.x.test`` and ``Foo.y.test``, which can be
.. disambiguated by their types:

这个（明显人为设计的）模块使用完整的限定名 ``Foo.x.test`` 和 ``Foo.y.test``
定义了两个函数，二者可以根据函数类型来消歧义：

::

    *Foo> test 3
    6 : Int
    *Foo> test "foo"
    "foofoo" : String

形参化的块
==========

.. Parameterised blocks
.. ====================

.. Groups of functions can be parameterised over a number of arguments
.. using a ``parameters`` declaration, for example:

一组函数的多个参数可通过 ``parameters`` 声明进行形参化（Parameterise），例如：

.. code-block:: idris

    parameters (x : Nat, y : Nat)
      addAll : Nat -> Nat
      addAll z = x + y + z

.. The effect of a ``parameters`` block is to add the declared parameters
.. to every function, type and data constructor within the
.. block. Specifically, adding the parameters to the front of the
.. argument list. Outside the block, the parameters must be given
.. explicitly. The ``addAll`` function, when called from the REPL, will
.. thus have the following type signature.

``parameters`` 形参块的作用是为块中每个函数、类型和数据构造器添加形参声明。
具体来说，就是将形参添加到参数列表的前面。在此块外，形参必须显式地给定。
因此在 REPL 中调用 ``addAll`` 函数时，它会拥有以下函数声明：

::

    *params> :t addAll
    addAll : Nat -> Nat -> Nat -> Nat

.. and the following definition.

以及以下定义：

.. code-block:: idris

    addAll : (x : Nat) -> (y : Nat) -> (z : Nat) -> Nat
    addAll x y z = x + y + z

.. Parameters blocks can be nested, and can also include data declarations,
.. in which case the parameters are added explicitly to all type and data
.. constructors. They may also be dependent types with implicit arguments:

形参块可以嵌套。在为所有类型和数据构造器显式地添加形参时，
块中也可包含数据声明。它们也可以是带有隐式参数的依赖类型：

.. code-block:: idris

    parameters (y : Nat, xs : Vect x a)
      data Vects : Type -> Type where
        MkVects : Vect y a -> Vects a

      append : Vects a -> Vect (x + y) a
      append (MkVects ys) = xs ++ ys

.. To use ``Vects`` or ``append`` outside the block, we must also give the
.. ``xs`` and ``y`` arguments. Here, we can use placeholders for the values
.. which can be inferred by the type checker:

为了在形参块外使用 ``Vects`` 或者 ``append``，我们必须给出参数 ``xs`` 和 ``y``。
在这里，我们可以用占位符来代表类型检查器能推断出的值：

::

    *params> show (append _ _ (MkVects _ [1,2,3] [4,5,6]))
    "[1, 2, 3, 4, 5, 6]" : String

.. hint:: **形参（Parameter）与实参（Argument）**

    在数学中，对于函数 ``f(x)``，``f`` 为函数名，``x`` 则称作函数 ``f``
    的形式参数（Parameter），简称形参；在函数应用时，传输函数的参数则称作实际参数
    （Argument），简称实参。例如 ``f(2)`` 中的 ``2`` 即为传入 ``f(x)`` 的形参。

    在英文原文中，有时并不明确区分形参与实参，一般统译作「参数」。只有当需要明确区分时，
    才分别译作「形参」与「实参」。
