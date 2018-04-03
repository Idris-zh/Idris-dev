.. _sect-misc:

****
杂项
****

.. **********
.. Miscellany
.. **********

.. In this section we discuss a variety of additional features:

.. + auto, implicit, and default arguments;
.. + literate programming;
.. + interfacing with external libraries through the foreign function;
.. + interface;
.. + type providers;
.. + code generation; and
.. + the universe hierarchy.

在本节中，我们讨论了多种附加特性：

+ 自动、隐式与默认参数
+ 文学编程（Literate Programming）
+ 通过外部函数与外部库交互
+ 接口
+ 类型提供器（Type Provider）
+ 代码生成，以及
+ 全域层级（Universe Hierarchy）

隐式参数
========

.. Implicit arguments
.. =======================

.. We have already seen implicit arguments, which allows arguments to be
.. omitted when they can be inferred by the type checker, e.g.

我们已经见过隐式参数了，它允许忽略能够被类型检查器推断出来的参数，例如：

.. code-block:: idris

    index : {a:Type} -> {n:Nat} -> Fin n -> Vect n a -> a

自动隐式参数
------------

.. Auto implicit arguments
.. ------------------------

.. In other situations, it may be possible to infer arguments not by type
.. checking but by searching the context for an appropriate value, or
.. constructing a proof. For example, the following definition of ``head``
.. which requires a proof that the list is non-empty:

在其它情况下，可以不通过类型检查来推断参数，而是通过在上下文中搜索恰当的值，
或通过构造证明来推断参数。例如，以下 ``head`` 的定义需要证明列表非空：

.. code-block:: idris

    isCons : List a -> Bool
    isCons [] = False
    isCons (x :: xs) = True

    head : (xs : List a) -> (isCons xs = True) -> a
    head (x :: xs) _ = x

.. If the list is statically known to be non-empty, either because its
.. value is known or because a proof already exists in the context, the
.. proof can be constructed automatically. Auto implicit arguments allow
.. this to happen silently. We define ``head`` as follows:

如果可以静态地获知列表非空，那是由于它的值是已知的，或上下文中存在对它的证明。
证明可以自动地构造，自动隐式参数允许它静默地发生。我们将 ``head`` 定义为：

.. code-block:: idris

    head : (xs : List a) -> {auto p : isCons xs = True} -> a
    head (x :: xs) = x

.. The ``auto`` annotation on the implicit argument means that Idris
.. will attempt to fill in the implicit argument by searching for a value
.. of the appropriate type. It will try the following, in order:

将隐式参数注解为 ``auto``，表示 Idris 会试图搜索与其类型对应的值来填充它。
它会按照以下顺序尝试：

.. - Local variables, i.e. names bound in pattern matches or ``let`` bindings,
..   with exactly the right type.
.. - The constructors of the required type. If they have arguments, it will
..   search recursively up to a maximum depth of 100.
.. - Local variables with function types, searching recursively for the
..   arguments.
.. - Any function with the appropriate return type which is marked with the
..   ``%hint`` annotation.

- 局部变量，即类型正确的，在模式匹配中绑定的名字或 ``let`` 绑定。
- 所需类型的构造器。若它们有参数，就会递归地搜索到最深 100 层的深度。
- 带函数类型的局部变量，对参数进行递归地搜索。
- 任何标出 ``%hint`` 注解的，带有相应返回类型的函数。

.. In the case that a proof is not found, it can be provided explicitly as normal:

在找不到证明的情况下，它会像往常一样被显式地给出：

.. code-block:: idris

    head xs {p = ?headProof}

定义隐式参数
------------

.. Default implicit arguments
.. ---------------------------

.. Besides having Idris automatically find a value of a given type, sometimes we
.. want to have an implicit argument with a specific default value. In Idris, we can
.. do this using the ``default`` annotation. While this is primarily intended to assist
.. in automatically constructing a proof where auto fails, or finds an unhelpful value,
.. it might be easier to first consider a simpler case, not involving proofs.

除了让 Idris 自动查找给定类型的值外，有时我们还想要带有具体默认值的隐式参数。在
Idris 中，我们可以用 ``default`` 注解来做这件事。尽管它主要是为了在 auto
自动构造证明失败，或找到的值没有帮助时起辅助作用。然而，首先考虑一个更简单的，
不涉及证明的情况可能会更容易。

.. If we want to compute the n'th fibonacci number (and defining the 0th fibonacci
.. number as 0), we could write:

如果我们想要计算第 n 个斐波那契数（第 0 个斐波那契数定义为 0），我们可以写：

.. code-block:: idris

	fibonacci : {default 0 lag : Nat} -> {default 1 lead : Nat} -> (n : Nat) -> Nat
	fibonacci {lag} Z = lag
	fibonacci {lag} {lead} (S n) = fibonacci {lag=lead} {lead=lag+lead} n

.. After this definition, ``fibonacci 5`` is equivalent to ``fibonacci {lag=0} {lead=1} 5``,
.. and will return the 5th fibonacci number. Note that while this works, this is not the
.. intended use of the ``default`` annotation. It is included here for illustrative purposes
.. only. Usually, ``default`` is used to provide things like a custom proof search script.

定义完之后，``fibonacci 5`` 等价于 ``fibonacci {lag=0} {lead=1} 5``，它会返回第 5
个斐波那契数。注意虽然它可以工作，但这并不是 ``default`` 注解的用途，在这里它只用作展示。
通常，``default`` 用于提供定制证明搜索脚本的东西。

隐式转换
========

.. Implicit conversions
.. ====================

.. Idris supports the creation of *implicit conversions*, which allow
.. automatic conversion of values from one type to another when required to
.. make a term type correct. This is intended to increase convenience and
.. reduce verbosity. A contrived but simple example is the following:

Idris 支持创建 **隐式转换** ，在需要某项的类型能够匹配时，
它允许将一个类型的值自动转换成另一个类型。这是为了增强便利性并减少冗余。
下面是个故意构造的简单例子：

.. code-block:: idris

    implicit intString : Int -> String
    intString = show

    test : Int -> String
    test x = "Number " ++ x

.. In general, we cannot append an ``Int`` to a ``String``, but the
.. implicit conversion function ``intString`` can convert ``x`` to a
.. ``String``, so the definition of ``test`` is type correct. An implicit
.. conversion is implemented just like any other function, but given the
.. ``implicit`` modifier, and restricted to one explicit argument.

通常，我们无法将一个 ``Int`` 附加到一个 ``String`` 之后，不过隐式转换函数
``intString`` 可以将 ``x`` 转换为 ``String``，因此 ``test`` 定义的类型是正确的。
隐式转换的实现和其它函数一样，只不过加上了 ``implicit`` 修饰符，
且被限制为只能接受一个显式参数。

.. Only one implicit conversion will be applied at a time. That is,
.. implicit conversions cannot be chained. Implicit conversions of simple
.. types, as above, are however discouraged! More commonly, an implicit
.. conversion would be used to reduce verbosity in an embedded domain
.. specific language, or to hide details of a proof. Such examples are
.. beyond the scope of this tutorial.

一次只有一个隐式转换会被应用。也就是说，隐式转换无法被链式调用。如前文所见，
简单类型的隐式转换是不被鼓励的！更常见的做法是，使用隐式转换来减少 EDSL
的冗长度，或者隐藏证明的细节。这些示例超出了本教程的范围。

文学编程
========

.. Literate programming
.. ====================

.. Like Haskell, Idris supports *literate* programming. If a file has
.. an extension of ``.lidr`` then it is assumed to be a literate file. In
.. literate programs, everything is assumed to be a comment unless the line
.. begins with a greater than sign ``>``, for example:

和 Haskell 一样，Idris 支持 **文学编程**。如果某个文件的扩展名外 ``.lidr``，
那么它会被当做文学编程文件。在文学编程中，除了以大于号 ``>`` 开头的行外，
所有的内容都会被视为注释。例如：

.. .. code-block:: idris

..     > module literate

..     This is a comment. The main program is below

..     > main : IO ()
..     > main = putStrLn "Hello literate world!\n"

.. code-block:: idris

    > module literate

    这是一行注释，主程序在下面

    > main : IO ()
    > main = putStrLn "Hello literate world!\n"

.. An additional restriction is that there must be a blank line between a
.. program line (beginning with ``>``) and a comment line (beginning with
.. any other character).

附加的限制为，程序行（以 ``>`` 开头）与注释行（以任何其它字符开头）之间必须有一空行。

外部函数调用
============

.. Foreign function calls
.. ======================

.. For practical programming, it is often necessary to be able to use
.. external libraries, particularly for interfacing with the operating
.. system, file system, networking, *et cetera*. Idris provides a
.. lightweight foreign function interface for achieving this, as part of
.. the prelude. For this, we assume a certain amount of knowledge of C and
.. the ``gcc`` compiler. First, we define a datatype which describes the
.. external types we can handle:

在编程实践中，我们经常需要使用外部库，特别在与操作系统、文件系统、网络 **等等**
进行交互时。作为 Prelude 的一部分，Idris 提供了轻量的外部函数接口。
我们假定读者有一定的 C 和 ``gcc`` 编译器的知识。首先，我们来定一个数据类型，
它描述了我们能够处理的外部类型：

.. code-block:: idris

    data FTy = FInt | FFloat | FChar | FString | FPtr | FUnit

.. Each of these corresponds directly to a C type. Respectively: ``int``,
.. ``double``, ``char``, ``char*``, ``void*`` and ``void``. There is also a
.. translation to a concrete Idris type, described by the following
.. function:

它们每一个都与 C 的类型直接对应。分别为：``int``、``double``、``char``、
``char*``、``void*`` 和 ``void``。以下函数还描述了它们到对应的 Idris 类型的翻译：

.. code-block:: idris

    interpFTy : FTy -> Type
    interpFTy FInt    = Int
    interpFTy FFloat  = Double
    interpFTy FChar   = Char
    interpFTy FString = String
    interpFTy FPtr    = Ptr
    interpFTy FUnit   = ()

.. A foreign function is described by a list of input types and a return
.. type, which can then be converted to an Idris type:

外部函数由一组输入类型和返回类型描述，它们可以被转换为 Idris 类型：

.. code-block:: idris

    ForeignTy : (xs:List FTy) -> (t:FTy) -> Type

.. A foreign function is assumed to be impure, so ``ForeignTy`` builds an
.. ``IO`` type, for example:

外部函数被视作不纯粹的，因此 ``ForeignTy`` 构建了一个 ``IO`` 类型，例如：

.. code-block:: idris

    Idris> ForeignTy [FInt, FString] FString
    Int -> String -> IO String : Type

    Idris> ForeignTy [FInt, FString] FUnit
    Int -> String -> IO () : Type

.. We build a call to a foreign function by giving the name of the
.. function, a list of argument types and the return type. The built in
.. construct ``mkForeign`` converts this description to a function callable
.. by Idris:

我们通过为函数赋予名字、一系列参数的类型和返回值构建了一个外部函数调用。
内建的构造 ``mkForeign`` 将该函数的描述转换为一个可由 Idris 调用的函数：

.. code-block:: idris

    data Foreign : Type -> Type where
        FFun : String -> (xs:List FTy) -> (t:FTy) ->
               Foreign (ForeignTy xs t)

    mkForeign : Foreign x -> x

.. Note that the compiler expects ``mkForeign`` to be fully applied to
.. build a complete foreign function call. For example, the ``putStr``
.. function is implemented as follows, as a call to an external function
.. ``putStr`` defined in the run-time system:

注意编译器期望 ``mkForeign`` 能够被完全地应用以构建完整的外部函数调用。例如，
``putStr`` 作为一个在运行时系统中定义的外部函数 ``putStr`` 的调用，其实现如下：

.. code-block:: idris

    putStr : String -> IO ()
    putStr x = mkForeign (FFun "putStr" [FString] FUnit) x

include 与链接器指令
--------------------

.. Include and linker directives
.. -----------------------------

.. Foreign function calls are translated directly to calls to C functions,
.. with appropriate conversion between the Idris representation of a
.. value and the C representation. Often this will require extra libraries
.. to be linked in, or extra header and object files. This is made possible
.. through the following directives:

外部函数调用会按照 Idris 和 C 的值的表示之间对应的转换，被直接翻译为 C 函数的调用。
通常这会需要将额外的库、头文件或目标文件链接进来。我们可以通过以下指令来完成：

.. -  ``%lib target x`` — include the ``libx`` library. If the target is
..    ``C`` this is equivalent to passing the ``-lx`` option to ``gcc``. If
..    the target is Java the library will be interpreted as a
..    ``groupId:artifactId:packaging:version`` dependency coordinate for
..    maven.

.. -  ``%include target x`` — use the header file or import ``x`` for the
..    given back end target.

.. -  ``%link target x.o`` — link with the object file ``x.o`` when using
..    the given back end target.

.. -  ``%dynamic x.so`` — dynamically link the interpreter with the shared
..    object ``x.so``.

-  ``%lib target x`` — 将 ``libx`` 库包含进来。如果目标为 C，它等价于向
   ``gcc`` 传递 ``-lx`` 选项。如果目标为 Java，该库会被解释为 maven 的依赖关系定位符
   ``groupId:artifactId:packaging:version``。

-  ``%include target x`` — 使用头文件或为给定的后端目标导入 ``x``。

-  ``%link target x.o`` — 在使用给定的后端目标时链接目标文件 ``x.o``。

-  ``%dynamic x.so`` — 动态链接共享的解释器目标文件 ``x.so``。


测试外部函数调用
----------------

.. Testing foreign function calls
.. ------------------------------

.. Normally, the Idris interpreter (used for typechecking and at the REPL)
.. will not perform IO actions. Additionally, as it neither generates C
.. code nor compiles to machine code, the ``%lib``, ``%include`` and
.. ``%link`` directives have no effect. IO actions and FFI calls can be
.. tested using the special REPL command ``:x EXPR``, and C libraries can
.. be dynamically loaded in the interpreter by using the ``:dynamic``
.. command or the ``%dynamic`` directive. For example:

一般来说，Idris 解释器（用于类型检查和 REPL）不会处理 IO 活动。除此之外，它既不会生成
C 代码，也不会将它编译成机器码，因此 ``%lib``、``%include`` 与 ``%link`` 是没有效果的。
IO 活动与 FFI 调用可使用特殊的 REPL 命令 ``:x EXPR`` 来测试，而 C 库可通过 ``:dynamic``
命令或 ``%dynamic`` 指令来动态地加载到解释器中。例如：

.. code-block:: idris

    Idris> :dynamic libm.so
    Idris> :x unsafePerformIO ((mkForeign (FFun "sin" [FFloat] FFloat)) 1.6)
    0.9995736030415051 : Double

类型提供器
==========

.. Type Providers
.. ==============

.. Idris type providers, inspired by F#’s type providers, are a means of
.. making our types be “about” something in the world outside of Idris. For
.. example, given a type that represents a database schema and a query that
.. is checked against it, a type provider could read the schema of a real
.. database during type checking.

Idris 类型提供器，灵感来自 F# 的类型提供器，它能让我们的类型与 Idris
之外的世界建立「联系」。例如，给定一个表示数据库模式（Schema）的类型，
和一个针对它检查过的查询，类型提供器可以在进行类型检查时读取真实数据库的模式。

.. Idris type providers use the ordinary execution semantics of Idris to
.. run an IO action and extract the result. This result is then saved as a
.. constant in the compiled code. It can be a type, in which case it is
.. used like any other type, or it can be a value, in which case it can be
.. used as any other value, including as an index in types.

Idris 类型提供器使用普通的 Idris 可执行语义来运行 IO 活动并提取出结果。
该结果会作为编译代码时的常量被保存。它可以是个类型，此时它能像其它类型一样使用；
它也可以是个值，此时它也可以像其它值一样使用，并作为一个索引被包含在类型中。

.. Type providers are still an experimental extension. To enable the、
.. extension, use the ``%language`` directive:、

类型提供器尚且是个实验性的扩展。要启用它，请使用 ``%language`` 指令：

.. code-block:: idris

    %language TypeProviders

.. A provider ``p`` for some type ``t`` is simply an expression of type
.. ``IO (Provider t)``. The ``%provide`` directive causes the type checker
.. to execute the action and bind the result to a name. This is perhaps
.. best illustrated with a simple example. The type provider ``fromFile``
.. reads a text file. If the file consists of the string ``Int``, then the
.. type ``Int`` will be provided. Otherwise, it will provide the type
.. ``Nat``.

某个类型 ``t`` 的提供器 ``p`` 不过就是个类型为 ``IO (Provider t)`` 的表达式。
``%provide`` 指令会导致类型检查器去执行该活动，并将其结果绑定到一个名字上。
我们最好用一个简单的例子来展示它。类型提供器 ``fromFile`` 用于读取文本文件。
如果该文件由字符串 ``Int`` 构成，那么它就会提供 ``Int`` 类型。否则，它就会提供
``Nat`` 类型。

.. code-block:: idris

    strToType : String -> Type
    strToType "Int" = Int
    strToType _ = Nat

    fromFile : String -> IO (Provider Type)
    fromFile fname = do Right str <- readFile fname
		          | Left err => pure (Provide Void)
		        pure (Provide (strToType (trim str)))

.. We then use the ``%provide`` directive:

接着我们使用 ``%provide`` 指令：

.. code-block:: idris

    %provide (T1 : Type) with fromFile "theType"

    foo : T1
    foo = 2

.. If the file named ``theType`` consists of the word ``Int``, then ``foo``
.. will be an ``Int``. Otherwise, it will be a ``Nat``. When Idris
.. encounters the directive, it first checks that the provider expression
.. ``fromFile theType`` has type ``IO (Provider Type)``. Next, it executes
.. the provider. If the result is ``Provide t``, then ``T1`` is defined as
.. ``t``. Otherwise, the result is an error.

如果名为 ``theType`` 的文件由单词 ``Int`` 构成，那么 ``foo`` 的类型会是 ``Int``。
否则，它会是 ``Nat``。当 Idris 遇到该指令时，它首先会检查确认提供器表达式
``fromFile theType`` 的类型为 ``IO (Provider Type)``。接着它会执行该提供器。
如果其结果为 ``Provide t``，那么 ``T1`` 就会被定义为 ``t``。否则，就会产生一个错误。

.. Our datatype ``Provider t`` has the following definition:

我们的数据类型 ``Provider t`` 定义如下：

.. code-block:: idris

    data Provider a = Error String
                    | Provide a

.. We have already seen the ``Provide`` constructor. The ``Error``
.. constructor allows type providers to return useful error messages. The
.. example in this section was purposefully simple. More complex type
.. provider implementations, including a statically-checked SQLite binding,
.. are available in an external collection [1]_.

我们已经见过 ``Provide`` 构造器了。``Error`` 构造器允许类型提供器返回有用的错误信息，
本节中的示例为了简单并未提供。更加复杂的类型提供器实现，包括一个静态检查的 SQLite 绑定，
可从外部链接 [1]_ 获取。

以 C 为编译目标
===============

.. C Target
.. ========

.. The default target of Idris is C. Compiling via:

Idris 的默认编译目标为 C。它通过以下命令编译：

::

    $ idris hello.idr -o hello

.. is equivalent to:

此命令等价于：

::

    $ idris --codegen C hello.idr -o hello

.. When the command above is used, a temporary C source is generated, which
.. is then compiled into an executable named ``hello``.

当使用以上命令编译时，它会生成一个临时的 C 源码，接着再编译成名为 ``hello``
的可执行文件。

.. In order to view the generated C code, compile via:

要查看生成的 C 代码，请通过以下命令编译：

::

    $ idris hello.idr -S -o hello.c

.. To turn optimisations on, use the ``%flag C`` pragma within the code, as
.. is shown below:

要开启优化，请在代码中使用 ``%flag C`` 编译指令：

.. code-block:: idris

    module Main
    %flag C "-O3"

    factorial : Int -> Int
    factorial 0 = 1
    factorial n = n * (factorial (n-1))

    main : IO ()
    main = do
         putStrLn $ show $ factorial 3

.. To compile the generated C with debugging information e.g. to use
.. ``gdb`` to debug segmentation faults in Idris programs, use the
.. ``%flag C`` pragma to include debugging symbols, as is shown below:

要在编译生成的 C 代码时加上调试信息，例如使用 ``gdb`` 在 Idris 程序中调试段错误时，
请使用 ``%flag C`` 编译指令来包括调试符号，如下所示：

.. code-block:: idris

    %flag C "-g"

以 JavaScript 为编译目标
========================

.. JavaScript Target
.. =================

.. Idris is capable of producing *JavaScript* code that can be run in a
.. browser as well as in the *NodeJS* environment or alike. One can use the
.. FFI to communicate with the *JavaScript* ecosystem.

Idris 可生成能够运行在浏览器以及 *NodeJS* 等类似环境中的 *JavaScript* 代码。
它可以通过 FFI 与 *JavaScript* 生态进行交互。

代码生成
--------

.. Code Generation
.. ---------------

.. Code generation is split into two separate targets. To generate code
.. that is tailored for running in the browser issue the following command:

代码生成分为两种独立的目标。要生成适合在浏览器中运行的代码，请使用以下命令：

::

    $ idris --codegen javascript hello.idr -o hello.js

.. The resulting file can be embedded into your HTML just like any other
.. *JavaScript* code.

产生的文件可以像其它 *JavaScript* 代码那样嵌入到 HTML 中。

.. Generating code for *NodeJS* is slightly different. Idris outputs a
.. *JavaScript* file that can be directly executed via ``node``.

生成 *NodeJS* 代码的方式有点不同。Idris 会输出一个可直接调用 ``node`` 运行的
*JavaScript* 文件。

::

    $ idris --codegen node hello.idr -o hello
    $ ./hello
    Hello world

.. Take into consideration that the *JavaScript* code generator is using
.. ``console.log`` to write text to ``stdout``, this means that it will
.. automatically add a newline to the end of each string. This behaviour
.. does not show up in the *NodeJS* code generator.

考虑到 *JavaScript* 代码生成器使用 ``console.log`` 将文本写入到 ``stdout``，
因此它会自动在每个字符串之后加上换行。而此行为不会在 *NodeJS* 代码生成器中出现。

使用 FFI
--------

.. Using the FFI
.. -------------

.. To write a useful application we need to communicate with the outside
.. world. Maybe we want to manipulate the DOM or send an Ajax request. For
.. this task we can use the FFI. Since most *JavaScript* APIs demand
.. callbacks we need to extend the FFI so we can pass functions as
.. arguments.

要编写一个有用的应用，我们需要与外部世界进行交流。可能我们想要操作 DOM 或发送
Ajax 请求。为此可以使用 FFI。由于大部分 *JavaScript* API 需要回调，因此我们需要扩展
FFI 以将函数作为参数来传入。

.. The *JavaScript* FFI works a little bit differently than the regular
.. FFI. It uses positional arguments to directly insert our arguments into
.. a piece of *JavaScript* code.

*JavaScript* FFI 的工作方式与一般的 FFI 有点不同。它使用位置参数将我们的参数直接插入一段
*JavaScript* 代码中。

.. One could use the primitive addition of *JavaScript* like so:

我们可以使用 *JavaScript* 的原语加法：

.. code-block:: idris

    module Main

    primPlus : Int -> Int -> IO Int
    primPlus a b = mkForeign (FFun "%0 + %1" [FInt, FInt] FInt) a b

    main : IO ()
    main = do
      a <- primPlus 1 1
      b <- primPlus 1 2
      print (a, b)

.. Notice that the ``%n`` notation qualifies the position of the ``n``-th
.. argument given to our foreign function starting from 0. When you need a
.. percent sign rather than a position simply use ``%%`` instead.

注意 ``%n`` 记法确定了从 0 开始的第 ``n`` 个给定的外部函数的参数。
当你需要一个百分号而非位置时，请使用 ``%%`` 来代替。

.. Passing functions to a foreign function is very similar. Let’s assume
.. that we want to call the following function from the *JavaScript* world:

将函数传入外部函数中是非常相似的。假设我们想要从 *JavaScript* 世界中调用以下函数：

.. code-block:: idris

    function twice(f, x) {
      return f(f(x));
    }

.. We obviously need to pass a function ``f`` here (we can infer it from
.. the way we use ``f`` in ``twice``, it would be more obvious if
.. *JavaScript* had types).

显然我们需要在这里传入一个函数 ``f`` （我们可以从 ``twice`` 中使用 ``f``
的方式推断出来，如果 *JavaScript* 有类型的话会更加明显）。

.. The *JavaScript* FFI is able to understand functions as arguments when
.. you give it something of type ``FFunction``. The following example code
.. calls ``twice`` in *JavaScript* and returns the result to our Idris
.. program:

当你给 *JavaScript* FFI 一个类型为 ``FFunction`` 的东西时，它能够理解要将函数作为参数。
以下示例代码在 *JavaScript* 中调用了 ``twice`` 并将结果返回到了 Idris 程序中：

.. code-block:: idris

    module Main

    twice : (Int -> Int) -> Int -> IO Int
    twice f x = mkForeign (
      FFun "twice(%0,%1)" [FFunction FInt FInt, FInt] FInt
    ) f x

    main : IO ()
    main = do
      a <- twice (+1) 1
      print a

.. The program outputs ``3``, just like we expected.

该程序输出 ``3``，正如我们所料。

包含外部的 *JavaScript* 文件
----------------------------

.. Including external *JavaScript* files
.. -------------------------------------

.. Whenever one is working with *JavaScript* one might want to include
.. external libraries or just some functions that she or he wants to call
.. via FFI which are stored in external files. The *JavaScript* and
.. *NodeJS* code generators understand the ``%include`` directive. Keep in
.. mind that *JavaScript* and *NodeJS* are handled as different code
.. generators, therefore you will have to state which one you want to
.. target. This means that you can include different files for *JavaScript*
.. and *NodeJS* in the same Idris source file.

只要某人使用 *JavaScript*，他就有可能想要包含外部库，或者通过 FFI
调用存储在外部文件中的函数。*JavaScript* 和 *NodeJS* 代码生成器能够理解
``%include`` 指令。请注意 *JavaScript* 和 *NodeJS* 是由不同的代码生成器处理的，
因此你需要指明所需的目标。这也就表示你可以在同一个 Idris 源文件中分别为
*JavaScript* 和 *NodeJS* 包含不同的文件。

.. So whenever you want to add an external *JavaScript* file you can do
.. this like so:

因此如果你想要添加外部的 *JavaScript* 文件，可以这样做：

.. For *NodeJS*:

对于 *NodeJS*：

.. code-block:: idris

      %include Node "path/to/external.js"

.. And for use in the browser:

要在浏览器中使用：

.. code-block:: idris

      %include JavaScript "path/to/external.js"

.. The given files will be added to the top of the generated code.
.. For library packages you can also use the ipkg objs option to include the
.. js file in the installation, and use:

给定的文件会被添加的生成代码的顶部。对于库包，你也可以使用 ipkg 文件中的 objs
选项来将 js 文件包含在安装中，并使用：

.. code-block:: idris

      %include Node "package/external.js"

.. The *JavaScript* and *NodeJS* backends of Idris will also lookup for the file
.. on that location.

Idris 的 *JavaScript* 和 *NodeJS* 后端也会在此位置查找文件。

包含 *NodeJS* 模块
------------------

.. Including *NodeJS* modules
.. --------------------------

.. The *NodeJS* code generator can also include modules with the ``%lib``
.. directive.

*NodeJS* 代码生成器也可以通过 ``%lib`` 指令来包含模块。

.. code-block:: idris

      %lib Node "fs"

.. This directive compiles into the following *JavaScript*

该指令会编译成以下 *JavaScript*：

.. code-block:: javascript

      var fs = require("fs");

缩减生成的 *JavaScript*
-----------------------

.. Shrinking down generated *JavaScript*
.. -------------------------------------

.. Idris can produce very big chunks of *JavaScript* code. However, the
.. generated code can be minified using the ``closure-compiler`` from
.. Google. Any other minifier is also suitable but ``closure-compiler``
.. offers advanced compilation that does some aggressive inlining and code
.. elimination. Idris can take full advantage of this compilation mode
.. and it’s highly recommended to use it when shipping a *JavaScript*
.. application written in Idris.

Idris 会产生非常大的 *JavaScript* 代码块。然而，生成的代码可通过 Google 的
``closure-compiler`` 缩小。其它的缩减器也可用，不过 ``closure-compiler``
提供了更高级的编译，它会做一些侵入性的内敛和代码消除。Idris 可以充分利用这种编译模式，
强烈建议在传输用 Idris 编写的 *JavaScript* 应用时使用它。

累积性
======

.. Cumulativity
.. ============

.. Since values can appear in types and *vice versa*, it is natural that
.. types themselves have types. For example:

由于值可以出现在类型中， **反之亦然**，因此类型自然也有类型。例如：

::

    *universe> :t Nat
    Nat : Type
    *universe> :t Vect
    Vect : Nat -> Type -> Type

.. But what about the type of ``Type``? If we ask Idris it reports:

但是 ``Type`` 的类型呢？如果我们询问 Idris，它会报告：

::

    *universe> :t Type
    Type : Type 1

.. If ``Type`` were its own type, it would lead to an inconsistency due to
.. `Girard’s paradox <http://www.cs.cmu.edu/afs/cs.cmu.edu/user/kw/www/scans/girard72thesis.pdf>`_,
.. so internally there is a *hierarchy* of types (or *universes*):

如果 ``Type`` 是它自己的类型，那么它会因为
`Girard 悖论 <http://www.cs.cmu.edu/afs/cs.cmu.edu/user/kw/www/scans/girard72thesis.pdf>`_
而导致不一致性，因此在内部存在类型的 **层级（Hierarchy）** （或 **全域**，Universe）：

.. code-block:: idris

    Type : Type 1 : Type 2 : Type 3 : ...

.. Universes are *cumulative*, that is, if ``x : Type n`` we can also have
.. that ``x : Type m``, as long as ``n < m``. The typechecker generates
.. such universe constraints and reports an error if any inconsistencies
.. are found. Ordinarily, a programmer does not need to worry about this,
.. but it does prevent (contrived) programs such as the following:

全域类型是 **积累（Cumulative）** 的，也就是说，如果有 ``x : Type n``，
我们也可以有 ``x : Type m`` 使得 ``n < m``。如果类型检查器发现了任何不一致性，
它就会生成这种全域约束并报告一个错误。一般来说，程序员无须担心它，
但它确实可以防止（构造出）如下的程序：

.. code-block:: idris

    myid : (a : Type) -> a -> a
    myid _ x = x

    idid :  (a : Type) -> a -> a
    idid = myid _ myid

.. The application of ``myid`` to itself leads to a cycle in the universe
.. hierarchy — ``myid``\ ’s first argument is a ``Type``, which cannot be
.. at a lower level than required if it is applied to itself.

将 ``myid`` 应用到其自身会导致在全域层级中出现一个循环，即 ``myid`` 第一个参数的类型为
``Type``，如果要将其应用到自身，那么其级别不能低于所要求的级别。

.. [1]
   https://github.com/david-christiansen/idris-type-providers
