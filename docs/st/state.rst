.. _introst:

**********************************
ST 介绍：用状态来工作
**********************************

.. **********************************
.. Introducing ST: Working with State
.. **********************************

.. The ``Control.ST`` library provides facilities for creating, reading, writing
.. and destroying state in Idris functions, and tracking changes of state in
.. a function's type. It is based around the concept of *resources*, which are,
.. essentially, mutable variables, and a dependent type, ``STrans`` which tracks
.. how those resources change when a function runs:

``Control.ST`` 库提供了在函数中创建、读取、写入和销毁状态，以及在函数类型中跟踪\
状态变化的功能。它基于\ **资源（Resources）**\ 的概念，本质上也就是可变变量和依赖类型，
``STrans`` 会在函数运行时跟踪资源的变化：

.. code-block:: idris

    STrans : (m : Type -> Type) ->
             (resultType : Type) ->
             (in_res : Resources) ->
             (out_res : resultType -> Resources) ->
             Type

.. A value of type ``STrans m resultType in_res out_res_fn`` represents a sequence
.. of actions which can manipulate state. The arguments are:

.. * ``m``, which is an underlying *computation context* in which the actions will be executed.
..   Usually, this will be a generic type with a ``Monad`` implementation, but
..   it isn't necessarily so. In particular, there is no need to understand monads
..   to be able to use ``ST`` effectively!
.. * ``resultType``, which is the type of the value the sequence will produce
.. * ``in_res``, which is a list of *resources* available *before* executing the actions.
.. * ``out_res``, which is a list of resources available *after* executing the actions,
..   and may differ depending on the result of the actions.

类型为 ``STrans m resultType in_res out_res_fn`` 的值表示一系列可以操作状态的动作。
其参数分别为：

* ``m`` 表示一个底层的\ **计算上下文（Computation Context）**，各种动作会在其中执行。
  通常，它是一个实现了 ``Monad`` 的泛型，但并非必须如此。具体来说，我们无需理解单子
  （Monad）就能高效地使用 ``ST``！
* ``resultType`` 表示这一系列动作会产生的值的类型。
* ``in_res`` 表示一个在执行动作\ **之前**\ 可用的\ **资源**\ 列表。
* ``out_res`` 表示一个在执行动作\ **之后**\ 可用的\ **资源**\ 列表，它随动作的结果而不同。

.. We can use ``STrans`` to describe *state transition systems* in a function's
.. type. We'll come to the definition of ``Resources`` shortly, but for the moment
.. you can consider it an abstract representation of the "state of the world".
.. By giving the input resources (``in_res``) and the output resources
.. (``out_res``) we are describing the *preconditions* under which a function
.. is allowed to execute, and *postconditions* which describe how a function
.. affects the overall state of the world.

我们可以用 ``STrans`` 在函数的类型中描述\ **状态转移系统（state transition systems）**。\
我们会在之后定义 ``Resources`` （资源），现在你可以先把它看做“现实世界的状态”的抽象表示。\
通过给定输入资源（``in_res``）和输出资源（``out_res``），我们可以描述允许函数执行的\
**先决条件（preconditions）**\ 和描述了函数如何影响世界的所有状态的\
**后置条件（postconditions）**。

.. We'll begin in this section by looking at some small examples of ``STrans``
.. functions, and see how to execute them. We'll also introduce ``ST``,
.. a type-level function which allows us to describe the state transitions of
.. a stateful function concisely.

本节以一些 ``STrans`` 函数的小例子开始，看看如何执行它们。我们还介绍了
``ST``，一个类型级的函数，它能让我们简明地描述有状态函数的状态转移。

.. .. topic:: Type checking the examples

..     For the examples in this section, and throughout this tutorial,
..     you'll need to ``import Control.ST`` and add the ``contrib`` package by
..     passing the ``-p contrib`` flag to ``idris``.

.. topic:: 对示例进行类型检查

    对于本节和整个教程，你需要 ``import Control.ST`` 并通过向 ``idris`` 传递
    ``-p contrib`` 参数来添加 ``contrib`` 包。

初试：操作 ``State``
====================

.. Introductory examples: manipulating ``State``
.. =============================================

.. An ``STrans`` function explains, in its type, how it affects a collection of
.. ``Resources``. A resource has a *label* (of type ``Var``), which we use to
.. refer to the resource throughout the function, and we write the state of a
.. resource, in the ``Resources`` list, in the form ``label ::: type``.

``STrans`` 函数通过其类型解释了它如何影响一组 ``Resources``。资源拥有一个类型为
``Var`` 的\ **标签（label）**，我们会用它在函数中引用该资源，并在 ``Resources``
列表中以 ``label ::: type`` 的形式改写资源的状态。

.. For example, the following function
.. has a resource ``x`` available on input, of type ``State Integer``, and that
.. resource is still a ``State Integer`` on output:

例如，以下函数的输入中有一个资源 ``x``，其类型为 ``State Integer``，\
输出的资源类型仍为 ``State Integer`` :

.. code-block:: idris

  increment : (x : Var) -> STrans m () [x ::: State Integer]
                                       (const [x ::: State Integer])
  increment x = do num <- read x
                   write x (num + 1)

.. .. sidebar:: Verbosity of the type of ``increment``

..     The type of ``increment`` may seem somewhat verbose, in that the
..     *input* and *output* resources are repeated, even though they are the
..     same. We'll introduce a much more concise way of writing this type at the
..     end of this section (:ref:`sttype`), when we describe the ``ST`` type
..     itself.

.. sidebar:: ``increment`` 类型中的冗余

    ``increment`` 的类型看起来有点冗余，其中\ **输入**\ 和\ **输出**\
    资源的类型重复了一遍，即便它们是相同的。当我们在 :ref:`sttype`\
    一节的结尾描述 ``ST`` 类型本身时，我们会介绍一种更加简洁的类型写法。

.. This function reads the value stored at the resource ``x`` with ``read``,
.. increments it then writes the result back into the resource ``x`` with
.. ``write``. We'll see the types of ``read`` and ``write`` shortly
.. (see :ref:`stransprimops`). We can also create and delete resources:

本函数通过 ``read`` 读取资源 ``x`` 中存储的值，将其自增后的结果用 ``write``
写回资源 ``x`` 中。我们之后会看到 ``read`` 和 ``write`` 的类型（见 :ref:`stransprimops`）。\
我们还可以创建和删除资源：

.. code-block:: idris

  makeAndIncrement : Integer -> STrans m Integer [] (const [])
  makeAndIncrement init = do var <- new init
                             increment var
                             x <- read var
                             delete var
                             pure x

.. The type of ``makeAndIncrement`` states that it has *no* resources available on
.. entry (``[]``) or exit (``const []``). It creates a new ``State`` resource with
.. ``new`` (which takes an initial value for the resource), increments the value,
.. reads it back, then deletes it using ``delete``, returning the final value
.. of the resource. Again, we'll see the types of ``new`` and ``delete``
.. shortly.

``makeAndIncrement`` 的类型指明了它的入口（``[]``）和出口（``const []``）中\
**没有**\ 可用的资源。它会用 ``new`` 创建一个新的 ``State`` 资源（接受一个资源的初始值），\
对其值自增，将该值读出来，然后用 ``delete`` 删除它，返回该资源的最终值。我们后面同样会看到
``new`` 和 ``delete`` 的类型。

.. The ``m`` argument to ``STrans`` (of type ``Type -> Type``) is the *computation context* in
.. which the function can be run. Here, the type level variable indicates that we
.. can run it in *any* context. We can run it in the identity context with
.. ``runPure``. For example, try entering the above definitions in a file
.. ``Intro.idr`` then running the following at the REPL:

``STrans``（类型为 ``Type -> Type``）的参数 ``m`` 叫做\ **计算上下文（computation context）**，\
函数会在其中运行。在这里，该类型级变量指明了我们可以在\ **任何**\ 上下文中运行它。\
我们可以在相同的上下文中用 ``runPure`` 运行它。例如，将以上定义保存在 ``Intro.idr``
文件中，然后在 REPL 中运行以下语句：

.. code::

    *Intro> runPure (makeAndIncrement 93)
    94 : Integer

.. It's a good idea to take an interactive, type-driven approach to implementing
.. ``STrans`` programs. For example, after creating the resource with ``new init``,
.. you can leave a *hole* for the rest of the program to see how creating the
.. resource has affected the type:

通过交互式的，类型驱动的方式来实现 ``STrans`` 是个不错的主意。例如，在通过
``new init`` 创建了资源后，你可以为程序剩余的部分留一个\ **坑（hole）**\
来看到创建资源是如何影响类型的：

.. code-block:: idris

  makeAndIncrement : Integer -> STrans m Integer [] (const [])
  makeAndIncrement init = do var <- new init
                             ?whatNext

.. If you check the type of ``?whatNext``, you'll see that there is now
.. a resource available, ``var``, and that by the end of the function there
.. should be no resource available:

如果你检查一下 ``?whatNext`` 的类型，就会发现有一个可用的资源 ``var``，\
而在函数调用完成后应当会没有可用的资源：

.. code-block:: idris

      init : Integer
      m : Type -> Type
      var : Var
    --------------------------------------
    whatNext : STrans m Integer [var ::: State Integer] (\value => [])

.. These small examples work in any computation context ``m``. However, usually,
.. we are working in a more restricted context. For example, we might want to
.. write programs which only work in a context that supports interactive
.. programs. For this, we'll need to see how to *lift* operations from the
.. underlying context.

这个小例子可以在任何计算上下文 ``m`` 中工作。然而通常，我们会在一个更加严格的上下文中工作。\
例如，我们可能想要编写一个只能在支持交互式程序的上下文中工作的程序。为此，\
我们需要学习如何从底层上下文中\ **提升（lift）**\ 操作。

提升：使用计算上下文
====================

.. Lifting: Using the computation context
.. ======================================

.. Let's say that, instead of passing an initial integer to ``makeAndIncrement``,
.. we want to read it in from the console. Then, instead of working in a generic
.. context ``m``, we can work in the specific context ``IO``:

比如说，我们现在并不想直接把初始整数传入 ``makeAndIncrement``，而是想要把它从控制台读进来。
那么我们就要把一般的工作上下文 ``m`` 换成特定的上下文 ``IO``：

.. code-block:: idris

    ioMakeAndIncrement : STrans IO () [] (const [])

.. This gives us access to ``IO`` operations, via the ``lift`` function. We
.. can define ``ioMakeAndIncrement`` as follows:

``lift`` 函数给了我们访问 ``IO`` 操作的方式。我们可以将
``ioMakeAndIncrement`` 定义如下：

.. code-block:: idris

  ioMakeAndIncrement : STrans IO () [] (const [])
  ioMakeAndIncrement
     = do lift $ putStr "Enter a number: "
          init <- lift $ getLine
          var <- new (cast init)
          lift $ putStrLn ("var = " ++ show !(read var))
          increment var
          lift $ putStrLn ("var = " ++ show !(read var))
          delete var

.. The ``lift`` function allows us to use funtions from the underlying
.. computation context (``IO`` here) directly. Again, we'll see the exact type
.. of ``lift`` shortly.

``lift`` 函数能让我们直接使用底层计算上下文（此处为 ``IO``）中的函数。\
同样，我们很快就会看到 ``lift`` 具体的类型。

.. .. topic:: !-notation

..     In ``ioMakeAndIncrement`` we've used ``!(read var)`` to read from the
..     resource. You can read about this ``!``-notation in the main Idris tutorial
..     (see :ref:`monadsdo`). In short, it allows us to use an ``STrans``
..     function inline, rather than having to bind the result to a variable
..     first.

..     Conceptually, at least, you can think of it as having the following type:

..     .. code-block:: idris

..         (!) : STrans m a state_in state_out -> a

..     It is syntactic sugar for binding a variable immediately before the
..     current action in a ``do`` block, then using that variable in place of
..     the ``!``-expression.

.. topic:: !-记法

    在 ``ioMakeAndIncrement`` 中，我们使用了 ``!(read var)`` 从资源中读取信息。
    你可以在 Idris 教程（见 :ref:`monadsdo`）中找到关于 ``!``-记法的详情。
    简单来说，它允许我们直接就地使用 ``STrans`` 类型的函数，\
    而不必先将其结果绑定到一个变量。

    至少从概念上来说，你可以将它当做拥有以下类型的函数：

    .. code-block:: idris

        (!) : STrans m a state_in state_out -> a

    这个语法糖会在执行 ``do``-语句块中的当前动作之前立即绑定一个变量，
    然后在 ``!``-表达式的位置就地使用该变量。

.. In general, though, it's bad practice to use a *specific* context like
.. ``IO``. Firstly, it requires us to sprinkle ``lift`` liberally throughout
.. our code, which hinders readability. Secondly, and more importantly, it will
.. limit the safety of our functions, as we'll see in the next section
.. (:ref:`smstypes`).

然而，通常使用像 ``IO`` 这样\ **特定**\ 的上下文是种糟糕的实践。首先，\
它需要我们在代码中到处泼洒 ``lift`` ，这会影响可读性。再者，也是更重要的一点，\
它会限制我们函数的安全性，我们会在下一节（:ref:`smstypes`）中看到这一点。

.. So, instead, we define *interfaces* to restrict the computation context.
.. For example, ``Control.ST`` defines a ``ConsoleIO`` interface which
.. provides the necessary methods for performing basic console interaction:

所以我们改用定义\ **接口**\ 的方式来限制计算上下文。例如，``Control.ST`` 定义了
``ConsoleIO`` 接口，它为控制台的基本交互提供了必要的方法：

.. code-block:: idris

    interface ConsoleIO (m : Type -> Type) where
      putStr : String -> STrans m () res (const res)
      getStr : STrans m String res (const res)

.. That is, we can write to and read from the console with any available
.. resources ``res``, and neither will affect the available resources.
.. This has the following implementation for ``IO``:

也就是说，我们可以通过任何可用的资源 ``res`` 读写控制台，二者均不会影响可用的资源。
它有以下 ``IO`` 的实现：

.. code-block:: idris

    ConsoleIO IO where
      putStr str = lift (Interactive.putStr str)
      getStr = lift Interactive.getLine

.. Now, we can define ``ioMakeAndIncrement`` as follows:

现在，我们可以将 ``ioMakeAndIncrement`` 定义为：

.. code-block:: idris

  ioMakeAndIncrement : ConsoleIO io => STrans io () [] (const [])
  ioMakeAndIncrement
     = do putStr "Enter a number: "
          init <- getStr
          var <- new (cast init)
          putStrLn ("var = " ++ show !(read var))
          increment var
          putStrLn ("var = " ++ show !(read var))
          delete var

.. Instead of working in ``IO`` specifically, this works in a generic context
.. ``io``, provided that there is an implementation of ``ConsoleIO`` for that
.. context. This has several advantages over the first version:

.. * All of the calls to ``lift`` are in the implementation of the interface,
..   rather than ``ioMakeAndIncrement``
.. * We can provide alternative implementations of ``ConsoleIO``, perhaps
..   supporting exceptions or logging in addition to basic I/O.
.. * As we'll see in the next section (:ref:`smstypes`), it will allow us to
..   define safe APIs for manipulating specific resources more precisely.

它不仅可以在特定的 ``IO`` 中工作，还可以在一般的 ``io`` 上下文中工作，\
我们只需在该上下文中提供一个 ``ConsoleIO`` 的实现即可。相较于第一版而言，它有以下优点：

* 所有对 ``lift`` 的调用都在接口的实现中，而非在 ``ioMakeAndIncrement`` 中
* 我们可以提供另一种 ``ConsoleIO`` 的实现，比如在基本的 I/O 中支持异常或日志。
* 在下一节（:ref:`smstypes`）中我们将会看到，它可以让我们定义安全的 API，\
  以便更加精确地操作具体的资源。

.. Earlier, we used ``runPure`` to run ``makeAndIncrement`` in the identity
.. context. Here, we use ``run``, which allows us to execute an ``STrans`` program
.. in any context (as long as it has an implementation of ``Applicative``) and we
.. can execute ``ioMakeAndIncrement`` at the REPL as follows:

我们之前在同一个上下文中使用 ``runPure`` 来运行 ``makeAndIncrement``。而在这里，\
我们则使用 ``run``，它允许我们在任何上下文中执行 ``STrans`` 程序（只要它实现了
``Applicative`` 即可），我们可以像下面这样在 REPL 中执行 ``ioMakeAndIncrement``：

.. code::

    *Intro> :exec run ioMakeAndIncrement
    Enter a number: 93
    var = 93
    var = 94

.. _depstate:

用依赖类型操作 ``State``
========================

.. Manipulating ``State`` with dependent types
.. ===========================================

.. In our first example of ``State``, when we incremented the value its
.. *type* remained the same. However, when we're working with
.. *dependent* types, updating a state may also involve updating its type.
.. For example, if we're adding an element to a vector stored in a state,
.. its length will change:

在第一个 ``State`` 的例子中，当我们将该值自增后，其\ **类型**\ 并未改变。然而，\
当我们使用依赖类型时，状态的更新同样也会涉及到其类型的更新。例如，\
当我们向存储在状态中的向量添加一个元素时，其长度会改变：

.. code-block:: idris

  addElement : (vec : Var) -> (item : a) ->
               STrans m () [vec ::: State (Vect n a)]
                    (const [vec ::: State (Vect (S n) a)])
  addElement vec item = do xs <- read vec
                           write vec (item :: xs)

.. Note that you'll need to ``import Data.Vect`` to try this example.

注意你需要 ``import Data.Vect`` 来执行此示例。

.. .. topic:: Updating a state directly with ``update``

..     Rather than using ``read`` and ``write`` separately, you can also
..     use ``update`` which reads from a ``State``, applies a function to it,
..     then writes the result. Using ``update`` you could write ``addElement``
..     as follows:

.. topic:: 直接用 ``update`` 更新状态

    除了分别使用 ``read`` 和 ``write`` 以外，你还可以使用 ``update``，它从一个
    ``State`` 中读取内容，对它应用一个函数，然后写入其结果。通过 ``update``
    你可以将 ``addElement`` 写为如下形式：

    .. code-block:: idris

      addElement : (vec : Var) -> (item : a) ->
                   STrans m () [vec ::: State (Vect n a)]
                        (const [vec ::: State (Vect (S n) a)])
      addElement vec item = update vec (item ::)

.. We don't always know *how* exactly the type will change in the course of a
.. sequence actions, however. For example, if we have a state containing a
.. vector of integers, we might read an input from the console and only add it
.. to the vector if the input is a valid integer. Somehow, we need a different
.. type for the output state depending on whether reading the integer was
.. successful, so neither of the following types is quite right:

然而，我们并不总是可以知道在一系列动作中类型具体是\ **如何**\ 变化的。例如，\
如果我们有一个包含整数向量的状态，我们可以从控制台读取一个输入，\
只有当该输入为有效的整数时才将它添加到该向量中。根据该整数是否读取成功，\
我们的输出状态需要不同的类型，简直是莫名其妙。所以，下面两个类型都不太正确：

.. .. code-block:: idris

..   readAndAdd_OK : ConsoleIO io => (vec : Var) ->
..                   STrans m ()  -- Returns an empty tuple
..                               [vec ::: State (Vect n Integer)]
..                        (const [vec ::: State (Vect (S n) Integer)])
..   readAndAdd_Fail : ConsoleIO io => (vec : Var) ->
..                     STrans m ()  -- Returns an empty tuple
..                                 [vec ::: State (Vect n Integer)]
..                          (const [vec ::: State (Vect n Integer)])

.. code-block:: idris

  readAndAdd_OK : ConsoleIO io => (vec : Var) ->
                  STrans m ()  -- 返回空元组
                              [vec ::: State (Vect n Integer)]
                       (const [vec ::: State (Vect (S n) Integer)])
  readAndAdd_Fail : ConsoleIO io => (vec : Var) ->
                    STrans m ()  -- 返回空元组
                                [vec ::: State (Vect n Integer)]
                         (const [vec ::: State (Vect n Integer)])

.. Remember, though, that the *output* resource types can be *computed* from
.. the result of a function. So far, we've used ``const`` to note that the
.. output resources are always the same, but here, instead, we can use a type
.. level function to *calculate* the output resources. We start by returning
.. a ``Bool`` instead of an empty tuple, which is ``True`` if reading the input
.. was successful, and leave a *hole* for the output resources:

不过请记住，\ **输出**\ 资源的类型可以从函数的结果中\ **计算**\ 出来。
目前，我们使用 ``const`` 表示输出资源总是保持不变。不过在这里，\
我们可以使用一个类型级函数来\ **计算**\ 出输出资源。我们首先将返回空元组换成
``Bool``，当读取输入成功时它返回 ``True``；然后为输出资源挖一个\ **坑**\ ：

.. code-block:: idris

  readAndAdd : ConsoleIO io => (vec : Var) ->
               STrans m Bool [vec ::: State (Vect n Integer)]
                             ?output_res

.. If you check the type of ``?output_res``, you'll see that Idris expects
.. a function of type ``Bool -> Resources``, meaning that the output resource
.. type can be different depending on the result of ``readAndAdd``:

如果你检查 ``?output_res`` 的类型，就会看到 Idris 期望一个类型为
``Bool -> Resources`` 的函数，它表示输出资源的类型可以视 ``readAndAdd`` 的结果而不同：

.. code-block:: idris

      n : Nat
      m : Type -> Type
      io : Type -> Type
      constraint : ConsoleIO io
      vec : Var
    --------------------------------------
    output_res : Bool -> Resources

.. So, the output resource is either a ``Vect n Integer`` if the input is
.. invalid (i.e. ``readAndAdd`` returns ``False``) or a ``Vect (S n) Integer``
.. if the input is valid. We can express this in the type as follows:

所以，当输入无效时输出资源为 ``Vect n Integer``（例如 ``readAndAdd`` 返回 ``False``），\
当输入有效时输出资源为 ``Vect (S n) Integer``。我们可以用类型将它表示出来：

.. code-block:: idris

  readAndAdd : ConsoleIO io => (vec : Var) ->
               STrans io Bool [vec ::: State (Vect n Integer)]
                     (\res => [vec ::: State (if res then Vect (S n) Integer
                                                     else Vect n Integer)])

.. Then, when we implement ``readAndAdd`` we need to return the appropriate
.. value for the output state. If we've added an item to the vector, we need to
.. return ``True``, otherwise we need to return ``False``:

接着，我们在实现 ``readAndAdd`` 时需要为输出状态返回适当的值。如果为向量添加了一个元素，
就返回 ``True``，否则就要返回 ``False``：

.. .. code-block:: idris

..   readAndAdd : ConsoleIO io => (vec : Var) ->
..                STrans io Bool [vec ::: State (Vect n Integer)]
..                      (\res => [vec ::: State (if res then Vect (S n) Integer
..                                                      else Vect n Integer)])
..   readAndAdd vec = do putStr "Enter a number: "
..                       num <- getStr
..                       if all isDigit (unpack num)
..                          then do
..                            update vec ((cast num) ::)
..                            pure True     -- added an item, so return True
..                          else pure False -- didn't add, so return False

.. code-block:: idris

  readAndAdd : ConsoleIO io => (vec : Var) ->
               STrans io Bool [vec ::: State (Vect n Integer)]
                     (\res => [vec ::: State (if res then Vect (S n) Integer
                                                     else Vect n Integer)])
  readAndAdd vec = do putStr "Enter a number: "
                      num <- getStr
                      if all isDigit (unpack num)
                         then do
                           update vec ((cast num) ::)
                           pure True     -- 添加一个元素，因此返回 True
                         else pure False -- 没有添加，因此返回 False

.. There is a slight difficulty if we're developing interactively, which is
.. that if we leave a hole, the required output state isn't easily visible
.. until we know the value that's being returned. For example. in the following
.. incomplete definition of ``readAndAdd`` we've left a hole for the
.. successful case:

如果我们进行交互式开发的话会稍微有点不同，如果我们挖一个坑，那么在我们知道要返回的值以前，\
所需的输出状态并不显而易见。例如，在以下未完成的 ``readAndAdd`` 定义中，\
我们为成功的情况留了个坑：

.. code-block:: idris

  readAndAdd vec = do putStr "Enter a number: "
                      num <- getStr
                      if all isDigit (unpack num)
                         then ?whatNow
                         else pure False

.. We can look at the type of ``?whatNow``, but it is unfortunately rather less
.. than informative:

我们可以查看 ``?whatNow`` 的类型，很遗憾信息不足：

.. code-block:: idris

      vec : Var
      n : Nat
      io : Type -> Type
      constraint : ConsoleIO io
      num : String
    --------------------------------------
    whatNow : STrans io Bool [vec ::: State (Vect (S n) Integer)]
                     (\res =>
                        [vec :::
                         State (ifThenElse res
                                           (Delay (Vect (S n) Integer))
                                           (Delay (Vect n Integer)))])

.. The problem is that we'll only know the required output state when we know
.. the value we're returning. To help with interactive development, ``Control.ST``
.. provides a function ``returning`` which allows us to specify the return
.. value up front, and to update the state accordingly. For example, we can
.. write an incomplete ``readAndAdd`` as follows:

问题是我们只有在知道值会被返回时才能知道需要的输出状态。为了辅助交互式开发，
``Control.ST`` 提供了一个 ``returning`` 函数，我们可以用它来提前指定返回值，\
然后更新相应的状态。例如，我们可以将未完成的 ``readAndAdd`` 编写为：

.. code-block:: idris

  readAndAdd vec = do putStr "Enter a number: "
                      num <- getStr
                      if all isDigit (unpack num)
                         then returning True ?whatNow
                         else pure False

.. This states that, in the successful branch, we'll be returning ``True``, and
.. ``?whatNow`` should explain how to update the states appropriately so that
.. they are correct for a return value of ``True``. We can see this by checking
.. the type of ``?whatNow``, which is now a little more informative:

它说明了在成功的分支中，我们会返回 ``True``，\ ``?whatNow`` 应该解释如何相应地更新状态，\
使得它对于返回值 ``True`` 来说是正确的。我们只需检查 ``?whatNow`` 就会知道，\
现在的信息多了一点：

.. code-block:: idris

      vec : Var
      n : Nat
      io : Type -> Type
      constraint : ConsoleIO io
      num : String
    --------------------------------------
    whatnow : STrans io () [vec ::: State (Vect n Integer)]
                     (\value => [vec ::: State (Vect (S n) Integer)])

.. This type now shows, in the output resource list of ``STrans``,
.. that we can complete the definition by adding an item to ``vec``, which
.. we can do as follows:

现在这个类型表示，在 ``STrans`` 类型的输出资源列表中，我们可以通过向 ``vec``
添加一个元素来完成其定义：

.. .. code-block:: idris

..   readAndAdd vec = do putStr "Enter a number: "
..                       num <- getStr
..                       if all isDigit (unpack num)
..                          then returning True (update vec ((cast num) ::))
..                          else returning False (pure ()) -- returning False, so no state update required

.. code-block:: idris

  readAndAdd vec = do putStr "Enter a number: "
                      num <- getStr
                      if all isDigit (unpack num)
                         then returning True (update vec ((cast num) ::))
                         else returning False (pure ()) -- 返回 False，因此无需更新状态

.. _stransprimops:

``STrans`` 原语操作
===================

.. ``STrans`` Primitive operations
.. ===============================

.. Now that we've written a few small examples of ``STrans`` functions, it's
.. a good time to look more closely at the types of the state manipulation
.. functions we've used. First, to read and write states, we've used
.. ``read`` and ``write``:

我们已经写过几个关于 ``STrans`` 函数的小例子了，是时候更加细致地了解我们用过的状态操作函数了。
首先，为了读写状态，我们使用了 ``read`` 和 ``write``：

.. code-block:: idris

    read : (lbl : Var) -> {auto prf : InState lbl (State ty) res} ->
           STrans m ty res (const res)
    write : (lbl : Var) -> {auto prf : InState lbl ty res} ->
            (val : ty') ->
            STrans m () res (const (updateRes res prf (State ty')))

.. These types may look a little daunting at first, particularly due to the
.. implicit ``prf`` argument, which has the following type:

它们的类型看上去有点吓人，特别是隐式的 ``prf`` 参数，其类型为：

.. code-block:: idris

    prf : InState lbl (State ty) res

.. This relies on a predicate ``InState``. A value of type ``InState x ty res``
.. means that the reference ``x`` must have type ``ty`` in the list of
.. resources ``res``. So, in practice, all this type means is that we can
.. only read or write a resource if a reference to it exists in the list of
.. resources.

它依赖于一个断言 ``InState``。一个类型为 ``InState x ty res`` 的值意味着在资源列表
``res`` 中，引用 ``x`` 必须拥有类型 ``ty``。因此在实践中，所有这种类型都意味着，
如果一个对该资源的引用在资源列表中，那么我们我们只能读取或写入该资源。

.. Given a resource label ``res``, and a proof that ``res`` exists in a list
.. of resources, ``updateRes`` will update the type of that resource. So,
.. the type of ``write`` states that the type of the resource will be updated
.. to the type of the given value.

给定一个资源标签 ``res`` 和一个 ``res`` 存在于资源列表中的证明，那么 ``updateRes``
会更新该资源的类型。因此，``write`` 的类型说明了该资源的类型会被更新为给定值的类型。

.. The type of ``update`` is similar to that for ``read`` and ``write``, requiring
.. that the resource has the input type of the given function, and updating it to
.. have the output type of the function:

``update`` 的类型与 ``read`` 和 ``write`` 类型类似，它也需要资源拥有给定函数的输入类型，
并将它更新为该函数的输出类型：

.. code-block:: idris

    update : (lbl : Var) -> {auto prf : InState lbl (State ty) res} ->
             (ty -> ty') ->
             STrans m () res (const (updateRes res prf (State ty')))

.. The type of ``new`` states that it returns a ``Var``, and given an initial
.. value of type ``state``, the output resources contains a new resource
.. of type ``State state``:

``new`` 的类型说明了它返回一个 ``Var``，给定一个类型为 ``state`` 的初始值，
输出资源包含一个新的类型为 ``State state`` 的资源：

.. code-block:: idris

    new : (val : state) ->
          STrans m Var res (\lbl => (lbl ::: State state) :: res)

.. It's important that the new resource has type ``State state``, rather than
.. merely ``state``, because this will allow us to hide implementation details
.. of APIs. We'll see more about what this means in the next section,
.. :ref:`smstypes`.

新资源拥有类型 ``State state`` 而非只有 ``state`` 是是很重要的，因为这能让我们隐藏
API 的实现细节。在下一节 :ref:`smstypes` 中，我们会看到更多关于其意义的内容。

.. The type of ``delete`` states that the given label will be removed from
.. the list of resources, given an implicit proof that the label exists in
.. the input resources:

``delete`` 的类型说明了给定一个隐式的标签在输入资源内的证明，该标签会从资源列表中移除：

.. code-block:: idris

    delete : (lbl : Var) -> {auto prf : InState lbl (State st) res} ->
             STrans m () res (const (drop res prf))

.. Here, ``drop`` is a type level function which updates the resource list,
.. removing the given resource ``lbl`` from the list.

在这里，``drop`` 是一个类型级函数，它用于更新资源列表，从该列表中移除给定的资源
``lbl``。

.. We've used ``lift`` to run functions in the underlying context. It has the
.. following type:

我们之前已经用 ``lift`` 在底层上下文中运行过函数了。它的类型如下：

.. code-block:: idris

    lift : Monad m => m t -> STrans m t res (const res)

.. Given a ``result`` value, ``pure`` is an ``STrans`` program which produces
.. that value, provided that the current list of resources is correct when
.. producing that value:

给定一个 ``result`` 值，``pure`` 会返回产生该值的 ``STrans`` 程序，
当产生该值时，它会假设当前资源列表是正确：

.. code-block:: idris

    pure : (result : ty) -> STrans m ty (out_fn result) out_fn

.. We can use ``returning`` to break down returning a value from an
.. ``STrans`` functions into two parts: providing the value itself, and updating
.. the resource list so that it is appropriate for returning that value:

我们可以用 ``returning`` 将从 ``STrans`` 函数中返回值的过程分为两部分：提供值本身，
以及更新资源列表使其对应于该返回该值：

.. code-block:: idris

    returning : (result : ty) ->
                STrans m () res (const (out_fn result)) ->
                STrans m ty res out_fn

.. Finally, we've used ``run`` and ``runPure`` to execute ``STrans`` functions
.. in a specific context. ``run`` will execute a function in any context,
.. provided that there is an ``Applicative`` implementation for that context,
.. and ``runPure`` will execute a function in the identity context:

最后，我们已经用 ``run`` 和 ``runPure`` 在特定上下文中执行过 ``STrans``
函数了。``run`` 会在任何上下文中执行函数，若该上下文实现了 ``Applicative``，
那么 ``runPure`` 会在同一上下文中执行函数：

.. code-block:: idris

    run : Applicative m => STrans m a [] (const []) -> m a
    runPure : STrans Basics.id a [] (const []) -> a

.. Note that in each case, the input and output resource list must be empty.
.. There's no way to provide an initial resource list, or extract the final
.. resources. This is deliberate: it ensures that *all* resource management is
.. carried out in the controlled ``STrans`` environment and, as we'll see, this
.. allows us to implement safe APIs with precise types explaining exactly how
.. resources are tracked throughout a program.

注意在每一种情况下，输入和输出资源列表都必须为空。没有一种方法能提供初始资源列表，
或提取最终的资源。这是有意设计的：它确保了\ **所有的**\ 资源管理都在受控的
``STrans`` 环境下进行，并且我们将会看到，这让我们能够实现安全的 API，
以精确的类型来解释在程序的执行中资源是如何被跟踪的。

.. These functions provide the core of the ``ST`` library; there are some
.. others which we'll encounter later, for more advanced situations, but the
.. functions we have seen so far already allow quite sophisticated state-aware
.. programming and reasoning in Idris.

这些函数构成了 ``ST`` 库的核心。在遇到更加复杂的情况时，我们还会用到一些其它的函数，
不过目前见过的函数足以让我们用 Idris 进行细致的状态跟踪和推理了。

.. _sttype:

``ST``：直接表示状态转移
========================

.. `ST`: Representing state transitions directly
.. =============================================

.. We've seen a few examples of small ``STrans`` functions now, and
.. their types can become quite verbose given that we need to provide explicit
.. input and output resource lists. This is convenient for giving types for
.. the primitive operations, but for more general use it's much more convenient
.. to be able to express *transitions* on individual resources, rather than
.. giving input and output resource lists in full. We can do this with
.. ``ST``:

我们已经见过一些简单的 ``STrans`` 函数的例子了，由于需要提供显式的输入输出资源列表，
它们的类型会变得非常冗长。在需要为原语操作提供类型时这很方便，不过对于更一般的使用来说，
能为独立的资源表示\ **状态转移**\ ，而无需完整地给出输入和输出资源列表的话会更加方便。
我们可以用 ``ST`` 来做到这一点：

.. code-block:: idris

    ST : (m : Type -> Type) ->
         (resultType : Type) ->
         List (Action resultType) -> Type

.. ``ST`` is a type level function which computes an appropriate ``STrans``
.. type given a list of *actions*, which describe transitions on resources.
.. An ``Action`` in a function type can take one of the following forms (plus
.. some others which we'll see later in the tutorial):

.. * ``lbl ::: ty`` expresses that the resource ``lbl`` begins and ends in
..   the state ``ty``
.. * ``lbl ::: ty_in :-> ty_out`` expresses that the resource ``lbl`` begins
..   in state ``ty_in`` and ends in state ``ty_out``
.. * ``lbl ::: ty_in :-> (\res -> ty_out)`` expresses that the resource ``lbl``
..   begins in state ``ty_in`` and ends in a state ``ty_out``, where ``ty_out``
..   is computed from the result of the function ``res``.

``ST`` 是一个类型级函数，它会为给定的\ **活动（Action）**\ 列表计算出对应的
``STrans`` 类型，该类型描述了资源的状态转移。函数类型中的 ``Action``
可接受以下形式（我们之后还会看到其它的形式）：

* ``lbl ::: ty`` 表示资源 ``lbl`` 的开始和结束状态为 ``ty``
* ``lbl ::: ty_in :-> ty_out`` 表示资源 ``lbl`` 以状态 ``ty_in`` 开始，以状态
  ``ty_out`` 结束
* ``lbl ::: ty_in :-> (\res -> ty_out)`` 表示资源 ``lbl`` 以状态 ``ty_in``
  开始，以状态 ``ty_out`` 结束，其中 ``ty_out`` 计算自函数 ``res`` 的结果。

.. So, we can write some of the function types we've seen so far as follows:

因此我们可以将见过的一些函数的类型写成如下形式：

.. code-block:: idris

  increment : (x : Var) -> ST m () [x ::: State Integer]

.. That is, ``increment`` begins and ends with ``x`` in state ``State Integer``.

即，``increment`` 的开始和结束均为 ``State Integer`` 状态的 ``x``。

.. code-block:: idris

  makeAndIncrement : Integer -> ST m Integer []

.. That is, ``makeAndIncrement`` begins and ends with no resources.

即，``makeAndIncrement`` 的开始和结束均没有资源。

.. code-block:: idris

  addElement : (vec : Var) -> (item : a) ->
               ST m () [vec ::: State (Vect n a) :-> State (Vect (S n) a)]

.. That is, ``addElement`` changes ``vec`` from ``State (Vect n a)`` to
.. ``State (Vect (S n) a)``.

即，``addElement`` 将 ``vec`` 从 ``State (Vect n a)`` 改变为 ``State (Vect (S n) a)``。

.. code-block:: idris

  readAndAdd : ConsoleIO io => (vec : Var) ->
               ST io Bool
                     [vec ::: State (Vect n Integer) :->
                      \res => State (if res then Vect (S n) Integer
                                            else Vect n Integer)]

.. By writing the types in this way, we express the minimum necessary to explain
.. how each function affects the overall resource state. If there is a resource
.. update depending on a result, as with ``readAndAdd``, then we need to describe
.. it in full. Otherwise, as with ``increment`` and ``makeAndIncrement``, we can
.. write the input and output resource lists without repetition.

By writing the types in this way, we express the minimum necessary to explain
how each function affects the overall resource state. If there is a resource
update depending on a result, as with ``readAndAdd``, then we need to describe
it in full. Otherwise, as with ``increment`` and ``makeAndIncrement``, we can
write the input and output resource lists without repetition.
通过按这种方式写出类型，我们表达了解释每个函数如何影响整体资源状态的最小必要条件。
如果某个资源的更新依赖于某个结果（如 ``readAndAdd``），那么我们需要完整地描述它。
否则（如 ``increment`` 和 ``makeAndIncrement``），我们可以写出输入输出资源列表而避免重复。

.. An ``Action`` can also describe *adding* and *removing* states:

.. * ``add ty``, assuming the operation returns a ``Var``, adds a new resource
..   of type ``ty``.
.. * ``remove lbl ty`` expresses that the operation removes the resource named
..   ``lbl``, beginning in state ``ty`` from the resource list.

``Action`` 也可以描述\ **添加**\ 和 **移除**\ 这类状态：

* ``add ty``，如果该操作返回一个 ``Var``，那么它会添加一个 ``ty`` 类型的新资源。
* ``remove lbl ty`` 表示该操作会移除名为 ``lbl`` 的资源，以资源列表中的状态 ``ty``
  开始。

.. So, for example, we can write:

例如，我们可以写出

.. code-block:: idris

  newState : ST m Var [add (State Int)]
  removeState : (lbl : Var) -> ST m () [remove lbl (State Int)]

.. The first of these, ``newState``, returns a new resource label, and adds that
.. resource to the list with type ``State Int``. The second, ``removeState``,
.. given a label ``lbl``, removes the resource from the list. These types are
.. equivalent to the following:

第一个函数 ``newState`` 返回一个新的资源标签并将该资源添加到 ``State Int``
类型的资源列表中。第二个函数 ``removeState`` 根据给定的标签 ``lbl``
从列表中移除该资源。二者的类型与以下形式等价：

.. code-block:: idris

  newState : STrans m Var [] (\lbl => [lbl ::: State Int])
  removeState : (lbl : Var) -> STrans m () [lbl ::: State Int] (const [])

.. These are the primitive methods of constructing an ``Action``.  Later, we will
.. encounter some other ways using type level functions to help with readability.

它们是构造 ``Action`` 的原语方法。我们后面还会遇到一些用类型级函数来提高可读性的方法。

.. In the remainder of this tutorial, we will generally use ``ST`` except on
.. the rare occasions we need the full precision of ``STrans``. In the next
.. section, we'll see how to use the facilities provided by ``ST`` to write
.. a precise API for a system with security properties: a data store requiring
.. a login.

除了极少数需要准确完整的 ``STrans`` 的情况外，在本教程剩余的部分中，我们通常会使用
``ST``。在下一节中，我们会看到如何使用 ``ST`` 提供的设施来为需要安全性的系统编写准确的
API：一个需要登录的数据存储系统。
