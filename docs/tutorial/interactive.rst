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
中为期望的行为添加更精确的描述，为函数的正确性增加额外的信心。
我们也见识过类型系统如何帮助开发 EDSL 了，它允许程序员来描述目标语言的类型系统。
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

Idris 的 REPL 提供了一些命令，可基于程序的类型来检查和修改程序的部分，
如在模式变量中拆分情况，检查坑的类型，甚至基本的证明搜索机制。在本节中，
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
每个命令都有一种就地 **更新** 源文件的形式：

::

    :命令! [行号] [名称]

.. When the REPL is loaded, it also starts a background process which
.. accepts and responds to REPL commands, using ``idris --client``. For
.. example, if we have a REPL running elsewhere, we can execute commands
.. such as:

当 REPL 被加载时，它还会用 ``idris --client`` 在后台启动一个进程，该进程接受并相应
REPL 命令。例如，如果我们在某处运行着 REPL，就可以执行这样的命令：

::

    $ idris --client ':t plus'
    Prelude.Nat.plus : Nat -> Nat -> Nat
    $ idris --client '2+2'
    4 : Integer

.. A text editor can take advantage of this, along with the editing
.. commands, in order to provide interactive editing support.

文本编辑器可以利用这一点和编辑命令来提供交互式编辑。

编辑命令
========

.. Editing Commands
.. ================

:addclause
----------

.. The ``:addclause n f`` command (abbreviated ``:ac n f``) creates a
.. template definition for the function named ``f`` declared on line
.. ``n``.  For example, if the code beginning on line 94 contains:

``:addclause n f`` 命令，缩写为 ``:ac n f``，它为声明在第 ``n`` 行的函数 ``f``
创建一个模版定义。例如，若从第 94 行开始的代码包含：

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

名字会根据可能是程序员给定的提示来选择，必要时机器会添加数字使其唯一。
提示可像下面这样给定：

.. code-block:: idris

    %name Vect xs, ys, zs, ws

.. This declares that any names generated for types in the ``Vect`` family
.. should be chosen in the order ``xs``, ``ys``, ``zs``, ``ws``.

它声明了为 ``Vect`` 族的类型生成的名字应按照 ``xs``、``ys``、``zs``、``ws``
的顺序来选取。

:casesplit
----------

.. The ``:casesplit n x`` command, abbreviated ``:cs n x``, splits the
.. pattern variable ``x`` on line ``n`` into the various pattern forms it
.. may take, removing any cases which are impossible due to unification
.. errors. For example, if the code beginning on line 94 is:

``:casesplit n x`` 命令，缩写为 ``:cs n x``，它将第 ``n`` 行的模式变量 ``x``
拆分为能够形成它的多种模式，移除任何由于一致性错误而不可能出现的情况。
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

即，模式变量 ``xs`` 被拆分成了 ``[]`` 和 ``x :: xs`` 两种可能的情况。和之前一样，
名字的选取启发自相同的规则。若我们（用 ``:cs!``）更新文件后再拆分同一行的
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

The ``:addmissing n f`` command, abbreviated ``:am n f``, adds the
clauses which are required to make the function ``f`` on line ``n``
cover all inputs. For example, if the code beginning on line 94 is

``:addmissing n f`` 命令，缩写为 ``:am n f``，它为第 ``n`` 行的函数 ``f``
添加使其覆盖所有输入的从句。

.. code-block:: idris

    vzipWith : (a -> b -> c) ->
               Vect n a -> Vect n b -> Vect n c
    vzipWith f [] [] = ?vzipWith_rhs_1

then ``:am 96 vzipWith`` gives:

.. code-block:: idris

    vzipWith f (x :: xs) (y :: ys) = ?vzipWith_rhs_2

That is, it notices that there are no cases for non-empty vectors,
generates the required clauses, and eliminates the clauses which would
lead to unification errors.

:proofsearch
------------

The ``:proofsearch n f`` command, abbreviated ``:ps n f``, attempts to
find a value for the hole ``f`` on line ``n`` by proof search,
trying values of local variables, recursive calls and constructors of
the required family. Optionally, it can take a list of *hints*, which
are functions it can try applying to solve the hole. For
example, if the code beginning on line 94 is:

.. code-block:: idris

    vzipWith : (a -> b -> c) ->
               Vect n a -> Vect n b -> Vect n c
    vzipWith f [] [] = ?vzipWith_rhs_1
    vzipWith f (x :: xs) (y :: ys) = ?vzipWith_rhs_2

then ``:ps 96 vzipWith_rhs_1`` will give

.. code-block:: idris

    []

This works because it is searching for a ``Vect`` of length 0, of
which the empty vector is the only possibility. Similarly, and perhaps
surprisingly, there is only one possibility if we try to solve ``:ps
97 vzipWith_rhs_2``:

.. code-block:: idris

    f x y :: (vzipWith f xs ys)

This works because ``vzipWith`` has a precise enough type: The
resulting vector has to be non-empty (a ``::``); the first element
must have type ``c`` and the only way to get this is to apply ``f`` to
``x`` and ``y``; finally, the tail of the vector can only be built
recursively.

:makewith
---------

The ``:makewith n f`` command, abbreviated ``:mw n f``, adds a
``with`` to a pattern clause. For example, recall ``parity``. If line
10 is:

.. code-block:: idris

    parity (S k) = ?parity_rhs

then ``:mw 10 parity`` will give:

.. code-block:: idris

    parity (S k) with (_)
      parity (S k) | with_pat = ?parity_rhs

If we then fill in the placeholder ``_`` with ``parity k`` and case
split on ``with_pat`` using ``:cs 11 with_pat`` we get the following
patterns:

.. code-block:: idris

      parity (S (plus n n)) | even = ?parity_rhs_1
      parity (S (S (plus n n))) | odd = ?parity_rhs_2

Note that case splitting has normalised the patterns here (giving
``plus`` rather than ``+``). In any case, we see that using
interactive editing significantly simplifies the implementation of
dependent pattern matching by showing a programmer exactly what the
valid patterns are.

Vim 交互式编辑
==============

.. Interactive Editing in Vim
.. ==========================

The editor mode for Vim provides syntax highlighting, indentation and
interactive editing support using the commands described above.
Interactive editing is achieved using the following editor commands,
each of which update the buffer directly:

- ``\d`` adds a template definition for the name declared on the
   current line (using ``:addclause``).

- ``\c`` case splits the variable at the cursor (using
   ``:casesplit``).

- ``\m`` adds the missing cases for the name at the cursor (using
   ``:addmissing``).

- ``\w`` adds a ``with`` clause (using ``:makewith``).

- ``\o`` invokes a proof search to solve the hole under the
   cursor (using ``:proofsearch``).

- ``\p`` invokes a proof search with additional hints to solve the
   hole under the cursor (using ``:proofsearch``).

There are also commands to invoke the type checker and evaluator:

- ``\t`` displays the type of the (globally visible) name under the
   cursor. In the case of a hole, this displays the context
   and the expected type.

- ``\e`` prompts for an expression to evaluate.

- ``\r`` reloads and type checks the buffer.

Corresponding commands are also available in the Emacs mode. Support
for other editors can be added in a relatively straightforward manner
by using ``idris –client``.
