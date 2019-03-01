.. _netexample:

**********************
示例：网络 Socket 编程
**********************

.. ***********************************
.. Example: Network Socket Programming
.. ***********************************

.. The POSIX sockets API supports communication between processes across a
.. network. A *socket* represents an endpoint of a network communication, and can be
.. in one of several states:

.. * ``Ready``, the initial state
.. * ``Bound``, meaning that it has been bound to an address ready for incoming
..   connections
.. * ``Listening``, meaning that it is listening for incoming connections
.. * ``Open``, meaning that it is ready for sending and receiving data;
.. * ``Closed``, meaning that it is no longer active.

POSIX 的插口（Socket，另译作「套接字」）API 支持跨网络的进程间通信。\ **插口**\
表示网络通信的端点，它可以是以下几种状态之一：

* ``Ready`` 为初始状态
* ``Bound`` 表示已绑定至某个地址，准备接受传入的连接
* ``Listening`` 表示正在监听传入的连接
* ``Open`` 表示准备发送或接受数据
* ``Closed`` 表示不再活动

.. The following diagram shows how the operations provided by the API modify the
.. state, where ``Ready`` is the initial state:

下图展示了该 API 提供的操作是如何改变其状态的，其中 ``Ready`` 为初始状态：

|netstate|

.. If a connection is ``Open``, then we can also ``send`` messages to the
.. other end of the connection, and ``recv`` messages from it.

如果某个连接为 ``Open`` 状态，那么我们可以用 ``send`` 发送消息到该连接的另一端，\
也可以用 ``recv`` 从另一端接收消息。

.. The ``contrib`` package provides a module ``Network.Socket`` which
.. provides primitives for creating sockets and sending and receiving
.. messages. It includes the following functions:

``contrib`` 包提供了一个 ``Network.Socket`` 模块，该模块提供了创建插口\
以及发送和接收消息的原语。其中包含以下函数：

.. code-block:: idris

    bind : (sock : Socket) -> (addr : Maybe SocketAddress) -> (port : Port) -> IO Int
    connect : (sock : Socket) -> (addr : SocketAddress) -> (port : Port) -> IO ResultCode
    listen : (sock : Socket) -> IO Int
    accept : (sock : Socket) -> IO (Either SocketError (Socket, SocketAddress))
    send : (sock : Socket) -> (msg  : String) -> IO (Either SocketError ResultCode)
    recv : (sock : Socket) -> (len : ByteLength) -> IO (Either SocketError (String, ResultCode))
    close : Socket -> IO ()

.. These functions cover the state transitions in the diagram above, but
.. none of them explain how the operations affect the state! It's perfectly
.. possible, for example, to try to send a message on a socket which is
.. not yet ready, or to try to receive a message after the socket is closed.

这些函数虽然涵盖了上图中所有的状态转移，然而却均未解释这些操作是如何影响其状态的！\
例如，我们完全有可能在一个尚未就绪的插口上发送消息，或者在插口关闭后从中接收消息。

.. Using ``ST``, we can provide a better API which explains exactly how
.. each operation affects the state of a connection. In this section, we'll
.. define a sockets API, then use it to implement an "echo" server which
.. responds to requests from a client by echoing back a single message sent
.. by the client.

我们可以用 ``ST`` 提供更好的 API，它精确地解释了每个操作是如何影响连接的状态的。\
在本节中，我们会定义一个插口 API，然后用它来实现一个「回显（echo）」服务器，通过\
原样返回客户端发送的消息来响应客户端的请求。

定义 ``Sockets`` 接口
=====================

.. Defining a ``Sockets`` interface
.. ================================

.. Rather than using ``IO`` for low level socket programming, we'll implement
.. an interface using ``ST`` which describes precisely how each operation
.. affects the states of sockets, and describes when sockets are created
.. and removed. We'll begin by creating a type to describe the abstract state
.. of a socket:

我们不直接用 ``IO`` 进行底层插口编程，而是用 ``ST`` 实现一个接口来精确地描述\
每个操作如何影响插口的状态，以及何时创建或删除插口。我们首先来创建描述插口\
抽象状态的类型：

.. code-block:: idris

  data SocketState = Ready | Bound | Listening | Open | Closed

.. Then, we'll begin defining an interface, starting with a ``Sock`` type
.. for representing sockets, parameterised by their current state:

接着定义一个接口，用 ``Sock`` 类型表示插口，以其当前状态作为该类型构造子的参数：

.. code-block:: idris

  interface Sockets (m : Type -> Type) where
    Sock : SocketState -> Type

.. We create sockets using the ``socket`` method. The ``SocketType`` is defined
.. by the sockets library, and describes whether the socket is TCP, UDP,
.. or some other form. We'll use ``Stream`` for this throughout, which indicates a
.. TCP socket.

我们用 ``socket`` 方法创建插口。插口库中定义了 ``SocketType``，它描述了插口为
TCP、UDP 还是其它形式。我们后面会一直使用 ``Stream`` 来表示 TCP 插口。

.. code-block:: idris

    socket : SocketType -> ST m (Either () Var) [addIfRight (Sock Ready)]

.. Remember that ``addIfRight`` adds a resource if the result of the operation
.. is of the form ``Right val``. By convention in this interface, we'll use
.. ``Either`` for operations which might fail, whether or not they might carry
.. any additional information about the error, so that we can consistently
.. use ``addIfRight`` and some other type level functions.

记住 ``addIfRight`` 会在操作结果的形式为 ``Right val`` 时添加资源。按照此接口的约定，\
我们用 ``Either`` 来表示可能失败的操作，它可以保存任何关于错误的附加信息。\
如此一来，我们就能一致地使用 ``addIfRight`` 和其它类型级函数了。

.. To define a server, once we've created a socket, we need to ``bind`` it
.. to a port. We can do this with the ``bind`` method:

现在来定义一个服务器。一旦我们创建了插口，就需要用 ``bind`` 方法将它绑定到一个端口上：


.. code-block:: idris

    bind : (sock : Var) -> (addr : Maybe SocketAddress) -> (port : Port) ->
           ST m (Either () ()) [sock ::: Sock Ready :-> (Sock Closed `or` Sock Bound)]

.. Binding a socket might fail, for example if there is already a socket
.. bound to the given port, so again it returns a value of type ``Either``.
.. The action here uses a type level function ``or``, and says that:

.. * If ``bind`` fails, the socket moves to the ``Sock Closed`` state
.. * If ``bind`` succeeds, the socket moves to the ``Sock Bound`` state, as
..   shown in the diagram above

绑定插口可能会失败，例如如果已经有一个插口被绑定到给定的端口上，因此\
它同样要返回一个类型为 ``Either`` 的值。这里的动作使用了类型级函数 ``or``，\
它的意思是：

* 若 ``bind`` 失败，插口就会转移到 ``Sock Closed`` 状态
* 若 ``bind`` 成功，插口就会转移到 ``Sock Bound`` 状态，如前图所示

``or`` 的实现如下：

.. code-block:: idris

    or : a -> a -> Either b c -> a
    or x y = either (const x) (const y)

.. So, the type of ``bind`` could equivalently be written as:

这样一来，``bind`` 的类型就能写成如下等价形式：

.. code-block:: idris

    bind : (sock : Var) -> (addr : Maybe SocketAddress) -> (port : Port) ->
           STrans m (Either () ()) [sock ::: Sock Ready]
                        (either [sock ::: Sock Closed] [sock ::: Sock Bound])

.. However, using ``or`` is much more concise than this, and attempts to
.. reflect the state transition diagram as directly as possible while still
.. capturing the possibility of failure.

然而，使用 ``or`` 会更加简洁，它也能尽可能直接地反映出状态转移图，\
同时仍然刻画了它可能失败的性质。

.. Once we've bound a socket to a port, we can start listening for connections
.. from clients:

一旦我们将插口绑定到了端口，就可以开始监听来自客户端的连接了：

.. code-block:: idris

    listen : (sock : Var) ->
             ST m (Either () ()) [sock ::: Sock Bound :-> (Sock Closed `or` Sock Listening)]

.. A socket in the ``Listening`` state is ready to accept connections from
.. individual clients:

``Listening`` 状态的插口表示准备接受来自独立客户端的连接：

.. code-block:: idris

    accept : (sock : Var) ->
             ST m (Either () Var)
                  [sock ::: Sock Listening, addIfRight (Sock Open)]

.. If there is an incoming connection from a client, ``accept`` adds a *new*
.. resource to the end of the resource list (by convention, it's a good idea
.. to add resources to the end of the list, because this works more tidily
.. with ``updateWith``, as discussed in the previous section). So, we now
.. have *two* sockets: one continuing to listen for incoming connections,
.. and one ready for communication with the client.

如果有客户端传入的连接，``accept`` 会在资源列表的最后添加一个\ **新的**\
资源（按照约定，在列表末尾添加资源可以更好地配合 ``updateWith`` 工作，如上一节所述）。
现在，我们有了\ **两个**\ 插口：一个继续监听传入的连接，另一个准备与客户端通信。

.. We also need methods for sending and receiving data on a socket:

我们还需要能够在插口上发送和接受数据的方法：

.. code-block:: idris

    send : (sock : Var) -> String ->
           ST m (Either () ()) [sock ::: Sock Open :-> (Sock Closed `or` Sock Open)]
    recv : (sock : Var) ->
           ST m (Either () String) [sock ::: Sock Open :-> (Sock Closed `or` Sock Open)]

.. Once we've finished communicating with another machine via a socket, we'll
.. want to ``close`` the connection and remove the socket:

一旦我们与另一台机器通过插口进行的通信结束， 就需要用 ``close`` 关闭连接并用
``remove`` 移除该插口：

.. code-block:: idris

    close : (sock : Var) ->
            {auto prf : CloseOK st} -> ST m () [sock ::: Sock st :-> Sock Closed]
    remove : (sock : Var) ->
             ST m () [Remove sock (Sock Closed)]

.. We have a predicate ``CloseOK``, used by ``close`` in an implicit proof
.. argument, which describes when it is okay to close a socket:

``close`` 使用了断言 ``CloseOK`` 作为隐式证明参数，它描述了何时可以关闭插口：

.. code-block:: idris

  data CloseOK : SocketState -> Type where
       CloseOpen : CloseOK Open
       CloseListening : CloseOK Listening

.. That is, we can close a socket which is ``Open``, talking to another machine,
.. which causes the communication to terminate.  We can also close a socket which
.. is ``Listening`` for incoming connections, which causes the server to stop
.. accepting requests.

也就是说，我们可以关闭 ``Open`` 状态的插口，告诉另一台机器终止通信。我们也可以\
关闭 ``Listening`` 状态下等待传入连接的插口，这会让服务器停止接受请求。

.. In this section, we're implementing a server, but for completeness we may
.. also want a client to connect to a server on another machine. We can do
.. this with ``connect``:

在本节中，我们实现了一个服务器，不过为了完整性，我们还需要在另一台机器上实现\
客户端来连接到服务器。这可以通过 ``connect`` 来完成：

.. code-block:: idris

    connect : (sock : Var) -> SocketAddress -> Port ->
              ST m (Either () ()) [sock ::: Sock Ready :-> (Sock Closed `or` Sock Open)]

.. For reference, here is the complete interface:

以下完整的接口作为参考：

.. code-block:: idris

  interface Sockets (m : Type -> Type) where
    Sock : SocketState -> Type
    socket : SocketType -> ST m (Either () Var) [addIfRight (Sock Ready)]
    bind : (sock : Var) -> (addr : Maybe SocketAddress) -> (port : Port) ->
           ST m (Either () ()) [sock ::: Sock Ready :-> (Sock Closed `or` Sock Bound)]
    listen : (sock : Var) ->
             ST m (Either () ()) [sock ::: Sock Bound :-> (Sock Closed `or` Sock Listening)]
    accept : (sock : Var) ->
             ST m (Either () Var) [sock ::: Sock Listening, addIfRight (Sock Open)]
    connect : (sock : Var) -> SocketAddress -> Port ->
              ST m (Either () ()) [sock ::: Sock Ready :-> (Sock Closed `or` Sock Open)]
    close : (sock : Var) -> {auto prf : CloseOK st} ->
            ST m () [sock ::: Sock st :-> Sock Closed]
    remove : (sock : Var) -> ST m () [Remove sock (Sock Closed)]
    send : (sock : Var) -> String ->
           ST m (Either () ()) [sock ::: Sock Open :-> (Sock Closed `or` Sock Open)]
    recv : (sock : Var) ->
           ST m (Either () String) [sock ::: Sock Open :-> (Sock Closed `or` Sock Open)]

.. We'll see how to implement this shortly; mostly, the methods can be implemented
.. in ``IO`` by using the raw sockets API directly. First, though, we'll see
.. how to use the API to implement an "echo" server.

我们稍后会看到如何实现它。这些方法大部分都可以直接通过原始的插口 API 在 ``IO``
中实现。不过首先，我们会看到如何用这些 API 实现一个「回显」服务器。

用 ``Sockets`` 实现「回显」服务器
=================================

.. Implementing an "Echo" server with ``Sockets``
.. ==============================================

.. At the top level, our echo server begins and ends with no resources available,
.. and uses the ``ConsoleIO`` and ``Sockets`` interfaces:

从顶层来说，我们的回显（echo）服务器在开始和结束时均没有资源可用，它使用了
``ConsoleIO`` 和 ``Sockets`` 接口：

.. code-block:: idris

  startServer : (ConsoleIO m, Sockets m) => ST m () []

.. The first thing we need to do is create a socket for binding to a port
.. and listening for incoming connections, using ``socket``. This might fail,
.. so we'll need to deal with the case where it returns ``Right sock``, where
.. ``sock`` is the new socket variable, or where it returns ``Left err``:

首先我们需要用 ``socket`` 创建一个插口，绑定到一个端口并监听传入的连接。\
它可能会失败，因此我们需要处理它返回 ``Right sock`` 的情况，其中 ``sock``
是新的插口变量，不过也可能返回 ``Left err``：

.. code-block:: idris

  startServer : (ConsoleIO m, Sockets m) => ST m () []
  startServer =
    do Right sock <- socket Stream
             | Left err => pure ()
       ?whatNow

.. It's a good idea to implement this kind of function interactively, step by
.. step, using holes to see what state the overall system is in after each
.. step. Here, we can see that after a successful call to ``socket``, we
.. have a socket available in the ``Ready`` state:

交互式地实现这类函数是个不错的想法，我们可以通过挖坑来逐步观察整个系统的状态\
是如何变化的。在成功调用了 ``socket`` 之后，我们就有了 ``Ready`` 状态的插口：

.. code-block:: idris

      sock : Var
      m : Type -> Type
      constraint : ConsoleIO m
      constraint1 : Sockets m
    --------------------------------------
    whatNow : STrans m () [sock ::: Sock Ready] (\result1 => [])

.. Next, we need to bind the socket to a port, and start listening for
.. connections. Again, each of these could fail. If they do, we'll remove
.. the socket. Failure always results in a socket in the ``Closed`` state,
.. so all we can do is ``remove`` it:

接着，我们需要将插口绑定到端口，然后开始监听连接。同样，每一步都可能会失败，\
此时我们会移除该插口。失败总会导致插口转为 ``Closed`` 状态，此时我们能做的就是用
``remove`` 移除它：

.. code-block:: idris

  startServer : (ConsoleIO m, Sockets m) => ST m () []
  startServer =
    do Right sock <- socket Stream        | Left err => pure ()
       Right ok <- bind sock Nothing 9442 | Left err => remove sock
       Right ok <- listen sock            | Left err => remove sock
       ?runServer

.. Finally, we have a socket which is listening for incoming connections:

最后，我们就有了一个监听传入连接的插口：

.. code-block:: idris

      ok : ()
      sock : Var
      ok1 : ()
      m : Type -> Type
      constraint : ConsoleIO m
      constraint1 : Sockets m
    --------------------------------------
    runServer : STrans m () [sock ::: Sock Listening]
                       (\result1 => [])

.. We'll implement this in a separate function. The type of ``runServer``
.. tells us what the type of ``echoServer`` must be (noting that we need
.. to give the ``m`` argument to ``Sock`` explicitly):

我们会在一个独立的函数中实现它。``runServer`` 的类型告诉我们 ``echoServer``
的类型必须是什么（我们无需显式地为 ``Sock`` 给出参数 ``m``）：

.. code-block:: idris

  echoServer : (ConsoleIO m, Sockets m) => (sock : Var) ->
               ST m () [remove sock (Sock {m} Listening)]

.. We can complete the definition of ``startServer`` as follows:

我们可以完成 ``startServer`` 的定义：

.. code-block:: idris

  startServer : (ConsoleIO m, Sockets m) => ST m () []
  startServer =
    do Right sock <- socket Stream        | Left err => pure ()
       Right ok <- bind sock Nothing 9442 | Left err => remove sock
       Right ok <- listen sock            | Left err => remove sock
       echoServer sock

.. In ``echoServer``, we'll keep accepting requests and responding to them
.. until something fails, at which point we'll close the sockets and
.. return. We begin by trying to accept an incoming connection:

在 ``echoServer`` 中，我们会继续接受并相应请求直到出现失败，此时我们会关闭插口并返回。\
我们从尝试接受传入的连接开始：

.. code-block:: idris

  echoServer : (ConsoleIO m, Sockets m) => (sock : Var) ->
               ST m () [remove sock (Sock {m} Listening)]
  echoServer sock =
    do Right new <- accept sock | Left err => do close sock; remove sock
       ?whatNow

.. If ``accept`` fails, we need to close the ``Listening`` socket and
.. remove it before returning, because the type of ``echoServer`` requires
.. this.

若 ``accept`` 失败，我们就需要关闭 ``Listening`` 状态的插口并在返回前移除它，因为
``echoServer`` 的类型要求如此。

.. As always, implementing ``echoServer`` incrementally means that we can check
.. the state we're in as we develop. If ``accept`` succeeds, we have the
.. existing ``sock`` which is still listening for connections, and a ``new``
.. socket, which is open for communication:

通常，逐步实现 ``echoServer`` 意味着我们可以在开发过程中检查当前的状态。如果
``accept`` 成功，那么我们既有的 ``sock`` 会继续监听连接，此外一个新的 ``new``
插口会被打开用于通信：

.. code-block:: idris

      new : Var
      sock : Var
      m : Type -> Type
      constraint : ConsoleIO m
      constraint1 : Sockets m
    --------------------------------------
    whatNow : STrans m () [sock ::: Sock Listening, new ::: Sock Open]
                          (\result1 => [])

.. To complete ``echoServer``, we'll receive a message on the ``new``
.. socket, and echo it back. When we're done, we close the ``new`` socket,
.. and go back to the beginning of ``echoServer`` to handle the next
.. connection:

要完成 ``echoServer``，我们需要从 ``new`` 插口上接收一条消息，然后原样返回它。\
在完成后，我们就关闭 ``new`` 插口，然后回到 ``echoServer`` 的开始处，准备响应下一次连接：

.. code-block:: idris

  echoServer : (ConsoleIO m, Sockets m) => (sock : Var) ->
               ST m () [remove sock (Sock {m} Listening)]
  echoServer sock =
    do Right new <- accept sock | Left err => do close sock; remove sock
       Right msg <- recv new | Left err => do close sock; remove sock; remove new
       Right ok <- send new ("You said " ++ msg)
             | Left err => do remove new; close sock; remove sock
       close new; remove new; echoServer sock

实现 ``Sockets``
================

.. Implementing ``Sockets``
.. ========================

.. To implement ``Sockets`` in ``IO``, we'll begin by giving a concrete type
.. for ``Sock``. We can use the raw sockets API (implemented in
.. ``Network.Sockeet``) for this, and use a ``Socket`` stored in a ``State``, no
.. matter what abstract state the socket is in:

为了在 ``IO`` 中实现 ``Sockets``，我们需要从给出具体的 ``Sock`` 类型开始。\
我们可以用原始的插口 API（在 ``Network.Socket`` 中实现），将 ``Socket`` 存储在
``State`` 中来得到具体的类型，而不必关心该插口所处的抽象状态：

.. code-block:: idris

  implementation Sockets IO where
    Sock _ = State Socket

.. Most of the methods can be implemented by using the raw socket API
.. directly, returning ``Left`` or ``Right`` as appropriate. For example,
.. we can implement ``socket``, ``bind`` and ``listen`` as follows:

大部分方法都可以直接用原始的插口 API 来实现，返回相应的 ``Left`` 或 ``Right``。\
例如，我们可以实现 ``socket``、``bind`` 和 ``listen``：

.. code-block:: idris

    socket ty = do Right sock <- lift $ Socket.socket AF_INET ty 0
                        | Left err => pure (Left ())
                   lbl <- new sock
                   pure (Right lbl)
    bind sock addr port = do ok <- lift $ bind !(read sock) addr port
                             if ok /= 0
                                then pure (Left ())
                                else pure (Right ())
    listen sock = do ok <- lift $ listen !(read sock)
                     if ok /= 0
                        then pure (Left ())
                        else pure (Right ())

.. There is a small difficulty with ``accept``, however, because when we
.. use ``new`` to create a new resource for the open connection, it appears
.. at the *start* of the resource list, not the end. We can see this by
.. writing an incomplete definition, using ``returning`` to see what the
.. resources need to be if we return ``Right lbl``:

然而，这里的 ``accept`` 有点不同， 因为我们在用 ``new`` 为打开连接创建新资源时，\
它出现在了资源列表的\ **起始**\ 处而非末尾。我们可以通过写出不完整的定义来看到这一点，
使用 ``returning`` 来查看返回 ``Right lbl`` 需要什么资源：

.. code-block:: idris

    accept sock = do Right (conn, addr) <- lift $ accept !(read sock)
                           | Left err => pure (Left ())
                     lbl <- new conn
                     returning (Right lbl) ?fixResources

.. It's convenient for ``new`` to add the resource to the beginning of the
.. list because, in general, this makes automatic proof construction with
.. an ``auto``-implicit easier for Idris. On the other hand, when we use
.. ``call`` to make a smaller set of resources, ``updateWith`` puts newly
.. created resources at the *end* of the list, because in general that reduces
.. the amount of re-ordering of resources.

使用 ``new`` 将资源添加到列表起始处是很方便的，因为通常来说，这会让 Idris 使用隐式
``auto`` 自动构造证明更加容易。另一方面，当我们用 ``call`` 来构造更小的资源集合时，\
``updateWith`` 会将新创建的资源放到列表的\ **末尾**\ 处，因为通常这样会减少\
需要重新排序的资源的数量。

.. If we look at the type of
.. ``fixResources``, we can see what we need to do to finish ``accept``:

如果我们查看 ``fixResources`` 的类型，就会知道结束 ``accept`` 需要做的事情：

.. code-block:: idris

      _bindApp0 : Socket
      conn : Socket
      addr : SocketAddress
      sock : Var
      lbl : Var
    --------------------------------------
    fixResources : STrans IO () [lbl ::: State Socket, sock ::: State Socket]
                          (\value => [sock ::: State Socket, lbl ::: State Socket])

.. The current list of resources is ordered ``lbl``, ``sock``, and we need them
.. to be in the order ``sock``, ``lbl``. To help with this situation,
.. ``Control.ST`` provides a primitive ``toEnd`` which moves a resource to the
.. end of the list. We can therefore complete ``accept`` as follows:

当前资源列表的顺序为 ``lbl``、``sock``，而我们需要它们的顺序变成 ``sock``、``lbl``。\
为此，``Control.ST`` 提供了原语 ``toEnd``，它会将一个资源移到列表的末尾。\
这样我们就能完成 ``accept`` 了：

.. code-block:: idris

    accept sock = do Right (conn, addr) <- lift $ accept !(read sock)
                           | Left err => pure (Left ())
                     lbl <- new conn
                     returning (Right lbl) (toEnd lbl)

.. For the complete implementation of ``Sockets``, take a look at
.. ``samples/ST/Net/Network.idr`` in the Idris distribution. You can also
.. find the complete echo server there, ``EchoServer.idr``. There is also
.. a higher level network protocol, ``RandServer.idr``, using a hierarchy of
.. state machines to implement a high level network communication protocol
.. in terms of the lower level sockets API. This also uses threading, to
.. handle incoming requests asynchronously. You can find some more detail
.. on threading and the random number server in the draft paper
.. `State Machines All The Way Down <https://www.idris-lang.org/drafts/sms.pdf>`_
.. by Edwin Brady.

``Sockets`` 的完整实现见 Idris 发行版中的 ``samples/ST/Net/Network.idr`` 文件。\
你也可以在同目录下的 ``EchoServer.idr`` 文件中找到回显服务器。此外还有一个高级\
网络协议 ``RandServer.idr``，它基于底层的插口 API，通过状态机的层级实现了一个\
高级的网络通信协议。它还使用线程来异步地处理传入的请求。你可以在 Edwin Brady
的文稿 `State Machines All The Way Down <https://www.idris-lang.org/drafts/sms.pdf>`_
（深入理解状态机）中找到关于线程和随机数服务器的更多详情。

.. |netstate| image:: ../image/netstate.png
                      :width: 300px

