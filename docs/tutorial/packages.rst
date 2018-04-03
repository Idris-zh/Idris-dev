.. _sect-packages:

**
包
**

.. ********
.. Packages
.. ********

.. Idris includes a simple build system for building packages and executables
.. from a named package description file. These files can be used with the
.. Idris compiler to manage the development process .

Idris 包括一套简单的构建系统，它会根据已命名的包描述文件来构建包以及可执行文件。
描述文件可以配合 Idris 编译器来管理开发过程。

包的描述
========

.. Package Descriptions
.. ====================

.. A package description includes the following:

.. + A header, consisting of the keyword ``package`` followed by a package
..   name. Package names can be any valid Idris identifier. The iPKG
..   format also takes a quoted version that accepts any valid filename.

.. + Fields describing package contents, ``<field> = <value>``.

包的描述包含以下内容：

+ 包头，由关键字 ``package`` 后跟一个包名构成。包名可以是任何有效的 Idris 标识符。
  iPKG 格式也可包含一个带引号的 ``version``，它接受任何有效的文件名。

+ 描述包内容的字段，``<field> = <value>``

.. At least one field must be the modules field, where the value is a
.. comma separated list of modules. For example, given an idris package
.. ``maths`` that has modules ``Maths.idr``, ``Maths.NumOps.idr``,
.. ``Maths.BinOps.idr``, and ``Maths.HexOps.idr``, the corresponding
.. package file would be:

其中至少有一个 modules 字段，对应的值为逗号分隔的模块列表。例如，
给定一个 Idris 包 ``maths``，包含 ``Maths.idr`` 、``Maths.NumOps.idr``
``Maths.BinOps.idr`` 和  ``Maths.HexOps.idr`` 几个模块，其相应的包文件应为：

::

    package maths

    modules = Maths
            , Maths.NumOps
            , Maths.BinOps
            , Maths.HexOps

.. Other examples of package files can be found in the ``libs`` directory
.. of the main Idris repository, and in `third-party libraries
.. <https://github.com/idris-lang/Idris-dev/wiki/Libraries>`_.

其它包文件的例子可以在主 Idris 仓库的 ``libs`` 目录中以及
`第三方库 <https://github.com/Idris-zh/Idris-dev/wiki/Libraries>`_ 中找到。

使用包文件
===========

.. Using Package files
.. ===================

.. Idris itself is aware about packages, and special commands are
.. available to help with, for example, building packages, installing
.. packages, and cleaning packages.  For instance, given the ``maths``
.. package from earlier we can use Idris as follows:

.. + ``idris --build maths.ipkg`` will build all modules in the package

.. + ``idris --install maths.ipkg`` will install the package, making it
..   accessible by other Idris libraries and programs.

.. + ``idris --clean maths.ipkg`` will delete all intermediate code and
..   executable files generated when building.

Idris 本身是知晓包的，还有一些专门的命令来帮助构建、安装以及清除包。
例如，对于前面给出的 ``maths`` 包，我们可以像下面这样使用 Idris：

+ ``idris --build maths.ipkg`` 会构建包中的所有模块。

+ ``idris --install maths.ipkg`` 会安装包，使其可以被其它 Idris 库和程序访问。

+ ``idris --clean maths.ipkg`` 会删除构建时候产生的所有代码及可执行文件。

.. Once the maths package has been installed, the command line option
.. ``--package maths`` makes it accessible (abbreviated to ``-p maths``).
.. For example:

一旦 maths 包安装完成，命令行选项 ``--package maths``
（简写为 ``-p maths``）就可以使用了。例如：

::

    idris -p maths Main.idr

测试 Idris 包
==============

.. Testing Idris Packages
.. ======================

.. The integrated build system includes a simple testing framework.
.. This framework collects functions listed in the ``ipkg`` file under ``tests``.
.. All test functions must return ``IO ()``.

集成的构建系统包含简单的测试框架。该框架会收集 ``tests`` 下的 ``ipkg``
文件中列出的函数。所有的测试函数必须返回 ``IO ()`` 。

.. When you enter ``idris --testpkg yourmodule.ipkg``,
.. the build system creates a temporary file in a fresh environment on your machine
.. by listing the ``tests`` functions under a single ``main`` function.
.. It compiles this temporary file to an executable and then executes it.

当输入 ``idris --testpkg yourmodule.ipkg`` 后，构建系统会列出单个 ``main``
函数下的 ``tests`` 函数，并在你机器上的全新环境中创建一个临时文件。
它会将该临时文件编译成可执行文件并执行它。

.. The tests themselves are responsible for reporting their success or failure.
.. Test functions commonly use ``putStrLn`` to report test results.
.. The test framework does not impose any standards for reporting and consequently
.. does not aggregate test results.

测试本身负责报告它们的成功或失败。测试函数通常用 ``putStrLn`` 报告测试结果。
测试框架不强加任何报告标准，因此也不会合计测试结果。

.. For example, lets take the following list of functions that are defined in a
.. module called ``NumOps`` for a sample package ``maths``:

我们以下面的函数列表为例，它们在样本包 ``maths`` 中名为 ``NumOps`` 的模块内定义：

.. name: Math/NumOps.idr
.. code-block:: idris

    module Maths.NumOps

    %access export -- to make functions under test visible

    double : Num a => a -> a
    double a = a + a

    triple : Num a => a -> a
    triple a = a + double a

.. A simple test module, with a qualified name of ``Test.NumOps`` can be declared as:

一个限定名为 ``Test.NumOps`` 的简单测试模块可声明为：

.. name: Math/TestOps.idr
.. code-block:: idris

    module Test.NumOps

    import Maths.NumOps

    %access export  -- to make the test functions visible

    assertEq : Eq a => (given : a) -> (expected : a) -> IO ()
    assertEq g e = if g == e
        then putStrLn "Test Passed"
        else putStrLn "Test Failed"

    assertNotEq : Eq a => (given : a) -> (expected : a) -> IO ()
    assertNotEq g e = if not (g == e)
        then putStrLn "Test Passed"
        else putStrLn "Test Failed"

    testDouble : IO ()
    testDouble = assertEq (double 2) 4

    testTriple : IO ()
    testTriple = assertNotEq (triple 2) 5

.. The functions ``assertEq`` and ``assertNotEq`` are used to run expected passing,
.. and failing, equality tests. The actual tests are ``testDouble`` and ``testTriple``,
.. and are declared in the ``maths.ipkg`` file as follows:

函数 ``assertEq`` 和 ``assertNotEq`` 分别用于运行预期为通过和失败的相等性测试。
实际上的测试为 ``testDouble`` 和 ``testTriple``，它们在 ``maths.ipkg`` 文件中的声明如下

::

    package maths

    modules = Maths.NumOps
            , Test.NumOps

    tests = Test.NumOps.testDouble
          , Test.NumOps.testTriple

.. The testing framework can then be invoked using ``idris --testpkg maths.ipkg``:

测试框架可通过 ``idris --testpkg maths.ipkg`` 命令调用：

::

    > idris --testpkg maths.ipkg
    Type checking ./Maths/NumOps.idr
    Type checking ./Test/NumOps.idr
    Type checking /var/folders/63/np5g0d5j54x1s0z12rf41wxm0000gp/T/idristests144128232716531729.idr
    Test Passed
    Test Passed

.. Note how both tests have reported success by printing ``Test Passed``
.. as we arranged for with the ``assertEq`` and ``assertNoEq`` functions.

注意当我们使用 ``assertEq`` 和 ``assertNoEq`` 函数测试时，它们是如何通过打印
``Test Passed`` 来报告测试成功的。

在 Atom 中使用包依赖
=====================

.. Package Dependencies Using Atom
.. ===============================

.. If you are using the Atom editor and have a dependency on another package,
.. corresponding to for instance ``import Lightyear`` or ``import Pruviloj``,
.. you need to let Atom know that it should be loaded. The easiest way to
.. accomplish that is with a .ipkg file. The general contents of an ipkg file
.. will be described in the next section of the tutorial, but for now here is
.. a simple recipe for this trivial case:

如果你在使用 Atom 编辑器，并且依赖了另一个包，例如 ``import Lightyear``
或者 ``import Pruviloj``，那么你需要让 Atom 知道它应该加载什么。最简单的方式是通过
.ipkg 文件来完成。本教程在下一节中会描述一个 ipkg 文件通常会包含哪些内容，
不过这里先给出此平凡例子的简单步骤：

.. - Create a folder myProject.

.. - Add a file myProject.ipkg containing just a couple of lines:

.. .. code-block:: idris

..     package myProject

..     pkgs = pruviloj, lightyear

.. - In Atom, use the File menu to Open Folder myProject.

- 创建一个 myProject 文件夹。

- 添加一个 myProject.ipkg 文件，包含如下代码：

.. code-block:: idris

    package myProject

    pkgs = pruviloj, lightyear

- 在 Atom 中，使用文件菜单打开 myProject 文件夹。

更多信息
========

.. More information
.. ================

.. More details, including a complete listing of available fields, can be
.. found in the reference manual in :ref:`ref-sect-packages`.

更多详情，包括可用字段的完整列表，可在参考手册的 :ref:`ref-sect-packages` 中找到。
