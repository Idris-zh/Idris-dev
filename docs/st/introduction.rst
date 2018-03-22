********
Overview
********

Pure functional languages with dependent types such as `Idris
<http://www.idris-lang.org/>`_ support reasoning about programs directly
in the type system, promising that we can *know* a program will run
correctly (i.e. according to the specification in its type) simply
because it compiles.

像 `Idris <http://www.idris-lang.org/>` 这样支持直接在类型系统中进行程序推理
的函数式语言，仅根据一个程序能够编译这一点，就能保证我们能够 *知道* 一个程序将会
正确地 (即: 根据其类型说明) 运行。

Realistically, though,  software relies on state, and many components rely on state machines. For
example, they describe network transport protocols like TCP, and
implement event-driven systems and regular expression matching. Furthermore,
many fundamental resources like network sockets and files are, implicitly,
managed by state machines, in that certain operations are only valid on
resources in certain states, and those operations can change the states of the
underlying resource. For example, it only makes sense to send a message on a
connected network socket, and closing a socket changes its state from "open" to
"closed". State machines can also encode important security properties. For
example, in the software which implements an ATM, it’s important that the ATM
dispenses cash only when the machine is in a state where a card has been
inserted and the PIN verified.

然而，事实上，软件依赖状态，并且很多组件依赖状态机。比如 描述 TCP 这样的网络传输协议,
实现事件驱动的系统，以及匹配正则表达式。更进一步说，很多基础资源，像网络套接字 (network socket)
和文件，都隐式地被状态机所管理，以使特定操作只有在资源到达特定状态时才合法，并且这些操作
也能够改变底层资源的状态。例如，只有向已建立连接的网络套接字发送消息才有意义，且关闭一个
套接字会将其状态从 "打开" 变为 "关闭"。状态机同样可以编码重要的安全性质。比如，在实现一个
ATM (自动取款机) 的软件中，非常重要的一个性质是，ATM 只有在卡片插入且通过密码验证的状态下
才能吐出钞票。

In this tutorial we will introduce the ``Control.ST`` library, which is included
with the Idris distribution (currently as part of the ``contrib`` package)
and supports programming and reasoning with state and side effects.  This
tutorial assumes familiarity with pure programming in Idris, as described in
:ref:`tutorial-index`.
For further background information, the ``ST`` library is based on ideas
discussed in Chapter 13 (available as a free sample chapter) and Chapter 14
of `Type-Driven Development with Idris <https://www.manning.com/books/type-driven-development-with-idris>`_.

这个教程将介绍  ``Control.ST`` 库，它已经被包含在 Idris 发布版当中 (目前是 ``contrib`` 包
的一部分) 并且支持编程、推理状态和副作用。这个教程假设读者已经熟悉 Idris 中的纯函数式编程,
如以下章节所描述 :ref:`tutorial-index` 。
``ST`` 库是基于 `Type-Driven Development with Idris <https://www.manning.com/books/type-driven-development-with-idris>`_ 一书的 13 章和 14 章所讨论的内容. 如果需要更多背景信息可以参考此书。

The ``ST`` library allows us to write programs which are composed of multiple
state transition systems. It supports composition in two ways: firstly, we can
use several independently implemented state transition systems at once;
secondly, we can implement one state transition system in terms of others.

``ST`` 库允许我们写由许多状态转移系统所组合的程序。它支持两种组合: 第一，我们能够同时
使用数个独立实现的状态转移系统; 第二，我们能够基于其它状态转移系统实现一个新的。

Introductory example: a data store requiring a login
====================================================

示例: 一个必须登录的数据存储程序
================================

Many software components rely on some form of state, and there may be
operations which are only valid in specific states. For example, consider
a secure data store in which a user must log in before getting access to
some secret data. This system can be in one of two states:

许多软件组件依赖一些状态，可能会有一些操作仅在某些特定状态下才合法。比如，考虑
一个安全的数据存储，它只有在用户登录的情况下才能访问某些机密数据。这个系统可以
处于以下两个状态之一:

* ``LoggedIn``, in which the user is allowed to read the secret
* ``LoggedOut``, in which the user has no access to the secret

* ``已登录`` ，在该状态下，用户被允许访问机密数据
* ``未登录`` ，在该状态下，用户无法访问机密数据

We can provide commands to log in, log out, and read the data, as illustrated
in the following diagram:

我们能提供 登录，登出，读取数据 命令，如以下图表所展示的:

|login|

The ``login`` command, if it succeeds, moves the overall system state from
``LoggedOut`` to ``LoggedIn``. The ``logout`` command moves the state from
``LoggedIn`` to ``LoggedOut``. Most importantly, the ``readSecret`` command
is only valid when the system is in the ``LoggedIn`` state.

``登录`` 命令，如果被成功执行，将使系统状态从 ``未登录`` 转移到 ``已登录`` 。
``登出`` 命令，使系统状态从 ``已登录`` 转移到 ``未登录`` 。最重要的是，``读取数据``
命令仅在系统处于 ``已登录`` 状态时才合法。

We routinely use type checkers to ensure that variables and arguments are used
consistently. However, statically checking that operations are performed only
on resources in an appropriate state is not well supported by mainstream type
systems. In the data store example, for example, it's important to check that
the user is successfully logged in before using ``readSecret``. The
``ST`` library allows us to represent this kind of *protocol* in the type
system, and ensure at *compile-time* that the secret is only read when the
user is logged in.

我们一如既往地使用类型检查器来保证变量和参数被一致地使用。然而，主流的类型系统并不能
很好地支持对于 操作仅在资源处于适当状态时才被执行 这类性质的静态检查。在数据存储的
例子当中，比如，检查 用户在执行 ``读取数据`` 前成功登录了 这一性质非常重要。
``ST`` 库允许我们在类型系统中表达这类 *协议* ，并且在 *编译阶段* 确保机密仅当用户
处于已登录状态时才能被读取。

Outline
=======

大纲
====

This tutorial starts (:ref:`introst`) by describing how to manipulate
individual states, introduce a data type ``STrans`` for describing stateful
functions, and ``ST`` which describes top level state transitions.
Next (:ref:`smstypes`) it describes how to represent state machines in
types, and how to define *interfaces* for describing stateful systems.
Then (:ref:`composing`) it describes how to compose systems of multiple
state machines. It explains how to implement systems which use several
state machines at once, and how to implement a high level stateful system
in terms of lower level systems.
Finally (:ref:`netexample`) we'll see a specific example of a stateful
API in practice, implementing the POSIX network sockets API.

这个教程以描述如何管理多个状态开始 (:ref:`introst`) ，引入了一个叫做 ``STrans``
的数据类型以描述有状态的函数，以及 ``ST`` 用于描述顶层的状态转移。
接下来的章节 (:ref:`smstypes`) 描述了如何用类型表示状态机，以及如何定义 *接口*
以描述有状态的系统。然后 (:ref:`composing`) 它描述了如何组合有多个状态机的系统。
它解释了如何实现同时使用多个状态机的系统，以及如何基于一个低级的系统实现一个高级
的有状态系统。
最后 (:ref:`netexample`) 我们将看到一个有状态 API 应用于真实场景的例子，它实现了
POSIX 网络套接字 API 。

The ``Control.ST`` library is also described in a draft paper by
`Edwin Brady <https://edwinb.wordpress.com/>`_, "State Machines All The Way
Down", available `here <https://www.idris-lang.org/drafts/sms.pdf>`_.
This paper presents many of the examples from this tutorial, and describes
the motivation, design and implementation of the library in more depth. 

本 ``Control.ST`` 库在 `Edwin Brady <https://edwinb.wordpress.com/>`_ 的一篇
文章草稿 "State Machines All The Way Down" 中亦有提及，你可以从 `这里 <https://www.idris-lang.org/drafts/sms.pdf>`_
获取到它。
这篇文章展示了本教程中的很多例子，并且更加深入地描述了它的动机，设计，以及实现。

.. |login| image:: ../image/login.png
                   :width: 500px


