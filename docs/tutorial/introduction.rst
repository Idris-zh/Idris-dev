.. _sect-intro:

************
.. Introduction

引言
************

.. In conventional programming languages, there is a clear distinction
.. between *types* and *values*. For example, in `Haskell
.. <http://www.haskell.org>`_, the following are types, representing
.. integers, characters, lists of characters, and lists of any value
.. respectively:

在传统编程语言中，**类型** 与 **值** 之间有明确的区分。例如在 `Haskell
<http://www.haskell.org>`_ 中, 下面这些类型分别表示整数、字符、字符列表
以及任意值的列表：

-  ``Int``, ``Char``, ``[Char]``, ``[a]``

.. Correspondingly, the following values are examples of inhabitants of
.. those types:

相应地，下面这些值分别为上述类型的成员：

-  ``42``, ``’a’``, ``"Hello world!"``, ``[2,3,4,5,6]``

.. In a language with *dependent types*, however, the distinction is less
.. clear. Dependent types allow types to “depend” on values — in other
.. words, types are a *first class* language construct and can be
.. manipulated like any other value. The standard example is the type of
.. lists of a given length [1]_, ``Vect n a``, where ``a`` is the element
.. type and ``n`` is the length of the list and can be an arbitrary term.

然而，在带有 **依赖类型** 的语言中，它们的区别并不明显。
依赖类型允许类型「依赖」于值，换句话说，类型是 **一等** 的语言构造，
即它可以像其它值一样操作。典型的例子就是带有长度的列表类型 [1]_ ：
``Vect n a``，其中 ``a`` 为元素类型，而 ``n`` 为该列表的长度且可以为任意项。

.. When types can contain values, and where those values describe
.. properties, for example the length of a list, the type of a function
.. can begin to describe its own properties. Take for example the
.. concatenation of two lists. This operation has the property that the
.. resulting list's length is the sum of the lengths of the two input
.. lists. We can therefore give the following type to the ``app``
.. function, which concatenates vectors:

当类型可以包含描述性质的值（如列表的长度）时，函数的类型便可开始描述其自身的性质了。
比如当连接两个列表时，该操作拥有性质：结果列表的长度为两个输入列表的长度之和。
因此我们可以为 ``app`` 函数赋予如下类型，它用于连接向量：

.. code-block:: idris

    app : Vect n a -> Vect m a -> Vect (n + m) a

.. This tutorial introduces Idris, a general purpose functional
.. programming language with dependent types. The goal of the Idris
.. project is to build a dependently typed language suitable for
.. verifiable general purpose programming. To this end, Idris is a compiled
.. language which aims to generate efficient executable code. It also has
.. a lightweight foreign function interface which allows easy interaction
.. with external ``C`` libraries.

本教程介绍了 Idris，一个通用目的的依赖类型函数式编程语言。Idris
项目的目的在于构建一个适用于可验证的，通用目的编程的依赖类型语言。
为此，Idris 旨在生成高效可执行代码的编译型语言。
它还拥有一个轻量级的外部函数接口，可与外部 ``C`` 库轻松交互。

.. Intended Audience

目标人群
=================

.. This tutorial is intended as a brief introduction to the language, and
.. is aimed at readers already familiar with a functional language such
.. as `Haskell <http://www.haskell.org>`_ or `OCaml <http://ocaml.org>`_.
.. In particular, a certain amount of familiarity with Haskell syntax is
.. assumed, although most concepts will at least be explained
.. briefly. The reader is also assumed to have some interest in using
.. dependent types for writing and verifying systems software.

本教程旨在简要介绍该语言，面向已经熟悉函数式语言（如 `Haskell <http://www.haskell.org>`_
或 `OCaml <http://ocaml.org>`_ ）的读者。尽管大多数概念会被简要地解释，
读者需要对 Haskell 的语法有一定程度的熟悉。我们亦假设读者有兴趣使用依赖类型来编写和验证系统软件。

.. For a more in-depth introduction to Idris, which proceeds at a much slower
.. pace, covering interactive program development, with many more examples, see
.. `Type-Driven Development with Idris <https://www.manning.com/books/type-driven-development-with-idris>`_
.. by Edwin Brady, available from `Manning <https://www.manning.com>`_.

对 Idris 更加深入的介绍，见 Edwin Brady 所著的 `Idris 类型驱动开发
<https://www.manning.com/books/type-driven-development-with-idris>`_ ，
其进度要慢得多，涵盖了交互式程序开发以及更多的例子，本书可从
`Manning <https://www.manning.com>`_ 获取。

.. Example Code

示例代码
============

.. This tutorial includes some example code, which has been tested with
.. against Idris. These files are available with the Idris distribution,
.. so that you can try them out easily. They can be found under
.. ``samples``. It is, however, strongly recommended that you type
.. them in yourself, rather than simply loading and reading them.

本教程包括一些经过 Idris 测试的示例代码。这些文件可从 Idris
发行版中获取，因此您可以轻松尝试它们。它们可在 ``samples`` 中找到。
然而，我们强烈建议您亲自输入它们，而非简单地加载并阅读。

.. .. [1]
..    Typically, and perhaps confusingly, referred to in the dependently typed programming literature as “vectors”

.. [1]
   也许会令人困惑，它在依赖类型编程的文献中通常被称作「向量 (Vector) 」。
