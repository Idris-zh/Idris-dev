.. _sect-interactive:

**********
交互式编辑
**********

.. *******************
.. Interactive Editing
.. *******************

.. By now, we have seen several examples of how Idris’ dependent type
.. system can give extra confidence in a function’s correctness by giving
.. a more precise description of its intended behaviour in its *type*. We
.. have also seen an example of how the type system can help with EDSL
.. development by allowing a programmer to describe the type system of an
.. object language. However, precise types give us more than verification
.. of programs — we can also exploit types to help write programs which
.. are *correct by construction*.

目前，我们已经见过几个 Idris 的例子了，它的依赖类型系统可以在函数的 **类型**
中为期望的行为添加更加精确的描述，为函数的正确性增加额外的信心。
我们也见识过类型系统如何帮助开发 EDSL 了，它允许程序员描述目标语言的类型系统。
然而，精确的类型不止赋予了我们程序验证的能力，我们还可以利用类型来帮助 **构造**
出 **正确** 的程序。

.. The Idris REPL provides several commands for inspecting and
.. modifying parts of programs, based on their types, such as case
.. splitting on a pattern variable, inspecting the type of a
.. hole, and even a basic proof search mechanism. In this
.. section, we explain how these features can be exploited by a text
.. editor, and specifically how to do so in `Vim
.. <https://github.com/idris-hackers/idris-vim>`_. An interactive mode
.. for `Emacs <https://github.com/idris-hackers/idris-mode>`_ is also
.. available.

Idris 的 REPL 提供了一些命令，可基于程序的类型来检查和修改程序的片段，
例如在模式变量中拆分情况，检查坑的类型，甚至还有基本的证明搜索机制。在本节中，
我们解释了如何在文本编辑器中，特别是如何在
`Vim <https://github.com/idris-hackers/idris-vim>`_ 中利用这些特性。
`Emacs <https://github.com/idris-hackers/idris-mode>`_ 的交互式模式也可以。


在 REPL 中编辑
==============

.. Editing at the REPL
.. ===================

.. The REPL provides a number of commands, which we will describe
.. shortly, which generate new program fragments based on the currently
.. loaded module. These take the general form

.. ::

..     :command [line number] [name]

REPL 提供了许多命令，我们稍后会介绍它们。它们基于当前加载的模块来生成新的程序片段。
其一般形式为：

::

    :命令 [行号] [名称]

.. That is, each command acts on a specific source line, at a specific
.. name, and outputs a new program fragment. Each command has an
.. alternative form, which *updates* the source file in-place:

.. ::

..     :command! [line number] [name]

也就是说，每条命令都会作用在源码中特定行的特定名称上，然后输出一个新的程序片段。
每个命令都有一种原地 **更新** 源文件的形式：

::

    :命令! [行号] [名称]

.. When the REPL is loaded, it also starts a background process which
.. accepts and responds to REPL commands, using ``idris --client``. For
.. example, if we have a REPL running elsewhere, we can execute commands
.. such as:

当 REPL 被加载时，它还会用 ``idris --client`` 在后台启动一个进程，该进程接受并响应
REPL 命令。例如，如果我们在某处运行着 REPL，就可以执行这样的命令：

::

    $ idris --client ':t plus'
    Prelude.Nat.plus : Nat -> Nat -> Nat
    $ idris --client '2+2'
    4 : Integer

.. A text editor can take advantage of this, along with the editing
.. commands, in order to provide interactive editing support.

文本编辑器可以利用此特性与编辑命令来提供交互式编辑。

编辑命令
========

.. Editing Commands
.. ================

:addclause
----------

.. The ``:addclause n f`` command, abbreviated ``:ac n f``, creates a
.. template definition for the function named ``f`` declared on line
.. ``n``. For example, if the code beginning on line 94 contains:

``:addclause n f`` 命令，缩写为 ``:ac n f``，它为第 ``n`` 行声明的函数 ``f``
创建一个模版定义。例如，若从第 94 行开始的代码为：

.. code-block:: idris

    vzipWith : (a -> b -> c) ->
               Vect n a -> Vect n b -> Vect n c

.. then ``:ac 94 vzipWith`` will give:

那么 ``:ac 94 vzipWith`` 会给出：

.. code-block:: idris

    vzipWith f xs ys = ?vzipWith_rhs

.. The names are chosen according to hints which may be given by a
.. programmer, and then made unique by the machine by adding a digit if
.. necessary. Hints can be given as follows:

名称可根据程序员给定的提示来选择，必要时机器还会添加数字使其唯一。
我们可以像下面这样给出提示：

.. code-block:: idris

    %name Vect xs, ys, zs, ws

.. This declares that any names generated for types in the ``Vect`` family
.. should be chosen in the order ``xs``, ``ys``, ``zs``, ``ws``.

它声明了 ``Vect`` 类型族的名字应按照 ``xs``、``ys``、``zs``、``ws``
的顺序来选取。

:casesplit
----------

.. The ``:casesplit n x`` command, abbreviated ``:cs n x``, splits the
.. pattern variable ``x`` on line ``n`` into the various pattern forms it
.. may take, removing any cases which are impossible due to unification
.. errors. For example, if the code beginning on line 94 is:

``:casesplit n x`` 命令，缩写为 ``:cs n x``，它将第 ``n`` 行的模式变量 ``x``
拆分为能构成它的多种模式，并移除任何由于一致性错误而不可能出现的情况。
例如，若从第 94 行开始的代码为：

.. code-block:: idris

    vzipWith : (a -> b -> c) ->
               Vect n a -> Vect n b -> Vect n c
    vzipWith f xs ys = ?vzipWith_rhs

.. then ``:cs 96 xs`` will give:

那么 ``:cs 96 xs`` 会给出：

.. code-block:: idris

    vzipWith f [] ys = ?vzipWith_rhs_1
    vzipWith f (x :: xs) ys = ?vzipWith_rhs_2

.. That is, the pattern variable ``xs`` has been split into the two
.. possible cases ``[]`` and ``x :: xs``. Again, the names are chosen
.. according to the same heuristic. If we update the file (using
.. ``:cs!``) then case split on ``ys`` on the same line, we get:

即，模式变量 ``xs`` 被拆分成了 ``[]`` 和 ``x :: xs`` 两种情况。与之前一样，
名称的选取启发自相同的规则。若我们（用 ``:cs!``）更新文件后再拆分同一行的
``ys``，就会得到：

.. code-block:: idris

    vzipWith f [] [] = ?vzipWith_rhs_3

.. That is, the pattern variable ``ys`` has been split into one case
.. ``[]``, Idris having noticed that the other possible case ``y ::
.. ys`` would lead to a unification error.

即，模式变量 ``ys`` 被拆分成了 ``[]`` 这一个情况，因为 Idris 发现另一种可能的情况
``y :: ys`` 会导致一致性错误。

:addmissing
-----------

.. The ``:addmissing n f`` command, abbreviated ``:am n f``, adds the
.. clauses which are required to make the function ``f`` on line ``n``
.. cover all inputs. For example, if the code beginning on line 94 is:

``:addmissing n f`` 命令，缩写为 ``:am n f``，它为第 ``n`` 行的函数 ``f``
添加能覆盖所有输入情况的从句。例如，若从第 94 行开始的代码为：

.. code-block:: idris

    vzipWith : (a -> b -> c) ->
               Vect n a -> Vect n b -> Vect n c
    vzipWith f [] [] = ?vzipWith_rhs_1

.. then ``:am 96 vzipWith`` gives:

那么 ``:am 96 vzipWith`` 会给出：

.. code-block:: idris

    vzipWith f (x :: xs) (y :: ys) = ?vzipWith_rhs_2

.. That is, it notices that there are no cases for empty vectors,
.. generates the required clauses, and eliminates the clauses which would
.. lead to unification errors.

即，它注意到不存在空向量的情况，然后生成了需要的从句，并消除了会导致不一致性错误的从句。

:proofsearch
------------

.. The ``:proofsearch n f`` command, abbreviated ``:ps n f``, attempts to
.. find a value for the hole ``f`` on line ``n`` by proof search,
.. trying values of local variables, recursive calls and constructors of
.. the required family. Optionally, it can take a list of *hints*, which
.. are functions it can try applying to solve the hole. For
.. example, if the code beginning on line 94 is:

``:proofsearch n f`` 命令，缩写为 ``:ps n f``，它试图通过证明搜索、
尝试局部变量的值、递归调用以及所需类型族的构造器来为第 ``n`` 行的坑 ``f``
查找一个值。该命令也可以接受一个可选的 **提示（Hint）** 列表，
也就是可用于尝试解决此坑的函数列表。例如，若从第 94 行开始的代码为：

.. code-block:: idris

    vzipWith : (a -> b -> c) ->
               Vect n a -> Vect n b -> Vect n c
    vzipWith f [] [] = ?vzipWith_rhs_1
    vzipWith f (x :: xs) (y :: ys) = ?vzipWith_rhs_2

.. then ``:ps 96 vzipWith_rhs_1`` will give

那么 ``:ps 96 vzipWith_rhs_1`` 会给出：

.. code-block:: idris

    []

.. This works because it is searching for a ``Vect`` of length 0, of
.. which the empty vector is the only possibility. Similarly, and perhaps
.. surprisingly, there is only one possibility if we try to solve ``:ps
.. 97 vzipWith_rhs_2``:

它能工作是因为它在对长度为 0 的 ``Vect`` 进行搜索，而空向量是唯一的可能。
同样，在试图解决 ``:ps 97 vzipWith_rhs_2`` 时也出乎意料地只有一种可能：

.. code-block:: idris

    f x y :: (vzipWith f xs ys)

.. This works because ``vzipWith`` has a precise enough type: The
.. resulting vector has to be non-empty (a ``::``); the first element
.. must have type ``c`` and the only way to get this is to apply ``f`` to
.. ``x`` and ``y``; finally, the tail of the vector can only be built
.. recursively.

它能工作是因为 ``vzipWith`` 拥有足够精确的类型：其结果向量一定非空（即至少有一个
``::``）；第一个元素的类型必须为 ``c``，而得到它的唯一方法就是将 ``f`` 应用于
``x`` 和 ``y``；最后，该向量的尾部只能递归地构造。

:makewith
---------

.. The ``:makewith n f`` command, abbreviated ``:mw n f``, adds a
.. ``with`` to a pattern clause. For example, recall ``parity``. If line
.. 10 is:

``:makewith n f`` 命令，缩写为 ``:mw n f``，它为模式添加一个 ``with`` 从句。
以之前的 ``parity`` 为例。若第 10 行为：

.. code-block:: idris

    parity (S k) = ?parity_rhs

.. then ``:mw 10 parity`` will give:

那么 ``:mw 10 parity`` 会给出：

.. code-block:: idris

    parity (S k) with (_)
      parity (S k) | with_pat = ?parity_rhs

.. If we then fill in the placeholder ``_`` with ``parity k`` and case
.. split on ``with_pat`` using ``:cs 11 with_pat`` we get the following
.. patterns:

若我们在占位符 ``_`` 处填上 ``parity k``，并用 ``:cs 11 with_pat`` 拆分
``with_pat`` 的情况，就会得到以下模式：

.. code-block:: idris

      parity (S (plus n n)) | even = ?parity_rhs_1
      parity (S (S (plus n n))) | odd = ?parity_rhs_2

.. Note that case splitting has normalised the patterns here (giving
.. ``plus`` rather than ``+``). In any case, we see that using
.. interactive editing significantly simplifies the implementation of
.. dependent pattern matching by showing a programmer exactly what the
.. valid patterns are.

注意情况拆分规范化了该模式（即给的是 ``plus`` 而非 ``+``）。我们会看到在任何情况下，
使用交互式编辑向程序员展示有效的模式都能显著简化依赖模式匹配的实现。

Vim 交互式编辑
==============

.. Interactive Editing in Vim
.. ==========================

.. The editor mode for Vim provides syntax highlighting, indentation and
.. interactive editing support using the commands described above.
.. Interactive editing is achieved using the following editor commands,
.. each of which update the buffer directly:

Vim 的编辑器模式提供语法高亮和缩进，通过前文所述的命令提供交互式编辑的支持。
交互式编辑使用以下编辑器命令来进行，每一条都会直接更新缓冲区：

.. - ``\d`` adds a template definition for the name declared on the
..    current line (using ``:addclause``).

.. - ``\c`` case splits the variable at the cursor (using
..    ``:casesplit``).

.. - ``\m`` adds the missing cases for the name at the cursor (using
..    ``:addmissing``).

.. - ``\w`` adds a ``with`` clause (using ``:makewith``).

.. - ``\o`` invokes a proof search to solve the hole under the
..    cursor (using ``:proofsearch``).

.. - ``\p`` invokes a proof search with additional hints to solve the
..    hole under the cursor (using ``:proofsearch``).

- ``\d`` 使用 ``:addclause`` 为当前行声明的名字添加模版定义。

- ``\c`` 使用 ``:casesplit`` 为光标处的变量执行情况拆分。

- ``\m`` 使用 ``:addmissing`` 为光标处的名字添加缺少的情况。

- ``\w`` 使用 ``:makewith`` 添加 ``with`` 从句。

- ``\o`` 使用 ``:proofsearch`` 调用证明搜索来解决光标处的坑。

- ``\p`` 使用 ``:proofsearch`` 根据附加的提示调用证明搜索以解决光标处的坑。

.. There are also commands to invoke the type checker and evaluator:

.. - ``\t`` displays the type of the (globally visible) name under the
..    cursor. In the case of a hole, this displays the context
..    and the expected type.

.. - ``\e`` prompts for an expression to evaluate.

.. - ``\r`` reloads and type checks the buffer.

还有一些用来调用类型检查器与求值器的命令：

- ``\t`` 显示光标下（全局可见的）名称的类型。对坑而言，它会显示其上下文和预期的类型。

- ``\e`` 提醒要求值的表达式。

- ``\r`` 重新加载缓冲区并执行类型检查。

.. Corresponding commands are also available in the Emacs mode. Support
.. for other editors can be added in a relatively straightforward manner
.. by using ``idris –client``.

对应的命令在 Emacs 模式中也可用。其它编辑器的支持可通过使用 ``idris –client``
以相对直接的方式来添加。
