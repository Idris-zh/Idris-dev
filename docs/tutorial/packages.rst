****
包
****

.. ********
.. Packages
.. ********


.. Idris includes a simple build system for building packages and executables from a named package description file.
.. These files can be used with the Idris compiler to manage the development process .

Idris 程序包含一个用来构建包的简单构建系统和来自一个命名包描述文件的一些可执行文件。
这些文件可以与 Idris 编译器一起使用来管理开发进程。

包描述
=======

.. Package Descriptions
.. ====================

.. A package description includes the following:
.. 
.. + A header, consisting of the keyword package followed by the package
..   name. Package names can be any valid Idris identifier. The iPKG
..   format also takes a quoted version that accepts any valid filename.
.. + Fields describing package contents, ``<field> = <value>``

包描述包含以下内容：

+ 包头，由跟在包名后面的关键词包组成。包名可以是任何有效的 Idris 标识符。
  iPKG 格式还可以是接受任何有效文件名的带引号版本。
+ 描述包内容的字段，``<field> = <value>``

.. At least one field must be the modules field, where the value is a
.. comma separated list of modules.  For example, given an idris package
.. ``maths`` that has modules ``Maths.idr``, ``Maths.NumOps.idr``,
.. ``Maths.BinOps.idr``, and ``Maths.HexOps.idr``, the corresponding
.. package file would be::

至少一个字段是 modules 字段，对应的值是通过逗号分割的模块列表。例如，
给定一个 idris ``maths`` 包，有 ``Maths.idr`` 、``Maths.NumOps.idr`` 、
``Maths.BinOps.idr`` 和  ``Maths.HexOps.idr`` 模块，相应的包文件会是：

    package maths

    modules = Maths
            , Maths.NumOps
            , Maths.BinOps
            , Maths.HexOps


.. Other examples of package files can be found in the ``libs`` directory
.. of the main Idris repository, and in `third-party libraries
.. <https://github.com/idris-lang/Idris-dev/wiki/Libraries>`_.

其他包文件的例子可以在主 Idris 仓库的 ``libs`` 目录下和 `第三方库 <https://github.com/idris-lang/Idris-dev/wiki/Libraries>`_ 里找到。

使用包文件
===========

.. Using Package files
.. ===================
.. 
.. 
.. Idris itself is aware about packages, and special commands are
.. available to help with, for example, building packages, installing
.. packages, and cleaning packages.  For instance, given the ``maths``
.. package from earlier we can use Idris as follows:

.. + ``idris --build maths.ipkg`` will build all modules in the package
.. 
.. + ``idris --install maths.ipkg`` will install the package, making it
..   accessible by other Idris libraries and programs.
.. 
.. + ``idris --clean maths.ipkg`` will delete all intermediate code and
..   executable files generated when building.
.. 
.. Once the maths package has been installed, the command line option
.. ``--package maths`` makes it accessible (abbreviated to ``-p maths``).
.. For example::
.. 
..     idris -p maths Main.idr

Idris 本身是知晓包的，还有一些专门的命令来帮助了解，例如，构建包、安装包和清除包。
举例说明，前面给定的 ``maths`` 包，我们可以像下面这样使用 Idris：

+ ``idris --build maths.ipkg`` 将构建包里的所有模块

+ ``idris --install maths.ipkg`` 会安装包，使其可有其他 Idris 库和程序可见。 

+ ``idris --clean maths.ipkg`` 将删除构建时候产生的所有代码和执行文件。

一旦 maths 包安装完成，命令行选项 ``--package maths`` 就可使用了，
（可以简写为 ``-p maths``）。
例如::

    idris -p maths Main.idr

Idris 测试包
==============

.. Testing Idris Packages
.. ======================
.. 
.. The integrated build system includes a simple testing framework.
.. This framework collects functions listed in the ``ipkg`` file under ``tests``.
.. All test functions must return ``IO ()``.

集成的构建系统有一个简单的测试框架。这个框架收集列在 ``tests`` 路径下 ``ipkg`` 文件里的
函数。所有测试函数必须返回 ``IO ()`` 。

当键入 ``idris --testpkg yourmodule.ipkg`` ，构建系统会通过列出在单一 ``main`` 函数
下 ``tests`` 函数，在机器上的一个全新环境里创建一个临时文件。构建系统把这个临时文件编译成可执行，然后执行它。

.. When you enter ``idris --testpkg yourmodule.ipkg``,
.. the build system creates a temporary file in a fresh environment on your machine
.. by listing the ``tests`` functions under a single ``main`` function.
.. It compiles this temporary file to an executable and then executes it.


.. The tests themselves are responsible for reporting their success or failure.
.. Test functions commonly use ``putStrLn`` to report test results.
.. The test framework does not impose any standards for reporting and consequently
.. does not aggregate test results.

测试本身负责报告它们的成功或失败。测试函数一般使用 ``putStrLn`` 报告测试结果。
测试框架不强加任何报告标准, 因而不会聚合测试结果。

例如，我们取出以下函数列表，这些函数定义在样本包 ``maths`` 下的一个叫做 ``NumOps`` 的模块里。

.. For example, lets take the following list of functions that are defined in a module called ``NumOps`` for a sample package ``maths``.

.. name: Math/NumOps.idr
.. code-block:: idris

    module Maths.NumOps

    %access export -- to make functions under test visible

    double : Num a => a -> a
    double a = a + a

    triple : Num a => a -> a
    triple a = a + double a

一个简单的测试模块，具有限定名 ``Test.NumOps`` 的模块可以声明为

.. A simple test module, with a qualified name of ``Test.NumOps`` can be declared as

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

.. The functions ``assertEq`` and ``assertNotEq`` are used to run expected passing, and failing, equality tests.
.. The actual tests are ``testDouble`` and ``testTriple``, and are declared in the ``maths.ipkg`` file as follows::

函数  ``assertEq`` 和 ``assertNotEq`` 用于运行预期的通过、失败和相等测试。
实际上的测试是 ``testDouble`` 和 ``testTriple`` ，在文件 ``maths.ipkg`` 中声明如下::

    package maths

    modules = Maths.NumOps
            , Test.NumOps

    tests = Test.NumOps.testDouble
          , Test.NumOps.testTriple

.. The testing framework can then be invoked using ``idris --testpkg maths.ipkg``::
可以使用 ``idris --testpkg maths.ipkg`` 命令调用测试框架::

    > idris --testpkg maths.ipkg
    Type checking ./Maths/NumOps.idr
    Type checking ./Test/NumOps.idr
    Type checking /var/folders/63/np5g0d5j54x1s0z12rf41wxm0000gp/T/idristests144128232716531729.idr
    Test Passed
    Test Passed

.. Note how both tests have reported success by printing ``Test Passed``
.. as we arranged for with the ``assertEq`` and ``assertNoEq`` functions.

注意，当我们使用 ``assertEq`` 和 ``assertNoEq`` 函数测试时， 
两个测试是怎样通过打印 ``Test Passed`` 报告测试成功。

在 Atom 中使用包依赖
=====================

.. Package Dependencies Using Atom 
.. ===============================
.. 
.. If you are using the Atom editor and have a dependency on another package, 
.. corresponding to for instance ``import Lightyear`` or ``import Pruviloj``, 
.. you need to let Atom know that it should be loaded. The easiest way to 
.. accomplish that is with a .ipkg file. The general contents of an ipkg file 
.. will be described in the next section of the tutorial, but for now here is 
.. a simple recipe for this trivial case. 

如果你使用的是 Atom 编辑器，并且对另外一个包有依赖关系，例如对应的是 ``import Lightyear`` 
或者 ``import Pruviloj`` ，你需要让 Atom 知道它应该加载什么。最简单的方式是使用 .ipkg 
文件来完成。我们会在教程的下一部分叙述一个 ipkg 文件的一般内容是什么，不过这里会给出一个
明显例子的简单窍门。

.. - Create a folder myProject. 
.. 
.. - Add a file myProject.ipkg containing just a couple of lines: 
.. 
.. ``package myProject`` 
.. 
.. ``pkgs = pruviloj, lightyear`` 
.. 
.. - In Atom, use the File menu to Open Folder myProject. 

- 创建一个 myProject 文件夹。

- 添加包含如下几行代码的 myProject.ipkg 文件: 

``package myProject`` 

``pkgs = pruviloj, lightyear`` 

- 在 Atom 中，使用文件菜单打开 myProject 文件夹。

更多信息
==========

包含可用字段的一个完全列表的更多详情，可以在参考文档 :ref:`ref-sect-packages` 找到。

.. More information
.. ================
.. 
.. More details, including a complete listing of available fields, can be
.. found in the reference manual in :ref:`ref-sect-packages`.
