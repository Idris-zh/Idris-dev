.. _composing:

**********
复合状态机
**********

.. ************************
.. Composing State Machines
.. ************************

.. In the previous section, we defined a ``DataStore`` interface and used it
.. to implement the following small program which allows a user to log in to
.. the store then display the store's contents;

在上一节中，我们定义了 ``DataStore`` 接口并用它实现了以下小程序，\
它能让用户登录该存储系统然后打印存储的内容：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) => ST m () []
  getData = do st <- connect
               OK <- login st
                  | BadPassword => do putStrLn "Failure"
                                      disconnect st
               secret <- readSecret st
               putStrLn ("Secret is: " ++ show secret)
               logout st
               disconnect st

.. This function only uses one state, the store itself. Usually, though,
.. larger programs have lots of states, and might add, delete and update
.. states over the course of its execution. Here, for example, a useful
.. extension might be to loop forever, keeping count of the number of times
.. there was a login failure in a state.

该函数只用了一个状态，即存储本身。然而，更大的程序通常会有更多的状态，\
在执行过程中可能还需要添加、删除或更新状态。就本例而言，无限循环，\
在状态中记录登录失败的次数都是有用的扩展。

.. Furthermore, we may have *hierarchies* of state machines, in that one
.. state machine could be implemented by composing several others. For
.. example, we can have a state machine representing the state of a
.. graphics system, and use this to implement a *higher level* graphics API
.. such as turtle graphics, which uses the graphics system plus some additional
.. state for the turtle.

此外，状态机还可以有\ **层级**\，即一个状态机可以通过复合其它状态机来实现。\
例如，我们可以有一个表示图形系统的状态机，并用它来实现「海龟绘图」这种\
**高级**\ 的图形 API，「海龟绘图」会使用图形系统加上一些额外的状态。

.. In this section, we'll see how to work with multiple states, and how to
.. compose state machines to make higher level state machines. We'll begin by
.. seeing how to add a login failure counter to ``getData``.

在本节中，我们会看到如何使用多个状态工作，以及如何将状态机复合成更高级别的状态机。\
我们先来看看如何为 ``getData`` 添加登录失败的计数器。


使用多个资源来工作
==================

.. Working with multiple resources
.. ===============================

.. To see how to work with multiple resources, we'll modify ``getData`` so
.. that it loops, and counts the total number of times the user fails to
.. log in. For example, if we write a ``main`` program which initialises the
.. count to zero, a session might run as follows:

为了了解如何使用多个资源来工作，我们需要修改 ``getData`` 使其循环，\
然后记录用户登录失败的总次数。例如，如果我们编写的 ``main`` 程序初始记录次数为 0，
那么会话过程看起来可能会是这样的：

.. code::

    *LoginCount> :exec main
    Enter password: Mornington Crescent
    Secret is: "Secret Data"
    Enter password: Dollis Hill
    Failure
    Number of failures: 1
    Enter password: Mornington Crescent
    Secret is: "Secret Data"
    Enter password: Codfanglers
    Failure
    Number of failures: 2
    ...

.. We'll start by adding a state resource to ``getData`` to keep track of the
.. number of failures:

我们首先为 ``getData`` 添加一个跟踪失败次数的状态资源：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) =>
            (failcount : Var) -> ST m () [failcount ::: State Integer]

.. topic:: ``getData`` 的类型检查

  .. If you're following along in the code, you'll find that ``getData``
  .. no longer compiles when you update this type. That is to be expected!
  .. For the moment, comment out the definition of ``getData``. We'll come back
  .. to it shortly.

  如果你跟着本教程写代码，就会发现更新了 ``getData`` 的类型后就无法通过编译了。\
  这是意料之中的！我们暂且先将 ``getData`` 的定义注释掉，回头再说。

.. Then, we can create a ``main`` program which initialises the state to ``0``
.. and invokes ``getData``, as follows:

接着，我们创建一个 ``main`` 程序，它将状态初始化为 0，然后调用 ``getData``：

.. code-block:: idris

  main : IO ()
  main = run (do fc <- new 0
                 getData fc
                 delete fc)

.. We'll start our implementation of ``getData`` just by adding the new
.. argument for the failure count:

现在着手实现 ``getData``。我们先来添加一个参数表示允许的失败次数：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) =>
            (failcount : Var) -> ST m () [failcount ::: State Integer]
  getData failcount
          = do st <- connect
               OK <- login st
                  | BadPassword => do putStrLn "Failure"
                                      disconnect st
               secret <- readSecret st
               putStrLn ("Secret is: " ++ show secret)
               logout st
               disconnect st

.. Unfortunately, this doesn't type check, because we have the wrong resources
.. for calling ``connect``. The error messages shows how the resources don't
.. match:

然而，它无法通过类型检查，因为我们用来调用 ``connect`` 的资源不正确。\
错误信息描述了资源为何不必配：

.. code-block:: idris

    When checking an application of function Control.ST.>>=:
        Error in state transition:
                Operation has preconditions: []
                States here are: [failcount ::: State Integer]
                Operation has postconditions: \result => [result ::: Store LoggedOut] ++ []
                Required result states here are: st2_fn

.. In other words, ``connect`` requires that there are *no* resources on
.. entry, but we have *one*, the failure count!
.. This shouldn't be a problem, though: the required resources are a *subset* of
.. the resources we have, after all, and the additional resources (here, the
.. failure count) are not relevant to ``connect``. What we need, therefore,
.. is a way to temporarily *hide* the additional resource.

换句话说，``connect`` 需要在进入时\ **没有**\ 资源，但我们却\ **有一个**\ 资源，\
即失败次数！这理应没什么问题：毕竟我们需要的资源是拥有的资源的\ **子集**\ ，\
而额外的资源（这里是失败次数）与 ``connect`` 无关。因此我们需要一种临时\ **隐藏**\
附加资源的方法。

.. We can achieve this with the ``call`` function:

我们可以用 ``call`` 函数来达此目的：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) =>
            (failcount : Var) -> ST m () [failcount ::: State Integer]
  getData failcount
     = do st <- call connect
          ?whatNow

.. Here we've left a hole for the rest of ``getData`` so that you can see the
.. effect of ``call``. It has removed the unnecessary parts of the resource
.. list for calling ``connect``, then reinstated them on return. The type of
.. ``whatNow`` therefore shows that we've added a new resource ``st``, and still
.. have ``failcount`` available:

我们在这里为 ``getData`` 剩下的部分挖了个坑，这样你可以看到 ``call`` 的作用。\
它移除了调用 ``connect`` 时资源列表中不必要的部分，然后在返回时恢复了它们。\
因此 ``whatNow`` 的类型表明我们添加了一个新的资源 ``st``，而 ``failcount``\
依然可用：

.. code-block:: idris

      failcount : Var
      m : Type -> Type
      constraint : ConsoleIO m
      constraint1 : DataStore m
      st : Var
    --------------------------------------
    whatNow : STrans m () [failcount ::: State Integer, st ::: Store LoggedOut]
                          (\result => [failcount ::: State Integer])

.. By the end of the function, ``whatNow`` says that we need to have finished with
.. ``st``, but still have ``failcount`` available. We can complete ``getData``
.. so that it works with an additional state resource by adding ``call`` whenever
.. we invoke one of the operations on the data store, to reduce the list of
.. resources:

在此函数函数的最后，``whatNow`` 表明我们需要以状态 ``st`` 结束，然而此时尚有
``failcount`` 可用。我们可以在调用数据存储系统中的操作时添加 ``call``
来消除资源列表，这样完成的 ``getData`` 就能带着附加的状态资源工作：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) =>
            (failcount : Var) -> ST m () [failcount ::: State Integer]
  getData failcount
          = do st <- call connect
               OK <- call $ login st
                  | BadPassword => do putStrLn "Failure"
                                      call $ disconnect st
               secret <- call $ readSecret st
               putStrLn ("Secret is: " ++ show secret)
               call $ logout st
               call $ disconnect st

.. This is a little noisy, and in fact we can remove the need for it by
.. making ``call`` implicit. By default, you need to add the ``call`` explicitly,
.. but if you import ``Control.ST.ImplicitCall``, Idris will insert ``call``
.. where it is necessary.

这样有点啰嗦，实际上我们可以将 ``call`` 变成隐式的从而去掉它。默认情况下，\
你需要显式地添加 ``call``，但如果你导入了 ``Control.ST.ImplicitCall``，那么
Idris 就会在需要的地方插入它。

.. code-block:: idris

  import Control.ST.ImplicitCall

.. It's now possible to write ``getData`` exactly as before:

现在的 ``getData`` 就和之前一样了：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) =>
            (failcount : Var) -> ST m () [failcount ::: State Integer]
  getData failcount
          = do st <- connect
               OK <- login st
                  | BadPassword => do putStrLn "Failure"
                                      disconnect st
               secret <- readSecret st
               putStrLn ("Secret is: " ++ show secret)
               logout st
               disconnect st

.. There is a trade off here: if you import ``Control.ST.ImplicitCall`` then
.. functions which use multiple resources are much easier to read, because the
.. noise of ``call`` has gone. On the other hand, Idris has to work a little
.. harder to type check your functions, and as a result it can take slightly
.. longer, and the error messages can be less helpful.

这里需要权衡：如果你导入了  ``Control.ST.ImplicitCall``，那么使用多个资源的函数\
会更加易读，因为没有啰嗦的 ``call`` 了。另一方面，Idris 对你函数的类型检查会变得\
有点困难，这会导致它花费更多时间，错误信息也会少一点帮助。

.. It is instructive to see the type of ``call``:

看一下 ``call`` 的类型，你会有所启发：

.. code-block:: idris

    call : STrans m t sub new_f -> {auto res_prf : SubRes sub old} ->
           STrans m t old (\res => updateWith (new_f res) old res_prf)

.. The function being called has a list of resources ``sub``, and
.. there is an implicit proof, ``SubRes sub old`` that the resource list in
.. the function being called is a subset of the overall resource list. The
.. ordering of resources is allowed to change, although resources which
.. appear in ``old`` can't appear in the ``sub`` list more than once (you will
.. get a type error if you try this).

被调用的函数有一个资源列表 ``sub``，还有一个隐式证明 ``SubRes sub old``，\
它证明了被调用函数的资源列表是整个资源列表的子集。尽管资源的顺序可以改变，然而在 ``old``
中出现的资源无法在 ``sub`` 列表中出现超过一次（如果你尝试它就会得到一个类型错误）。

.. The function ``updateWith`` takes the *output* resources of the
.. called function, and updates them in the current resource list. It makes
.. an effort to preserve ordering as far as possible, although this isn't
.. always possible if the called function does some complicated resource
.. manipulation.

函数 ``updateWith`` 接受被调用函数的\ **输出**\ 资源，然后在当前资源列表中更新它们。\
此函数会尽可能保持顺序不变，尽管被调用的函数在进行复杂的资源操作时并不总是可以保持顺序。

.. .. topic:: Newly created resources in called functions

..    If the called function creates any new resources, these will typically
..    appear at the *end* of the resource list, due to the way ``updateWith``
..    works. You can see this in the type of ``whatNow`` in our incomplete
..    definition of ``getData`` above.

.. topic:: 在被调用的函数中新创建的资源

   如果被调用的函数创建了新的资源，那么基于 ``updateWith`` 的工作方式，\
   它们通常会出现在资源列表的\ **末尾**\ 。你可以在前面未完成的 ``getData``
   的定义中看到这一点。

.. Finally, we can update ``getData`` so that it loops, and keeps
.. ``failCount`` updated as necessary:

最后我们可以更新 ``getData`` 使其可以循环，并在需要时保持更新 ``failCount``：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) =>
            (failcount : Var) -> ST m () [failcount ::: State Integer]
  getData failcount
     = do st <- call connect
          OK <- login st
             | BadPassword => do putStrLn "Failure"
                                 fc <- read failcount
                                 write failcount (fc + 1)
                                 putStrLn ("Number of failures: " ++ show (fc + 1))
                                 disconnect st
                                 getData failcount
          secret <- readSecret st
          putStrLn ("Secret is: " ++ show secret)
          logout st
          disconnect st
          getData failcount

.. Note that here, we're connecting and disconnecting on every iteration.
.. Another way to implement this would be to ``connect`` first, then call
.. ``getData``, and implement ``getData`` as follows:

注意在这里我们会在每次迭代中建立并断开连接。另一种实现方式是首先用 ``connect``
建立连接，然后调用 ``getData``，其实现如下：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) =>
            (st, failcount : Var) -> ST m () [st ::: Store {m} LoggedOut, failcount ::: State Integer]
  getData st failcount
     = do OK <- login st
             | BadPassword => do putStrLn "Failure"
                                 fc <- read failcount
                                 write failcount (fc + 1)
                                 putStrLn ("Number of failures: " ++ show (fc + 1))
                                 getData st failcount
          secret <- readSecret st
          putStrLn ("Secret is: " ++ show secret)
          logout st
          getData st failcount

.. It is important to add the explicit ``{m}`` in the type of ``Store {m}
.. LoggedOut`` for ``st``, because this gives Idris enough information to know
.. which implementation of ``DataStore`` to use to find the appropriate
.. implementation for ``Store``. Otherwise, if we only write ``Store LoggedOut``,
.. there's no way to know that the ``Store`` is linked with the computation
.. context ``m``.

在 ``st`` 的类型 ``Store {m} LoggedOut`` 中添加显式的 ``{m}`` 是十分重要的，因为这给了
Idris 足够的信息来判断哪一个 ``DataStore`` 的实现是用于查找其对应的 ``Store`` 的实现的。
否则，如果我们只写 ``Store LoggedOut``，那么将无法获知 ``Store`` 所关联到的计算上下文 ``m``。

.. We can then ``connect`` and ``disconnect`` only once, in ``main``:

接着我们可以在 ``main`` 中只 ``connect`` 并 ``disconnect`` 一次：

.. code-block:: idris

  main : IO ()
  main = run (do fc <- new 0
                 st <- connect
                 getData st fc
                 disconnect st
                 delete fc)

.. By using ``call``, and importing ``Control.ST.ImplicitCall``, we can
.. write programs which use multiple resources, and reduce the list of
.. resources as necessary when calling functions which only use a subset of
.. the overall resources.

通过使用 ``call`` 或导入 ``Control.ST.ImplicitCall``，我们可以编写使用多个资源的程序，
然后在调用一个只用到全部资源的子集的函数时，将资源列表按需删减。

复合资源：状态机的层级
======================

.. Composite resources: Hierarchies of state machines
.. ==================================================

.. We've now seen how to use multiple resources in one function, which is
.. necessary for any realistic program which manipulates state. We can think
.. of this as "horizontal" composition: using multiple resources at once.
.. We'll often also need "vertical" composition: implementing one resource
.. in terms of one or more other resources.

我们现在已经见过如何在一个函数中使用多个资源了，这对于任何能够操作状态的实际的程序\
来说都是必须的。我们可以把它看做是「横向的」复合：一次使用多个资源。我们通常还需要\
「纵向」的复合：基于一个或多个资源来实现单个资源。

.. We'll see an example of this in this section. First, we'll implement a
.. small API for graphics, in an interface ``Draw``, supporting:

.. * Opening a window, creating a double-buffered surface to draw on
.. * Drawing lines and rectangles onto a surface
.. * "Flipping" buffers, displaying the surface we've just drawn onto in
..   the window
.. * Closing a window

在本节中，我们会看到一个这种的例子。首先，我们在一个接口 ``Draw``
中实现一个小型的图形 API，它支持：

* 打开一个窗口，创建一个双缓冲（double-buffered）的平面（surface）来绘图
* 在平面上绘制线条和矩形
* 缓冲区「翻页（flipping）」，在窗口中显示我们刚绘制完毕的图像
* 关闭一个窗口

.. hint::
  绘图并不是即时呈现的，需要一个缓冲区（buffer）来暂存即将呈现的图像。
  缓冲区一般有三个基本操作：clear（清空），flip（翻页）和 rewind（重显）。
  clear 会将当前缓冲区清空以便重新绘图；flip 会将当前缓冲区中的图像显示在屏幕上，\
  然后清空缓冲区等待下一次绘图；而 rewind 则保持缓冲区内容不变，重新显示到屏幕上。

  术语 flip 和 rewind 来源于磁带的播放，rewind 即倒带重放，而 flip 则表示把磁带\
  翻过来播放另一面。flip 一词尚无标准译法，此处译作「翻页」表示本画面完成，\
  开始下一画面。

.. Then, we'll use this API to implement a higher level API for turtle graphics,
.. in an ``interface``.
.. This will require not only the ``Draw`` interface, but also a representation
.. of the turtle state (location, direction and pen colour).

接着，我们在一个 ``interface`` 中用此 API 来为「海龟绘图」实现一个更高级的 API。\
这不仅需要 ``Draw`` 接口，还需要表示海龟的状态（位置，方向和画笔颜色）。

.. .. topic:: SDL bindings

..     For the examples in this section, you'll need to install the
..     (very basic!) SDL bindings for Idris, available from
..     https://github.com/edwinb/SDL-idris. These bindings implement a small
..     subset of the SDL API, and are for illustrative purposes only.
..     Nevertheless, they are enough to implement small graphical programs
..     and demonstrate the concepts of this section.

..     Once you've installed this package, you can start Idris with the
..     ``-p sdl`` flag, for the SDL bindings, and the ``-p contrib`` flag,
..     for the ``Control.ST`` library.

.. topic:: SDL 绑定

    为了测试本节中的示例，你需要为 Idris 安装一个非常基本的 SDL 绑定，它可从
    https://github.com/edwinb/SDL-idris 获取。这些绑定实现了 SDL API
    的一个很小的子集，只用作演示的目的。尽管如此，它们已经足以实现一个\
    小型的绘图程序来展示本节的概念了。

    一旦你安装完这个包，就可以通过参数启动 Idris 了，``-p sdl`` 用于 SDL 绑定，\
    ``-p contrib`` 用于 ``Control.ST``。

``Draw`` 接口
-------------

.. The ``Draw`` interface
.. ----------------------

.. We're going to use the Idris SDL bindings for this API, so you'll need
.. to import ``Graphics.SDL`` once you've installed the bindings.
.. We'll start by defining the ``Draw`` interface, which includes a data type
.. representing a surface on which we'll draw lines and rectangles:

我们要在此 API 中使用 Idris 的 SDL 绑定，因此你需要在安装完该绑定库后导入
``Graphics.SDL``。我们先来定义 ``Draw`` 接口，它包含一个表示平面的数据类型，\
我们会在其之上绘制线条和矩形：

.. code-block:: idris

    interface Draw (m : Type -> Type) where
        Surface : Type

.. We'll need to be able to create a new ``Surface`` by opening a window:

我们需要能够通过打开新窗口来创建新的 ``Surface``：

.. code-block:: idris

    initWindow : Int -> Int -> ST m Var [add Surface]

.. However, this isn't quite right. It's possible that opening a window
.. will fail, for example if our program is running in a terminal without
.. a windowing system available. So, somehow, ``initWindow`` needs to cope
.. with the possibility of failure. We can do this by returning a
.. ``Maybe Var``, rather than a ``Var``, and only adding the ``Surface``
.. on success:

然而这样不太正确。如果我们的程序运行在没有可用的窗口系统的终端环境中，\
那么打开窗口可能会失败。因此， ``initWindow`` 需要以某种方式来应对可能的失败。\
我们可以通过返回 ``Maybe Var`` 而非 ``Var``，以及只在成功时添加 ``Surface``
来做到这一点：

.. code-block:: idris

    initWindow : Int -> Int -> ST m (Maybe Var) [addIfJust Surface]

.. This uses a type level function ``addIfJust``, defined in ``Control.ST``
.. which returns an ``Action`` that only adds a resource if the operation
.. succeeds (that is, returns a result of the form ``Just val``.

它使用了 ``Control.ST`` 中定义的类型级函数 ``addIfJust``，该函数返回一个
``Action``，仅在操作成功时添加资源（也就是说，它返回一个形如 ``Just val`` 的结果）。

.. .. topic:: ``addIfJust`` and ``addIfRight``

..   ``Control.ST`` defines functions for constructing new resources if an
..   operation succeeds. As well as ``addIfJust``, which adds a resource if
..   an operation returns ``Just ty``, there's also ``addIfRight``:

.. topic:: ``addIfJust`` 与 ``addIfRight``

  ``Control.ST`` 中定义了能够在操作成功时构造新资源的函数，其中的 ``addIfJust``
  会在操作返回 ``Just ty`` 时添加资源。此外还有 ``addIfRight``：

  .. code-block:: idris

     addIfJust : Type -> Action (Maybe Var)
     addIfRight : Type -> Action (Either a Var)

  .. Each of these is implemented in terms of the following primitive action
  .. ``Add``, which takes a function to construct a resource list from the result
  .. of an operation:

  二者均基于下面的原语动作 ``Add`` 开发。此动作接受一个函数，该函数从操作的结果中\
  构造出一个资源列表：

  .. code-block:: idris

     Add : (ty -> Resources) -> Action ty

  .. Using this, you can create your own actions to add resources
  .. based on the result of an operation, if required. For example,
  .. ``addIfJust`` is implemented as follows:

  如有需要，你可以用它来创建自己的动作，以此来基于某个操作的结果添加资源。\
  例如，``addIfJust`` 的实现如下：

  .. code-block:: idris

     addIfJust : Type -> Action (Maybe Var)
     addIfJust ty = Add (maybe [] (\var => [var ::: ty]))

.. If we create windows, we'll also need to be able to delete them:

如果我们能创建窗口，那么也需要能删除它：

.. code-block:: idris

    closeWindow : (win : Var) -> ST m () [remove win Surface]

.. We'll also need to respond to events such as keypresses and mouse clicks.
.. The ``Graphics.SDL`` library provides an ``Event`` type for this, and
.. we can ``poll`` for events which returns the last event which occurred,
.. if any:

我们还需要响应按下键盘或点击鼠标这类事件。``Graphics.SDL`` 库为此提供了
``Event`` 类型，而我们可以用 ``poll`` 轮询事件，如果存在的话，它会返回\
最后一个发生的事件：

.. code-block:: idris

    poll : ST m (Maybe Event) []

.. The remaining methods of ``Draw`` are ``flip``, which flips the buffers
.. displaying everything that we've drawn since the previous ``flip``, and
.. two methods for drawing: ``filledRectangle`` and ``drawLine``.

``Draw`` 中剩下的方法包括：``flip``，它会将从上次 ``flip`` 以来绘制的所有图像\
都显示出来；还有两个绘图的方法 ``filledRectangle`` 和 ``drawLine``。

.. code-block:: idris

    flip : (win : Var) -> ST m () [win ::: Surface]
    filledRectangle : (win : Var) -> (Int, Int) -> (Int, Int) -> Col -> ST m () [win ::: Surface]
    drawLine : (win : Var) -> (Int, Int) -> (Int, Int) -> Col -> ST m () [win ::: Surface]

.. We define colours as follows, as four components (red, green, blue, alpha):

.. .. code-block:: idris

..   data Col = MkCol Int Int Int Int

..   black : Col
..   black = MkCol 0 0 0 255

..   red : Col
..   red = MkCol 255 0 0 255

..   green : Col
..   green = MkCol 0 255 0 255

..   -- Also blue, yellow, magenta, cyan, white, similarly...

我们按如下方式定义色彩，四个整数表示色彩通道（红、绿、蓝、不透明度）：

.. code-block:: idris

  data Col = MkCol Int Int Int Int

  black : Col
  black = MkCol 0 0 0 255

  red : Col
  red = MkCol 255 0 0 255

  green : Col
  green = MkCol 0 255 0 255

  -- 蓝、黄、品红、青、白类似……

.. If you import ``Graphics.SDL``, you can implement the ``Draw`` interface
.. using the SDL bindings as follows:

在导入 ``Graphics.SDL`` 之后，你就可以像下面这样用 SDL 的绑定实现 ``Draw`` 接口了：

.. code-block:: idris

  implementation Draw IO where
    Surface = State SDLSurface

    initWindow x y = do Just srf <- lift (startSDL x y)
                             | pure Nothing
                        var <- new srf
                        pure (Just var)

    closeWindow win = do lift endSDL
                         delete win

    flip win = do srf <- read win
                  lift (flipBuffers srf)
    poll = lift pollEvent

    filledRectangle win (x, y) (ex, ey) (MkCol r g b a)
         = do srf <- read win
              lift $ filledRect srf x y ex ey r g b a
    drawLine win (x, y) (ex, ey) (MkCol r g b a)
         = do srf <- read win
              lift $ drawLine srf x y ex ey r g b a

.. In this implementation, we've used ``startSDL`` to initialise a window, which,
.. returns ``Nothing`` if it fails. Since the type of ``initWindow`` states that
.. it adds a resource when it returns a value of the form ``Just val``, we
.. add the surface returned by ``startSDL`` on success, and nothing on
.. failure.  We can only successfully initialise if ``startDSL`` succeeds.

在本实现中，我们使用 ``startSDL`` 来初始化窗口，它在失败时返回 ``Nothing``。\
由于 ``initWindow`` 的类型说明它会在返回形如 ``Just val`` 的值时添加一个资源，
因此我们在成功时添加 ``startSDL`` 返回的平面，在失败时什么也不做。我们只能在
``startDSL`` 成功时初始化成功。

.. Now that we have an implementation of ``Draw``, we can try writing some
.. functions for drawing into a window and execute them via the SDL bindings.
.. For example, assuming we have a surface ``win`` to draw onto, we can write a
.. ``render`` function as follows which draws a line onto a black background:

现在我们有了 ``Draw`` 的实现，可以试着写一些函数了，他们在窗口中绘图并通过
SDL 绑定执行它们。例如，假设我们有一个可以绘图的平面 ``win``，那么可以编写
``render`` 函数在黑色背景上绘图：

.. code-block:: idris

  render : Draw m => (win : Var) -> ST m () [win ::: Surface {m}]
  render win = do filledRectangle win (0,0) (640,480) black
                  drawLine win (100,100) (200,200) red
                  flip win

.. The ``flip win`` at the end is necessary because the drawing primitives
.. are double buffered, to prevent flicker. We draw onto one buffer, off-screen,
.. and display the other.  When we call ``flip``, it displays the off-screen
.. buffer, and creates a new off-screen buffer for drawing the next frame.

最后的 ``flip win`` 是必须的，因为绘图原语使用了双缓冲区来避免图形闪烁。\
我们在屏幕之外的缓冲区上绘图，同时显示另一个缓冲区。在调用 ``flip`` 时，\
它会将当前屏幕之外的缓冲区显示出来，并创建一个新的屏幕外缓冲区绘制下一帧。

.. To include this in a program, we'll write a main loop which renders our
.. image and waits for an event to indicate the user wants to close the
.. application:

要将它包含在程序里，我们需要编写一个主循环来渲染图像，同时等待用户关闭应用的事件：

.. code-block:: idris

  loop : Draw m => (win : Var) -> ST m () [win ::: Surface {m}]
  loop win = do render win
                Just AppQuit <- poll
                     | _ => loop win
                pure ()

.. Finally, we can create a main program which initialises a window, if
.. possible, then runs the main loop:

最后，我们创建一个主程序。如果可能，它会创建窗口，然后运行主循环：

.. code-block:: idris

  drawMain : (ConsoleIO m, Draw m) => ST m () []
  drawMain = do Just win <- initWindow 640 480
                   | Nothing => putStrLn "Can't open window"
                loop win
                closeWindow win

.. We can try this at the REPL using ``run``:

我们可以在 REPL 中用 ``run`` 运行它：

.. code::

  *Draw> :exec run drawMain

更高级的接口：``TurtleGraphics``
--------------------------------------------

.. A higher level interface: ``TurtleGraphics``
.. --------------------------------------------

.. Turtle graphics involves a "turtle" moving around the screen, drawing a line as
.. it moves with a "pen". A turtle has attributes describing its location, the
.. direction it's facing, and the current pen colour. There are commands for
.. moving the turtle forwards, turning through an angle, and changing the
.. pen colour, among other things. One possible interface would be the
.. following:

「海龟绘图」会操纵一只「海龟」在屏幕上移动，在它移动时用「画笔」来画线。\
一只海龟拥有描述它位置的属性，它面对的方向，以及当前画笔的颜色。此外，\
还有一些命令来让海龟向前移动，转一个角度，以及更改画笔的颜色。下面是一种可行的接口：

.. code-block:: idris

  interface TurtleGraphics (m : Type -> Type) where
    Turtle : Type

    start : Int -> Int -> ST m (Maybe Var) [addIfJust Turtle]
    end : (t : Var) -> ST m () [Remove t Turtle]

    fd : (t : Var) -> Int -> ST m () [t ::: Turtle]
    rt : (t : Var) -> Int -> ST m () [t ::: Turtle]

    penup : (t : Var) -> ST m () [t ::: Turtle]
    pendown : (t : Var) -> ST m () [t ::: Turtle]
    col : (t : Var) -> Col -> ST m () [t ::: Turtle]

    render : (t : Var) -> ST m () [t ::: Turtle]

.. Like ``Draw``, we have a command for initialising the turtle (here called
.. ``start``) which might fail if it can't create a surface for the turtle to
.. draw on. There is also a ``render`` method, which is intended to render the
.. picture drawn so far in a window.  One possible program with this interface
.. is the following, with draws a colourful square:

和 ``Draw`` 一样，我们也需要一个初始化海龟的命令（这里叫做 ``start``），\
如果它无法创建用来绘图的平面就会失败。此外还有一个 ``render`` 方法，\
它用来渲染窗口中目前已绘制的图像。使用此接口的一个可用的程序如下所示，\
它画了一个彩色的正方形：

.. code-block:: idris

  turtle : (ConsoleIO m, TurtleGraphics m) => ST m () []
  turtle = with ST do
              Just t <- start 640 480
                   | Nothing => putStr "Can't make turtle\n"
              col t yellow
              fd t 100; rt t 90
              col t green
              fd t 100; rt t 90
              col t red
              fd t 100; rt t 90
              col t blue
              fd t 100; rt t 90
              render t
              end t

.. .. topic:: ``with ST do``

..   The purpose of ``with ST do`` in ``turtle`` is to disambiguate ``(>>=)``,
..   which could be either the version from the ``Monad`` interface, or the
..   version from ``ST``. Idris can work this out itself, but it takes time to
..   try all of the possibilities, so the ``with`` clause can
..   speed up type checking.

.. topic:: ``with ST do``

  在 ``turtle`` 中使用 ``with ST do`` 是为了区分 ``(>>=)`` 的版本，它可能来自于
  ``Monad`` 或者 ``ST``。虽然 Idris 自己能够解决此问题，不过它会尝试所有可能性，\
  因此使用 ``with`` 从句可以减少耗时，加速类型检查。

.. To implement the interface, we could try using ``Surface`` to represent
.. the surface for the turtle to draw on:

要实现此接口，我们可以尝试用 ``Surface`` 来表示海龟用来作画的平面：

.. code-block:: idris

    implementation Draw m => TurtleGraphics m where
      Turtle = Surface {m}

.. Knowing that a ``Turtle`` is represented as a ``Surface``, we can use the
.. methods provided by ``Draw`` to implement the turtle.  Unfortunately, though,
.. this isn't quite enough. We need to store more information: in particular, the
.. turtle has several attributes which we need to store somewhere.
.. So, not only do we need to represent the turtle as a ``Surface``, we need
.. to store some additional state. We can achieve this using a *composite*
.. resource.

知道了 ``Turtle`` 被表示为 ``Surface`` 之后，我们就能使用 ``Draw`` 提供的方法\
来实现海龟了。然而这还不够，我们还需要存储更多信息：具体来说，海龟有一些属性\
需要存储在某处。因此我们不仅需要将海龟表示为一个 ``Surface``，还需要存储一些\
附加的状态。我们可以通过\ **复合**\ 资源来做到这一点。

复合资源简介
------------

.. Introducing composite resources
.. -------------------------------

.. A *composite* resource is built up from a list of other resources, and
.. is implemented using the following type, defined by ``Control.ST``:

**复合**\ 资源由一个资源列表构造而来，它使用以下 ``Control.ST`` 中定义的类型实现：

.. code-block:: idris

  data Composite : List Type -> Type

.. If we have a composite resource, we can split it into its constituent
.. resources, and create new variables for each of those resources, using
.. the *split* function. For example:

如果我们有一个复合资源，那么可以用 ``split`` 将其分解为构成它的资源，\
并为每个资源创建新的变量。例如：

.. code-block:: idris

  splitComp : (comp : Var) -> ST m () [comp ::: Composite [State Int, State String]]
  splitComp comp = do [int, str] <- split comp
                      ?whatNow

.. The call ``split comp`` extracts the ``State Int`` and ``State String`` from
.. the composite resource ``comp``, and stores them in the variables ``int``
.. and ``str`` respectively. If we check the type of ``whatNow``, we'll see
.. how this has affected the resource list:

调用 ``split comp`` 会从复合资源 ``comp`` 中提取出 ``State Int`` 和
``State String``，并分别将它们存储在变量 ``int`` 和 ``str`` 中。如果我们检查
``whatNow`` 的类型，就会看到它影响了资源列表：

.. code-block:: idris

      int : Var
      str : Var
      comp : Var
      m : Type -> Type
    --------------------------------------
    whatNow : STrans m () [int ::: State Int, str ::: State String, comp ::: State ()]
                          (\result => [comp ::: Composite [State Int, State String]])

.. So, we have two new resources ``int`` and ``str``, and the type of
.. ``comp`` has been updated to the unit type, so currently holds no data.
.. This is to be expected: we've just extracted the data into individual
.. resources after all.

这样，我们就有了两个新的资源 ``int`` 和 ``str``，而 ``comp`` 的类型则被更新为单元类型，
即当前没有保存数据。这点符合预期：我们只是要将数据提取为独立的资源而已。

.. Now that we've extracted the individual resources, we can manipulate them
.. directly (say, incrementing the ``Int`` and adding a newline to the
.. ``String``) then rebuild the composite resource using ``combine``:

现在我们提取出了独立的资源，可以直接操作它们（比如，增加 ``Int`` 或为 ``String``
添加换行）并使用 ``combine`` 重新构造复合资源：

.. code-block:: idris

  splitComp : (comp : Var) ->
              ST m () [comp ::: Composite [State Int, State String]]
  splitComp comp = do [int, str] <- split comp
                      update int (+ 1)
                      update str (++ "\n")
                      combine comp [int, str]
                      ?whatNow

.. As ever, we can check the type of ``whatNow`` to see the effect of
.. ``combine``:

同样，我们可以检查 ``whatNow`` 的类型来查看 ``combine`` 的作用：

.. code-block:: idris

      comp : Var
      int : Var
      str : Var
      m : Type -> Type
    --------------------------------------
    whatNow : STrans m () [comp ::: Composite [State Int, State String]]
                     (\result => [comp ::: Composite [State Int, State String]])

.. The effect of ``combine``, therefore, is to take existing
.. resources and merge them into one composite resource. Before we run
.. ``combine``, the target resource must exist (``comp`` here) and must be
.. of type ``State ()``.

所以 ``combine`` 的作用就是接受既有的资源并将它们合并成一个复合资源。\
在执行 ``combine`` 之前，目标资源必须存在（这里是 ``comp``）且类型为 ``State ()``。

.. It is instructive to look at the types of ``split`` and ``combine`` to see
.. the requirements on resource lists they work with. The type of ``split``
.. is the following:

查看 ``split`` 和 ``combine`` 的类型，了解它们使用的资源列表的要求是很有启发的。\
``split`` 的类型如下：

.. code-block:: idris

    split : (lbl : Var) -> {auto prf : InState lbl (Composite vars) res} ->
            STrans m (VarList vars) res (\vs => mkRes vs ++ updateRes res prf (State ()))

.. The implicit ``prf`` argument says that the ``lbl`` being split must be
.. a composite resource. It returns a variable list, built from the composite
.. resource, and the ``mkRes`` function makes a list of resources of the
.. appropriate types. Finally, ``updateRes`` updates the composite resource to
.. have the type ``State ()``.

隐式的 ``prf`` 参数说明要被分解的 ``lbl`` 必须为复合资源。它返回一个由复合资源分解\
而来的变量列表，而 ``mkRes`` 函数则创建一个对应类型的资源列表。最后，``updateRes``
会将复合资源的类型更新为 ``State ()``。

.. The ``combine`` function does the inverse:

而 ``combine`` 函数则反过来：

.. code-block:: idris

    combine : (comp : Var) -> (vs : List Var) ->
              {auto prf : InState comp (State ()) res} ->
              {auto var_prf : VarsIn (comp :: vs) res} ->
              STrans m () res (const (combineVarsIn res var_prf))

.. The implicit ``prf`` argument here ensures that the target resource ``comp``
.. has type ``State ()``. That is, we're not overwriting any other data.
.. The implicit ``var_prf`` argument is similar to ``SubRes`` in ``call``, and
.. ensures that every variable we're using to build the composite resource
.. really does exist in the current resource list.

隐式的 ``prf`` 参数确保了目标资源 ``comp`` 的类型为 ``State ()``。也就是说，\
我们不会覆盖任何其它的信息。隐式的 ``var_prf`` 参数类似于 ``call`` 中的 ``SubRes``，\
它确保了我们用来构造复合资源的每个变量都确实存在于当前的资源列表中。

.. We can use composite resources to implement our higher level ``TurtleGraphics``
.. API in terms of ``Draw``, and any additional resources we need.

我们可以基于 ``Draw`` 以及任何需要的附加资源，通过复合资源的方式来实现高级的
``TurtleGraphics`` API。

实现 ``Turtle``
---------------

.. Implementing ``Turtle``
.. -----------------------

.. Now that we've seen how to build a new resource from an existing collection,
.. we can implement ``Turtle`` using a composite resource, containing the
.. ``Surface`` to draw on, and individual states for the pen colour and the
.. pen location and direction. We also have a list of lines, which describes
.. what we'll draw onto the ``Surface`` when we call ``render``:

.. .. code-block:: idris

..   Turtle = Composite [Surface {m}, -- surface to draw on
..                       State Col,  -- pen colour
..                       State (Int, Int, Int, Bool), -- pen location/direction/d
..                       State (List Line)] -- lines to draw on render

现在我们已经知道如何用一组既有的资源来构造出新的资源了。我们可以用复合资源实现
``Turtle``，它包括一个用于绘图的 ``Surface``，以及一些表示画笔颜色、位置和方向的\
独立的状态。我们还有一个线条的列表，它描述了我们在调用 ``render`` 时要绘制到
``Surface`` 上的图像：

.. code-block:: idris

  Turtle = Composite [Surface {m}, -- 用来绘图的平面
                      State Col,  -- 画笔颜色
                      State (Int, Int, Int, Bool), -- 画笔的位置/方向/落笔标记
                      State (List Line)] -- 渲染时要画的线条

.. A ``Line`` is defined as a start location, and end location, and a colour:

一条 ``Line`` 由它的起始位置，终止位置和颜色定义：

.. code-block:: idris

  Line : Type
  Line = ((Int, Int), (Int, Int), Col)

.. To implement ``start``, which creates a new ``Turtle`` (or returns ``Nothing``
.. if this is impossible), we begin by initialising the drawing surface then
.. all of the components of the state. Finally, we combine all of these
.. into a composite resource for the turtle:

首先来实现 ``start``，它会创建一个新的 ``Turtle``（如果不可能则返回 ``Nothing``）。\
我们从初始化绘图平面开始，然后是所有状态构成的组件。最后，我们将所有的组件\
组合成一个复合资源来表示海龟：

.. code-block:: idris

    start x y = do Just srf <- initWindow x y
                        | Nothing => pure Nothing
                   col <- new white
                   pos <- new (320, 200, 0, True)
                   lines <- new []
                   turtle <- new ()
                   combine turtle [srf, col, pos, lines]
                   pure (Just turtle)

.. To implement ``end``, which needs to dispose of the turtle,
.. we deconstruct the composite resource, close the window,
.. then remove each individual resource. Remember that we can only ``delete``
.. a ``State``, so we need to ``split`` the composite resource, close the
.. drawing surface cleanly with ``closeWindow``, then ``delete`` the states:

然后来实现 ``end``，它会把海龟处理掉。我们先析构复合资源，然后关闭窗口，删除所有\
独立的资源。记住我们只能用 ``delete`` 删除一个 ``State``，因此需要用 ``split``
将复合资源分解掉，用 ``closeWindow`` 干净地关闭绘图平面，然后用 ``delete`` 删除状态：

.. code-block:: idris

    end t = do [srf, col, pos, lines] <- split t
               closeWindow srf; delete col; delete pos; delete lines; delete t

.. For the other methods, we need to ``split`` the resource to get each
.. component, and ``combine`` into a composite resource when we're done.
.. As an example, here's ``penup``:

.. .. code-block:: idris

..     penup t = do [srf, col, pos, lines] <- split t -- Split the composite resource
..                  (x, y, d, _) <- read pos          -- Deconstruct the pen position
..                  write pos (x, y, d, False)        -- Set the pen down flag to False
..                  combine t [srf, col, pos, lines]  -- Recombine the components

对于其它的方法，我们需要用 ``split`` 分解资源以获取每一个组件，然后在结束时用
``combine`` 将它们组合成一个复合资源。例如，以下为 ``penup`` 的实现：

.. code-block:: idris

    penup t = do [srf, col, pos, lines] <- split t -- 分解复合资源
                 (x, y, d, _) <- read pos          -- 析构画笔位置
                 write pos (x, y, d, False)        -- 将落笔标记置为 False
                 combine t [srf, col, pos, lines]  -- 重新组合组件

.. The remaining operations on the turtle follow a similar pattern. See
.. ``samples/ST/Graphics/Turtle.idr`` in the Idris distribution for the full
.. details. It remains to render the image created by the turtle:

.. .. code-block:: idris

..     render t = do [srf, col, pos, lines] <- split t -- Split the composite resource
..                   filledRectangle srf (0, 0) (640, 480) black -- Draw a background
..                   drawAll srf !(read lines)         -- Render the lines drawn by the turtle
..                   flip srf                          -- Flip the buffers to display the image
..                   combine t [srf, col, pos, lines]
..                   Just ev <- poll
..                     | Nothing => render t           -- Keep going until a key is pressed
..                   case ev of
..                        KeyUp _ => pure ()           -- Key pressed, so quit
..                        _ => render t
..      where drawAll : (srf : Var) -> List Line -> ST m () [srf ::: Surface {m}]
..            drawAll srf [] = pure ()
..            drawAll srf ((start, end, col) :: xs)
..               = do drawLine srf start end col       -- Draw a line in the appropriate colour
..                    drawAll srf xs

海龟剩下的操作遵循相同的模式。完整的细节见 Idris 发行版中的
``samples/ST/Graphics/Turtle.idr`` 文件。之后就是是渲染海龟创建的图像：

.. code-block:: idris

    render t = do [srf, col, pos, lines] <- split t -- 分解复合资源
                  filledRectangle srf (0, 0) (640, 480) black -- 绘制背景
                  drawAll srf !(read lines)         -- 渲染海龟绘制的线条
                  flip srf                          -- 缓冲区翻页以显示图像
                  combine t [srf, col, pos, lines]
                  Just ev <- poll
                    | Nothing => render t           -- 继续直到按下按键
                  case ev of
                       KeyUp _ => pure ()           -- 按键按下，于是退出
                       _ => render t
     where drawAll : (srf : Var) -> List Line -> ST m () [srf ::: Surface {m}]
           drawAll srf [] = pure ()
           drawAll srf ((start, end, col) :: xs)
              = do drawLine srf start end col       -- 按相应的颜色绘制线条
                   drawAll srf xs
