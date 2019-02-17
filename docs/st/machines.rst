.. _smstypes:

****************
用类型表示状态机
****************

.. ***********************
.. State Machines in Types
.. ***********************

.. In the introduction, we saw the following state transition diagram representing
.. the (abstract) states of a data store, and the actions we can perform on the
.. store:

在\ :ref:`stoverview`\ 一节中，我们用状态转移图展示了数据存储系统的抽象状态，
以及可以对它执行的动作：

|login|

.. We say that these are the *abstract* states of the store, because the concrete
.. state will contain a lot more information: for example, it might contain
.. user names, hashed passwords, the store contents, and so on. However, as far
.. as we are concerned for the actions ``login``, ``logout`` and ``readSecret``,
.. it's whether we are logged in or not which affects which are valid.

我们之所以把它称作该存储系统的\ **抽象**\ 状态，是因为具体的状态会包含更多信息。\
例如，它可能包含用户名、散列化的密码和存储内容等等。然而，就目前我们我们所关心的动作
``login``、``logout`` 和 ``readSecret`` 而言，登录状态决定了哪些动作是有效的。

.. We've seen how to manipulate states using ``ST``, and some small examples
.. of dependent types in states. In this section, we'll see how to use
.. ``ST`` to provide a safe API for the data store. In the API, we'll encode
.. the above diagram in the types, in such a way that we can only execute the
.. operations ``login``, ``logout`` and ``readSecret`` when the state is
.. valid.

我们已经见过如何用 ``ST`` 操作状态，用依赖类型表示状态了。在本节中，我们会看到如何用
``ST`` 为数据存储系统提供安全的 API。在 API 中，我们会用类型编码上面的状态转移图。
通过这种方式，我们可以只在状态有效时才能执行 ``login``、``logout`` 和 ``readSecret``
操作。

.. So far, we've used ``State`` and the primitive operations, ``new``, ``read``,
.. ``write`` and ``delete`` to manipulate states. For the data store API,
.. however, we'll begin by defining an *interface* (see :ref:`sect-interfaces` in
.. the Idris tutorial) which describes the operations on the store, and explains
.. in their types exactly when each operation is valid, and how it affects
.. the store's state. By using an interface, we can be sure that
.. this is the *only* way to access the store.

我们已经用过 ``State`` 及其原语操作 ``new``、``read``、``write`` 和 ``delete``
来操作状态了。而对于数据存储的 API，我们则以定义\ **接口**\ 开始（见 Idris 教程中的\
:ref:`sect-interfaces`\ 一节）。接口描述了对存储系统的操作，其类型则准确解释了每种操作\
何时才有效，以及它是如何影响存储系统的状态的。通过接口，我们可以确保这是访问存储系统的\
\ **唯一的**\ 方式。

为数据存储系统定义接口
======================

.. Defining an interface for the data store
.. ========================================

.. We'll begin by defining a data type, in a file ``Login.idr``, which represents
.. the two abstract states of the store, either ``LoggedOut`` or ``LoggedIn``:

我们首先在 ``Login.idr`` 文件中定义数据类型，它表示存储系统的两种抽象状态，即
``LoggedOut`` 和 ``LoggedIn``：

.. code-block:: idris

    data Access = LoggedOut | LoggedIn

.. We can define a data type for representing the current state of a store,
.. holding all of the necessary information (this might be user names, hashed
.. passwords, store contents and so on) and parameterise it by the logged in
.. status of the store:

我们可以定义一个数据类型来表示存储系统的当前状态，保存所有必要的信息\
（如用户名、散列化的密码、存储内容等等），并根据登录状态来参数化该数据类型：

.. code-block:: idris

  Store : Access -> Type

.. Rather than defining a concrete type now, however, we'll include this in
.. a data store *interface* and define a concrete type later:

不过我们现在先不定义具体的类型，而是将以下代码包含在数据存储的\ **接口**\ 中，\
之后再定义具体的类型：

.. code-block:: idris

  interface DataStore (m : Type -> Type) where
    Store : Access -> Type

.. We can continue to populate this interface with operations on the store.  Among
.. other advantages, by separating the *interface* from its *implementation* we
.. can provide different concrete implementations for different contexts.
.. Furthermore, we can write programs which work with a store without needing
.. to know any details of how the store is implemented.

我们可以继续为此接口补充其它存储系统的操作。这样做优点众多。通过将\ **接口**\
与其\ **实现**\ 相分离，我们可以为不同的上下文提供相应的具体实现。此外，\
我们还可以编写与存储系统协作的程序而无需知道任何实现的细节。

.. We'll need to be able to ``connect`` to a store, and ``disconnect`` when
.. we're done. Add the following methods to the ``DataStore`` interface:

我们需要用 ``connect`` 连接到该存储系统，在结束后用 ``disconnect`` 断开连接。
我们为 ``DataStore`` 接口添加以下方法：

.. code-block:: idris

    connect : ST m Var [add (Store LoggedOut)]
    disconnect : (store : Var) -> ST m () [remove store (Store LoggedOut)]

.. The type of ``connect`` says that it returns a new resource which has the
.. initial type ``Store LoggedOut``. Conversely, ``disconnect``, given a
.. resource in the state ``Store LoggedOut``, removes that resource.
.. We can see more clearly what ``connect`` does by trying the following
.. (incomplete) definition:

``connect`` 的类型表明它会返回一个初始类型为 ``Store LoggedOut`` 的新资源。
反之，``disconnect`` 则会给出一个状态为 ``Store LoggedOut`` 的资源并移除该资源。
我们可以通过以下（未完成的）定义更加清楚地看到 ``connect`` 做了什么：

.. code-block:: idris

  doConnect : DataStore m => ST m () []
  doConnect = do st <- connect
                 ?whatNow

.. Note that we're working in a *generic* context ``m``, constrained so that
.. there must be an implementation of ``DataStore`` for ``m`` to be able to
.. execute ``doConnect``.
.. If we check the type of ``?whatNow``, we'll see that the remaining
.. operations begin with a resource ``st`` in the state ``Store LoggedOut``,
.. and we need to finish with no resources.

注意我们正在一个 \ **一般**\ 的上下文 ``m`` 中工作，为了能够执行 ``doConnect``，\
我们必须为 ``m`` 实现 ``DataStore`` 接口来限制它。如果我们检查 ``?whatNow``
的类型，就会看到剩下的操作以一个状态为 ``Store LoggedOut`` 的资源 ``st`` 开始，
以没有可用的资源结束：

.. code-block:: idris

      m : Type -> Type
      constraint : DataStore m
      st : Var
    --------------------------------------
    whatNow : STrans m () [st ::: Store LoggedOut] (\result => [])

.. Then, we can remove the resource using ``disconnect``:

接着，我们可以用 ``disconnect`` 来移除该资源：

.. code-block:: idris

  doConnect : DataStore m => ST m () []
  doConnect = do st <- connect
                 disconnect st
                 ?whatNow

.. Now checking the type of ``?whatNow`` shows that we have no resources
.. available:

现在检查 ``?whatNow`` 的类型会显示我们没有可用的资源：

.. code-block:: idris

      m : Type -> Type
      constraint : DataStore m
      st : Var
    --------------------------------------
    whatNow : STrans m () [] (\result => [])

.. To continue our implementation of the ``DataStore`` interface, next we'll add a
.. method for reading the secret data. This requires that the ``store`` is in the
.. state ``Store LoggedIn``:

为了继续完善 ``DataStore`` 接口的实现，我们接下来添加一个读取机密数据的方法。
这需要 ``store`` 的状态为 ``Store LoggedIn``：

.. code-block:: idris

    readSecret : (store : Var) -> ST m String [store ::: Store LoggedIn]

.. At this point we can try writing a function which connects to a store,
.. reads the secret, then disconnects. However, it will be unsuccessful, because
.. ``readSecret`` requires us to be logged in:

此时我们可以试着编写一个函数，它先连接到存储系统，然后读取机密数据，之后断开连接。
然而它并不会成功，因为执行 ``readSecret`` 需要我们处于已登录状态。

.. code-block:: idris

  badGet : DataStore m => ST m () []
  badGet = do st <- connect
              secret <- readSecret st
              disconnect st

.. This results in the following error, because ``connect`` creates a new
.. store in the ``LoggedOut`` state, and ``readSecret`` requires the store
.. to be in the ``LoggedIn`` state:

它会产生以下错误，因为 ``connect`` 创建了状态为 ``LoggedOut`` 的新存储，而
``readSecret`` 需要该存储的状态为 ``LoggedIn``：

.. code-block:: idris

    When checking an application of function Control.ST.>>=:
        Error in state transition:
                Operation has preconditions: [st ::: Store LoggedOut]
                States here are: [st ::: Store LoggedIn]
                Operation has postconditions: \result => []
                Required result states here are: \result => []

.. The error message explains how the required input states (the preconditions)
.. and the required output states (the postconditions) differ from the states
.. in the operation. In order to use ``readSecret``, we'll need a way to get
.. from a ``Store LoggedOut`` to a ``Store LoggedIn``. As a first attempt,
.. we can try the following type for ``login``:

该错误信息解释了所需的输入状态（前提条件）和输出状态（后置条件）与该操作中的状态\
有何不同。为了使用 ``readSecret``，我们需要一种方式将 ``Store LoggedOut`` 转换为
``Store LoggedIn`` 状态。我们可以先尝试将 ``login``\ （登录）指定为以下类型：

.. .. code-block:: idris

..     login : (store : Var) -> ST m () [store ::: Store LoggedOut :-> Store LoggedIn] -- Incorrect type!

.. code-block:: idris

    login : (store : Var) -> ST m () [store ::: Store LoggedOut :-> Store LoggedIn] -- 类型不正确！

.. Note that in the *interface* we say nothing about *how* ``login`` works;
.. merely how it affects the overall state. Even so, there is a problem with
.. the type of ``login``, because it makes the assumption that it will always
.. succeed. If it fails - for example because the implementation prompts for
.. a password and the user enters the password incorrectly - then it must not
.. result in a ``LoggedIn`` store.

注意，\ **接口**\ 中并没有说明 ``login`` 是\ **如何**\ 工作的，只是表达了它如何\
影响所有的状态。即便如此，\ ``login`` 的类型还是有点问题，因为它假设了登录\
总会成功。如果登录失败（比如在该实现提示输入密码时用户输入了错误的密码），\
那么它一定不会产生 ``LoggedIn`` 状态的存储。

.. Instead, therefore, ``login`` will return whether logging in was successful,
.. via the following type;

因此，``login`` 需要通过以下类型返回登录是否成功：

.. code-block:: idris

    data LoginResult = OK | BadPassword

.. Then, we can *calculate* the result state (see :ref:`depstate`) from the
.. result. Add the following method to the ``DataStore`` interface:

接着，我们可以从结果中\ **计算**\ 出结果状态（见\ :ref:`depstate`）。我们为
``DataStore`` 接口添加以下方法：

.. code-block:: idris

    login : (store : Var) ->
            ST m LoginResult [store ::: Store LoggedOut :->
                               (\res => Store (case res of
                                                    OK => LoggedIn
                                                    BadPassword => LoggedOut))]

.. If ``login`` was successful, then the state after ``login`` is
.. ``Store LoggedIn``. Otherwise, the state is ``Store LoggedOut``.

如果 ``login`` 成功，那么 ``login`` 之后的状态会变成 ``Store LoggedIn``。\
否则，状态仍然为 ``Store LoggedOut``。

.. To complete the interface, we'll add a method for logging out of the store.
.. We'll assume that logging out is always successful, and moves the store
.. from the ``Store LoggedIn`` state to the ``Store LoggedOut`` state.

为完成此接口，我们还需要添加一个退出该存储系统的方法。我们假设退出总是成功，并将存储系统的状态从 ``Store LoggedIn`` 转换为 ``Store LoggedOut``。

.. code-block:: idris

    logout : (store : Var) -> ST m () [store ::: Store LoggedIn :-> Store LoggedOut]

.. This completes the interface, repeated in full for reference below:

这样就完成了此接口。完整代码如下：

.. code-block:: idris

  interface DataStore (m : Type -> Type) where
    Store : Access -> Type

    connect : ST m Var [add (Store LoggedOut)]
    disconnect : (store : Var) -> ST m () [remove store (Store LoggedOut)]

    readSecret : (store : Var) -> ST m String [store ::: Store LoggedIn]
    login : (store : Var) ->
            ST m LoginResult [store ::: Store LoggedOut :->
                               (\res => Store (case res of
                                                    OK => LoggedIn
                                                    BadPassword => LoggedOut))]
    logout : (store : Var) -> ST m () [store ::: Store LoggedIn :-> Store LoggedOut]

.. Before we try creating any implementations of this interface, let's see how
.. we can write a function with it, to log into a data store, read the secret
.. if login is successful, then log out again.

在尝试创建此接口的实现之前，我们来看看如何用它来编写函数，以此来登录数据存储系统、\
在登录成功后读取机密数据，然后再退出。

用数据存储接口编写函数
======================

.. Writing a function with the data store
.. ======================================

.. As an example of working with the ``DataStore`` interface, we'll write a
.. function ``getData``, which connects to a store in order to read some data from
.. it. We'll write this function interactively, step by step, using the types of
.. the operations to guide its development. It has the following type:

我们以编写 ``getData`` 函数为例，展示如何使用 ``DataStore`` 接口。
该函数用于连接到存储系统并从中读取数据。我们使用该操作的类型来逐步指导开发，
交互式地编写此函数。它的类型如下：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) => ST m () []

.. This type means that there are no resources available on entry or exit.
.. That is, the overall list of actions is ``[]``, meaning that at least
.. externally, the function has no overall effect on the resources. In other
.. words, for every resource we create during ``getData``, we'll also need to
.. delete it before exit.

该类型表示在进入或退出时没有资源可用。也就是说，整个活动列表为 ``[]``，\
这表示至少从外部来说，该函数完全没有对资源产生作用。换句话说，对于每一个在调用
``getData`` 时创建的资源，我们都需要在退出前删除它。

.. Since we want to use methods of the ``DataStore`` interface, we'll
.. constraint the computation context ``m`` so that there must be an
.. implementation of ``DataStore``. We also have a constraint ``ConsoleIO m``
.. so that we can display any data we read from the store, or any error
.. messages.

由于我们要使用 ``DataStore`` 接口的方法，因此必须约束计算上下文 ``m``
使其实现 ``DataStore`` 接口。我们还有一个 ``ConsoleIO m`` 约束，\
这样就能将我们从存储系统中读取的任何数据或者错误信息显示出来。

.. We start by connecting to the store, creating a new resource ``st``, then
.. trying to ``login``:

我们从连接到存储系统开始，创建一个新的资源 ``st``，然后尝试用 ``login`` 登录：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) => ST m () []
  getData = do st <- connect
               ok <- login st
               ?whatNow

.. Logging in will either succeed or fail, as reflected by the value of
.. ``ok``. If we check the type of ``?whatNow``, we'll see what state the
.. store currently has:

登录可能成功也可能失败，两种状态可从 ``ok`` 值反映出来。如果我们检查 ``?whatNow``
的类型，就会看到当前存储系统的状态：

.. code-block:: idris

      m : Type -> Type
      constraint : ConsoleIO m
      constraint1 : DataStore m
      st : Var
      ok : LoginResult
    --------------------------------------
    whatNow : STrans m () [st ::: Store (case ok of
                                              OK => LoggedIn
                                              BadPassword => LoggedOut)]
                          (\result => [])

.. The current state of ``st`` therefore depends on the value of ``ok``,
.. meaning that we can make progress by case splitting on ``ok``:

由于 ``st`` 的当前状态依赖于 ``ok`` 的值，因此我们可以对 ``ok`` 分情况讨论来继续推进：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) => ST m () []
  getData = do st <- connect
               ok <- login st
               case ok of
                    OK => ?whatNow_1
                    BadPassword => ?whatNow_2

.. The types of the holes in each branch, ``?whatNow_1`` and ``?whatNow_2``,
.. show how the state changes depending on whether logging in was successful.
.. If it succeeded, the store is ``LoggedIn``:

两个分支上的坑 ``?whatNow_1`` 和 ``?whatNow_2`` 的类型展现了状态是如何随着\
登录成功与否而改变的。如果登录成功，那么该存储系统的状态为 ``LoggedIn``:

.. code-block:: idris

    --------------------------------------
    whatNow_1 : STrans m () [st ::: Store LoggedIn] (\result => [])

.. On the other hand, if it failed, the store is ``LoggedOut``:

如果失败，那么它的状态为 ``LoggedOut``:

.. code-block:: idris

    --------------------------------------
    whatNow_2 : STrans m () [st ::: Store LoggedOut] (\result => [])

.. In ``?whatNow_1``, since we've successfully logged in, we can now read
.. the secret and display it to the console:

在 ``?whatNow_1`` 中，由于登录成功，因此可以读取机密数据并将它显示在终端上：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) => ST m () []
  getData = do st <- connect
               ok <- login st
               case ok of
                    OK => do secret <- readSecret st
                             putStrLn ("Secret is: " ++ show secret)
                             ?whatNow_1
                    BadPassword => ?whatNow_2

.. We need to finish the ``OK`` branch with no resources available. We can
.. do this by logging out of the store then disconnecting:

我们要以「无资源可用」的状态来结束 ``OK`` 分支，因此需要退出存储系统并断开连接：

.. code-block:: idris

  getData : (ConsoleIO m, DataStore m) => ST m () []
  getData = do st <- connect
               ok <- login st
               case ok of
                    OK => do secret <- readSecret st
                             putStrLn ("Secret is: " ++ show secret)
                             logout st
                             disconnect st
                    BadPassword => ?whatNow_2

.. Note that we *must* ``logout`` of ``st`` before calling ``disconnect``,
.. because ``disconnect`` requires that the store is in the ``LoggedOut``
.. state.

注意我们在调用 ``disconnect`` 断开连接前，\ **必须**\ 用 ``logout`` 退出
``st``，因为 ``disconnect`` 需要存储系统处于 ``LoggedOut`` 状态。

.. Furthermore, we can't simply use ``delete`` to remove the resource, as
.. we did with the ``State`` examples in the previous section, because
.. ``delete`` only works when the resource has type ``State ty``, for some
.. type ``ty``. If we try to use ``delete`` instead of ``disconnect``, we'll
.. see an error message like the following:

此外，我们不能像上一节中 ``State`` 的示例那样，简单地用 ``delete`` 来删除该资源，\
因为对于某个类型 ``ty`` 来说，``delete`` 只能在资源的类型为 ``State ty`` 时起效。\
如果我们试图用 ``delete`` 来代替 ``disconnect``，就会看到以下错误：

.. code-block:: idris

    When checking argument prf to function Control.ST.delete:
            Can't find a value of type
                    InState st (State st) [st ::: Store LoggedOut]

.. In other words, the type checker can't find a proof that the resource
.. ``st`` has a type of the form ``State st``, because its type is
.. ``Store LoggedOut``. Since ``Store`` is part of the ``DataStore`` interface,
.. we *can't* yet know the concrete representation of the ``Store``, so we
.. need to remove the resource via the interface, with ``disconnect``, rather
.. than directly with ``delete``.

换句话说，类型检查器找不到一个「资源 ``st`` 拥有 ``State st`` 形式的类型」的证明，\
因为其类型为 ``Store LoggedOut``。由于 ``Store`` 是 ``DataStore`` 接口的一部分，\
而我们\ **尚未知道**\  ``Store`` 的具体表示，因此我们需要通过此接口的 ``disconnect``
而非直接用 ``delete`` 来删除资源。

.. We can complete ``getData`` as follows, using a pattern matching bind
.. alternative (see the Idris tutorial, :ref:`monadsdo`) rather than a
.. ``case`` statement to catch the possibility of an error with ``login``:

我们可以将 ``getData`` 完成如下，使用模式匹配来绑定候选（见 Idris 教程的\
:ref:`monadsdo`），而非使用 ``case`` 语句来捕获 ``login`` 可能产生的错误：

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

.. We can't yet try this out, however, because we don't have any implementations
.. of ``DataStore``! If we try to execute it in an ``IO`` context, for example,
.. we'll get an error saying that there's no implementation of ``DataStore IO``:

然而它现在还跑不起来，因为我们还没有任何 ``DataStore`` 的实现！如果我们试着在一个
``IO`` 上下文中执行它，就会产生一个没有 ``DataStore IO`` 的实现的错误：

.. code::

    *Login> :exec run {m = IO} getData
    When checking an application of function Control.ST.run:
            Can't find implementation for DataStore IO

.. The final step in implementing a data store which correctly follows the
.. state transition diagram, therefore, is to provide an implementation
.. of ``DataStore``.

因此，要实现遵循其状态转移图的数据存储系统，最后一步就是提供一个 ``DataStore``
的实现。

实现接口
========

.. Implementing the interface
.. ==========================

.. To execute ``getData`` in ``IO``, we'll need to provide an implementation
.. of ``DataStore`` which works in the ``IO`` context. We can begin as
.. follows:

要在 ``IO`` 中执行 ``getData``，我们需要提供一个能够在 ``IO`` 上下文中工作的
``DataStore`` 的实现。我们可以这样开始：

.. code-block:: idris

  implementation DataStore IO where

.. Then, we can ask Idris to populate the interface with skeleton definitions
.. for the necessary methods (press ``Ctrl-Alt-A`` in Atom for "add definition"
.. or the corresponding shortcut for this in the Idris mode in your favourite
.. editor):

接着，我们可以让 Idris 根据必要方法的基本定义来填充该接口（在 Atom 中按下
``Ctrl-Alt-A``，或者在你喜欢的编辑器中按下对应的快捷键来「添加定义」）：

.. code-block:: idris

  implementation DataStore IO where
    Store x = ?DataStore_rhs_1
    connect = ?DataStore_rhs_2
    disconnect store = ?DataStore_rhs_3
    readSecret store = ?DataStore_rhs_4
    login store = ?DataStore_rhs_5
    logout store = ?DataStore_rhs_6

.. The first decision we'll need to make is how to represent the data store.
.. We'll keep this simple, and store the data as a single ``String``, using
.. a hard coded password to gain access. So, we can define ``Store`` as
.. follows, using a ``String`` to represent the data no matter whether we
.. are ``LoggedOut`` or ``LoggedIn``:

我们首先要确定表示该数据存储系统的方式。为简单起见，我们将数据存储为单个的
``String``，并使用硬编码的密码来获取访问权限。我们可以将 ``Store`` 定义如下，\
无论在 ``LoggedOut`` 还是在 ``LoggedIn`` 状态下均使用 ``String`` 来表示数据。

.. code-block:: idris

    Store x = State String

.. Now that we've given a concrete type for ``Store``, we can implement operations
.. for connecting, disconnecting, and accessing the data. And, since we used
.. ``State``, we can use ``new``, ``delete``, ``read`` and ``write`` to
.. manipulate the store.

现在我们给出了 ``Store`` 的一个具体类型，我们可以实现建立连接、断开连接和访问数据的操作。\
而由于我们使用了 ``State``，因此也就可以使用 ``new``、``delete``、``read`` 和 ``write``
来操作该存储系统。

.. Looking at the types of the holes tells us how we need to manipulate the
.. state. For example, the ``?DataStore_rhs_2`` hole tells us what we need
.. to do to implement ``connect``. We need to return a new ``Var`` which
.. represents a resource of type ``State String``:

坑的类型会告诉我们如何操作状态。例如，``?DataStore_rhs_2`` 坑告诉我们要实现
``connect`` 需要做些什么。我们需要返回一个新的 ``Var``，表示一个类型为
``State String`` 的资源：

.. code-block:: idris

    --------------------------------------
    DataStore_rhs_2 : STrans IO Var [] (\result => [result ::: State String])

.. We can implement this by creating a new variable with some data for the
.. content of the store (we can use any ``String`` for this) and returning
.. that variable:

我们可以通过创建一个带有某些数据作为存储内容的新变量来实现它（我们可以使用任何
``String``），然后返回该变量：

.. code-block:: idris

    connect = do store <- new "Secret Data"
                 pure store

.. For ``disconnect``, we only need to delete the resource:

对于 ``disconnect`` 而言，我们只需删除该资源即可：

.. code-block:: idris

    disconnect store = delete store

.. For ``readSecret``, we need to read the secret data and return the
.. ``String``. Since we now know the concrete representation of the data is
.. a ``State String``, we can use ``read`` to access the data directly:

对于 ``readSecret``，我们需要读取机密数据并返回 ``String``。\
由于我们并不知道该数据的具体表示为 ``State String``，因此可以直接用 ``read``
来访问数据：

.. code-block:: idris

    readSecret store = read store

.. We'll do ``logout`` next and return to ``login``. Checking the hole
.. reveals the following:

我们先来完成 ``logout``，之后回到 ``login`` 上来。检查坑的类型会显示以下信息：

.. code-block:: idris

      store : Var
    --------------------------------------
    DataStore_rhs_6 : STrans IO () [store ::: State String] (\result => [store ::: State String])

.. So, in this minimal implementation, we don't actually have to do anything!

因此在此小型实现中，我们实际上不用做任何事情！

.. code-block:: idris

    logout store = pure ()

.. For ``login``, we need to return whether logging in was successful. We'll
.. do this by prompting for a password, and returning ``OK`` if it matches
.. a hard coded password, or ``BadPassword`` otherwise:

对于 ``login``，我们需要返回登录是否成功。为此，我们需要提示用户输入密码，\
并在匹配到硬编码的密码时返回 ``OK``，否则返回 ``BadPassword``：

.. code-block:: idris

    login store = do putStr "Enter password: "
                     p <- getStr
                     if p == "Mornington Crescent"
                        then pure OK
                        else pure BadPassword

.. For reference, here is the complete implementation which allows us to
.. execute a ``DataStore`` program at the REPL:

下面给出完整的实现以供参考，它能让我们在 REPL 中执行 ``DataStore`` 程序：

.. code-block:: idris

  implementation DataStore IO where
    Store x = State String
    connect = do store <- new "Secret Data"
                 pure store
    disconnect store = delete store
    readSecret store = read store
    login store = do putStr "Enter password: "
                     p <- getStr
                     if p == "Mornington Crescent"
                        then pure OK
                        else pure BadPassword
    logout store = pure ()

.. Finally, we can try this at the REPL as follows (Idris defaults to the
.. ``IO`` context at the REPL if there is an implementation available, so no
.. need to give the ``m`` argument explicitly here):

最后，我们可以像下面这样在 REPL 中尝试它（如果有可用的 ``IO`` 实现，那么在 Idris
的 REPL 中，上下文会默认为 ``IO``，因此这里无需显式给出 ``m`` 参数）：

.. code::

    *Login> :exec run getData
    Enter password: Mornington Crescent
    Secret is: "Secret Data"

    *Login> :exec run getData
    Enter password: Dollis Hill
    Failure

.. We can only use ``read``, ``write``, ``new`` and ``delete`` on a resource
.. with a ``State`` type. So, *within* the implementation of ``DataStore``,
.. or anywhere where we know the context is ``IO``, we can access the data store
.. however we like: this is where the internal details of ``DataStore`` are
.. implemented. However, if we merely have a constraint ``DataStore m``, we can't
.. know how the store is implemented, so we can only access via the API given
.. by the ``DataStore`` interface.

对于 ``State`` 类型的资源，我们只能使用 ``read``、``write``、``new`` 和 ``delete``。\
因此，在 ``DataStore`` 的实现或任何已知上下文为 ``IO`` 的环境\ **内部**\，\
我们可以随意访问数据存储系统，因为这里是实现 ``DataStore`` 内部细节的地方。\
然而，如果我们只有 ``DataStore m`` 的约束，那么就无法知道该存储系统是否已实现，\
因此我们只能通过 ``DataStore`` 提供的 API 来访问它。

.. It is therefore good practice to use a *generic* context ``m`` for functions
.. like ``getData``, and constrain by only the interfaces we need, rather than
.. using a concrete context ``IO``.

因此比较好的做法是在 ``getData`` 这类函数中使用\ **泛型（Generic）**\ 上下文 ``m``，\
并只根据我们需要的接口进行约束，而非使用具体的上下文 ``IO``。

.. We've now seen how to manipulate states, and how to encapsulate state
.. transitions for a specific system like the data store in an interface.
.. However, realistic systems will need to *compose* state machines. We'll
.. either need to use more than one state machine at a time, or implement one
.. state machine in terms of one or more others. We'll see how to achieve this
.. in the next section.

现在我们已经学过如何处理状态，以及如何用接口来封装数据存储这类具体系统的状态转移了。
然而，真正的系统需要能够\ **组合**\ 多种状态机。我们每次需要使用多个状态机，\
或者基于多个状态机来实现一个状态机。

.. |login| image:: ../image/login.png
                   :width: 500px
