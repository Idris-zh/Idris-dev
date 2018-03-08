.. _sect-namespaces:

**********************
.. Modules and Namespaces

模块和命名空间
**********************

.. An Idris program consists of a collection of modules. Each module
.. includes an optional ``module`` declaration giving the name of the
.. module, a list of ``import`` statements giving the other modules which
.. are to be imported, and a collection of declarations and definitions of
.. types, interfaces and functions. For example, the listing below gives a
.. module which defines a binary tree type ``BTree`` (in a file
.. ``Btree.idr``):

一个 Idris 程序由模块集合组成。每个模块包含一个可选的 ``module`` 声明来命名模块，
一系列的 ``import`` 语句列出导入的其他模块，还有一堆类型的声明和定义。例如，下面这个
模块，定义了一个二叉树类型 ``BTree`` (在文件 ``Btree.idr`` 中):

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

修饰词 ``export`` 和 ``public export`` 说那些名称是其他模块可见的。下面会更深入的说明这些。

.. Then, this gives a main program (in a file
.. ``bmain.idr``) which uses the ``Btree`` module to sort a list:

这里给出了一个主程序（在 ``bmain.idr`` 文件里），利用 ``Btree`` 模块对列表排序：

.. code-block:: idris

    module Main

    import Btree

    main : IO ()
    main = do let t = toTree [1,8,2,7,9,3]
              print (Btree.toList t)

.. The same names can be defined in multiple modules: names are *qualified* with
.. the name of the module.  The names defined in the ``Btree`` module are, in
.. full:

同一名称可以在多个模块中被定义：名称可以通过模块的名字*来限定*，``Btree`` 模块定义的全部命名如下：

+ ``Btree.BTree``
+ ``Btree.Leaf``
+ ``Btree.Node``
+ ``Btree.insert``
+ ``Btree.toList``
+ ``Btree.toTree``

.. If names are otherwise unambiguous, there is no need to give the fully
.. qualified name. Names can be disambiguated either by giving an explicit
.. qualification, or according to their type.

如果名称是可以区分的，就没必要给出完全的限制名称。通过明确的限定，或者根据它们的类型，可以
对名称去歧义。

.. There is no formal link between the module name and its filename,
.. although it is generally advisable to use the same name for each. An
.. ``import`` statement refers to a filename, using dots to separate
.. directories. For example, ``import foo.bar`` would import the file
.. ``foo/bar.idr``, which would conventionally have the module declaration
.. ``module foo.bar``. The only requirement for module names is that the
.. main module, with the ``main`` function, must be called
.. ``Main``—although its filename need not be ``Main.idr``.

模块名和文件名没有正式的联系，尽管通常会建议使用同一名字。一个引用文件名的 ``import`` 语句，
通过 ``.`` 来分割路径。例如，``import foo.bar`` 会导入文件 ``foo/bar.idr``, 
通常会有模块声明 ``module foo.bar``。对模块名字的唯一要求是，使用 ``main`` 函数，
必须命名为 ``Main``，然而文件名不需要是 ``Main.idr``。

.. Export Modifiers

Export 修饰符
================

.. Idris allows for fine-grained control over the visibility of a
.. module's contents. By default, all names defined in a module are kept
.. private.  This aides in specification of a minimal interface and for
.. internal details to be left hidden.  Idris allows for functions,
.. types, and interfaces to be marked as: ``private``, ``export``, or
.. ``public export``.  Their general meaning is as follows:

Idris 允许对模块内容可见性进行细粒度控制。默认情况下，定义在一个模块中的所有名字
都是私有的。这会对最小化界面和隐藏内部细节有很大帮助。Idris 允许函数、类型和界面
被标记为： ``private``, ``export``, 或者 ``public export``。它们通常含义如下:

.. - ``private`` meaning that it's not exported at all. This is the
..   default.

- ``private`` 的意思是完全不被导出。这是默认的。

.. - ``export`` meaning that its top level type is exported.

- ``export`` 的意思是它的顶级类型是导出的。

.. - ``public export`` meaning that the entire definition is exported.

- ``public export`` 的意思是整个定义都是导出的。

.. A further restriction in modifying the visibility is that definitions
.. must not refer to anything within a lower level of visibility. For
.. example, ``public export`` definitions cannot use private names, and
.. ``export`` types cannot use private names. This is to prevent private
.. names leaking into a module's interface.

改变可见性中的更进一步限制是，定义一定不能引用到任何更低层次的可见性。例如，
``public export`` 定义不能使用私有命名, 而且 ``export`` 类型不能使用私有命名。
这是为了避免私有命名泄漏到模块的界面。

.. Meaning for Functions

对于函数的涵义
---------------------

.. - ``export`` the type is exported

- ``export`` 类型是导出的

.. - ``public export`` the type and definition are exported, and the
..   definition can be used after it is imported. In other words, the
..   definition itself is considered part of the module's interface. The
..   long name ``public export`` is intended to make you think twice
..   about doing this.

- ``public export`` 类型和定义是导出的, 而且在导出之后定义是可以被使用的。
  换言之, 定义本身被认为是模块界面的一部分。长名字 ``public export`` 的目的
  是让你三思后再这样做。

.. .. note::

..   Type synonyms in Idris are created by writing a function. When
..   setting the visibility for a module, it might be a good idea to
..   ``public export`` all type synonyms if they are to be used outside
..   the module. Otherwise, Idris won't know what the synonym is a
..   synonym for.

.. note::

   Idris 中的类型同义通过写一个函数来创建。当为一个模块设置可见性，
   如果类型同义被使用在模块外，``public export`` 所有的类型同义可能是个好主意。
   否则，Idris 不会知晓同义词是什么同义词。

.. Since ``public export`` means that a function's definition is exported,
.. this effectively makes the function definition part of the module's API.
.. Therefore, it's generally a good idea to avoid using ``public export`` for
.. functions unless you really mean to export the full definition.

既然 ``public export`` 意味着一个函数的定义是导出的，这实际上就使得这个函数定义成为
模块 API 的一部分。因此，通常最好避免使用 ``public export`` 函数，除非你实在想导出完整定义。

.. Meaning for Data Types

对于数据类型的涵义
----------------------

.. For data types, the meanings are:

对于数据类型，涵义如下：

.. - ``export``  the type constructor is exported

- ``export``  类型构造函数是导出的

.. - ``public export`` the type constructor and data constructors are
..   exported

- ``public export`` 类型构造函数和数据构造函数是导出的


.. Meaning for Interfaces

对于界面的涵义
----------------------

.. For interfaces, the meanings are:

对于界面，涵义如下：

.. - ``export`` the interface name is exported

- ``export`` 界面名字是导出的

.. - ``public export`` the interface name, method names and default
..   definitions are exported

- ``public export`` 界面名、方法名和默认定义都是导出的

.. ``%access`` Directive

``%access`` 指令
----------------------

.. The default export mode can be changed with the ``%access``
.. directive, for example:

默认的导出模式可以通过 ``%access`` 指令改变，例如：

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

在这种情况下，没有访问修饰符的任何函数可以导出为 ``export``，而不是 ``private``。

.. Propagating Inner Module API's

内部模块 API 的传递
-------------------------------

.. Additionally, a module can re-export a module it has imported, by using
.. the ``public`` modifier on an ``import``. For example:

另外，一个模块能再次导出一个已经导入的模块，通过在 ``import`` 使用 ``public`` 修饰符。
例如：

.. code-block:: idris

    module A

    import B
    import public C

.. The module ``A`` will export the name ``a``, as well as any public or
.. abstract names in module ``C``, but will not re-export anything from
.. module ``B``.

模块 ``A`` 会导出名字 ``a``, 还有模块 ``C`` 中任何公共或者抽象的命名，
但不能从模块 ``B`` 再导入任何东西。

.. Explicit Namespaces

显式命名空间
===================

.. Defining a module also defines a namespace implicitly. However,
.. namespaces can also be given *explicitly*. This is most useful if you
.. wish to overload names within the same module:

定义一个模块也隐式的定义了一个命名空间。不过，命名空间也可以*显式*给出。当你想重载同一
模块内命名，这个会很有用：

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

这个（诚然做作的）模块使用完全限定命名 ``Foo.x.test`` 和 ``Foo.y.test`` 定义了两个函数, 
可以通过函数类型去歧义：

::

    *Foo> test 3
    6 : Int
    *Foo> test "foo"
    "foofoo" : String

.. Parameterised blocks

参数化的块
====================

.. Groups of functions can be parameterised over a number of arguments
.. using a ``parameters`` declaration, for example:

使用 ``parameters`` 声明可以在许多参数上对函数组进行参数化，例如：

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

 ``parameters`` 块的作用是对块内的每个函数、类型和数据构造函数添加声明参数。
 具体的说，将参数到添加到参数列表的前面。

::

    *params> :t addAll
    addAll : Nat -> Nat -> Nat -> Nat

.. and the following definition.

还有下面的定义。

.. code-block:: idris

    addAll : (x : Nat) -> (y : Nat) -> (z : Nat) -> Nat
    addAll x y z = x + y + z

.. Parameters blocks can be nested, and can also include data declarations,
.. in which case the parameters are added explicitly to all type and data
.. constructors. They may also be dependent types with implicit arguments:

可以嵌套参数块，而且在将参数显式的添加到所有类型和数据构造函数的情况下，可以包含数据声明。
它们也可能是使用隐式参数的依赖类型：

.. code-block:: idris

    parameters (y : Nat, xs : Vect x a)
      data Vects : Type -> Type where
        MkVects : Vect y a -> Vects a

      append : Vects a -> Vect (x + y) a
      append (MkVects ys) = xs ++ ys

.. To use ``Vects`` or ``append`` outside the block, we must also give the
.. ``xs`` and ``y`` arguments. Here, we can use placeholders for the values
.. which can be inferred by the type checker:

为了在代码块外使用 ``Vects`` 或者 ``append``，我们必须给出 ``xs`` 和 ``y`` 参数。
在这里，对可以通过类型检查器推断出的值，可是使用占位符。

::

    *params> show (append _ _ (MkVects _ [1,2,3] [4,5,6]))
    "[1, 2, 3, 4, 5, 6]" : String
