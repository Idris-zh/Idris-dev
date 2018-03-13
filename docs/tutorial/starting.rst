.. _sect-starting:

****
入门
****

.. ***************
.. Getting Started
.. ***************

前提
====

.. Prerequisites
.. =============

.. Before installing Idris, you will need to make sure you have all
.. of the necessary libraries and tools. You will need:

在安装 Idris 之前，你需要确认是否拥有所有必须的库和工具。你需要：

.. - A fairly recent version of `GHC <https://www.haskell.org/ghc/>`_. The
..   earliest version we currently test with is 7.10.3.

- 近期版本的 `GHC <https://www.haskell.org/ghc/>`_ 。当前测试过的最早版本为 7.10.3.

.. - The *GNU Multiple Precision Arithmetic Library* (GMP) is available
..   from MacPorts/Homebrew and all major Linux distributions.

- **GNU 多精度运算库** (GMP) 可从 MacPorts/Homebrew 和所有主流 Linux 发行版中获取。

下载并安装
==========

.. Downloading and Installing
.. ==========================

.. The easiest way to install Idris, if you have all of the
.. prerequisites, is to type:

如果你满足所有的前提，那么安装 Idris 最简单的方式就是输入：

::

    cabal update; cabal install idris

.. This will install the latest version released on Hackage, along with
.. any dependencies. If, however, you would like the most up to date
.. development version you can find it, as well as build instructions, on
.. GitHub at: https://github.com/idris-lang/Idris-dev.

这会安装 Hackage 中的最新版本及其所有依赖。如果你想要最新开发版的话，
可以在 `GitHub <https://github.com/idris-lang/Idris-dev>`_ 上找到它，
然后根据构建指令来安装。

.. If you haven't previously installed anything using Cabal, then Idris
.. may not be on your path. Should the Idris executable not be found
.. please ensure that you have added ``~/.cabal/bin`` to your ``$PATH``
.. environment variable. Mac OS X users may find they need to add
.. ``~/Library/Haskell/bin`` instead, and Windows users will typically
.. find that Cabal installs programs in ``%HOME%\AppData\Roaming\cabal\bin``.

如果你之前从未安装过使用 Cabal 的东西，Idris 可能不在你的路径中。
如果 Idris 可执行文件无法找到，请将 ``~/.cabal/bin`` 添加到 ``$PATH``
环境变量中。Mac OS X 用户则需要添加 ``~/Library/Haskell/bin``，Windows
用户通常会发现 Cabal 将程序安装在 ``%HOME%\AppData\Roaming\cabal\bin`` 中。

.. To check that installation has succeeded, and to write your first
.. Idris program, create a file called ``hello.idr`` containing the
.. following text:

为了检查是否安装成功，你需要编写第一个程序。请创建一个名为 ``hello.idr`` 的文件，
其中包含以下文本：

.. code-block:: idris

    module Main

    main : IO ()
    main = putStrLn "Hello world"

.. If you are familiar with Haskell, it should be fairly clear what the
.. program is doing and how it works, but if not, we will explain the
.. details later. You can compile the program to an executable by
.. entering ``idris hello.idr -o hello`` at the shell prompt. This will
.. create an executable called ``hello``, which you can run:

如果你熟悉 Haskell，那会很清楚这段程序在做什么，是如何做的；
如果你对它感到陌生，我们稍后会详细地解释。你可以在命令行中输入
``idris hello.idr -o hello`` 来将程序编译成可执行文件。
这会创建一个名为 ``hello`` 的可执行文件，你可以运行它：

::

    $ idris hello.idr -o hello
    $ ./hello
    Hello world

.. Please note that the dollar sign ``$`` indicates the shell prompt!
.. Some useful options to the Idris command are:

.. - ``-o prog`` to compile to an executable called ``prog``.

.. - ``--check`` type check the file and its dependencies without starting the interactive environment.

.. - ``--package pkg`` add package as dependency, e.g. ``--package contrib`` to make use of the contrib package.

.. - ``--help`` display usage summary and command line options.

请注意，美元符号 ``$`` 表示命令行。下面是一些常用的 Idris 命令行选项：

- ``-o prog`` 编译成名为 ``prog`` 的可执行文件。

- ``--check`` 无需启动交互式环境就对文件及其依赖进行类型检查。

- ``--package pkg`` 将包添加为依赖，例如通过 ``--package contrib`` 来使用 contrib 包。

- ``--help`` 显示用法摘要和命令行选项。

交互式环境
==========

.. The Interactive Environment
.. ===========================

.. Entering ``idris`` at the shell prompt starts up the interactive
.. environment. You should see something like the following:

在命令行中输入 ``idris`` 来启动交互式环境。你会看到如下内容：

.. literalinclude:: ../listing/idris-prompt-start.txt

.. This gives a ``ghci`` style interface which allows evaluation of, as
.. well as type checking of, expressions; theorem proving, compilation;
.. editing; and various other operations. The command ``:?`` gives a list
.. of supported commands. Below, we see an example run in
.. which ``hello.idr`` is loaded, the type of ``main`` is checked and
.. then the program is compiled to the executable ``hello``. Type
.. checking a file, if successful, creates a bytecode version of the file
.. (in this case ``hello.ibc``) to speed up loading in future. The
.. bytecode is regenerated if the source file changes.

它会提供一个 ``ghci`` 风格的界面，可以：像类型检查那样求值表达式、进行定理证明、
编译、编辑、以及多种其它操作。命令 ``:?`` 会列出所支持的命令。在以下示例中
``hello.idr`` 已被加载，``main`` 的类型已通过检查，之后该程序被编译成了可执行的 ``hello``。
在对某文件类型检查时，如果通过，就会创建它的的字节码版本（本例中为 ``hello.ibc``）
以提升未来的加载速度。在源文件被修改之后，字节码会重新生成。

.. _run1:
.. literalinclude:: ../listing/idris-prompt-helloworld.txt
