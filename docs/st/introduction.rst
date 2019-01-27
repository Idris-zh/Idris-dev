.. _stoverview:

****
概览
****

.. ********
.. Overview
.. ********

.. Pure functional languages with dependent types such as `Idris
.. <http://www.idris-lang.org/>`_ support reasoning about programs directly
.. in the type system, promising that we can *know* a program will run
.. correctly (i.e. according to the specification in its type) simply
.. because it compiles.

像 `Idris <http://www.idris-lang.org/>`_ 这种可以直接用类型系统对程序进行推理的函数式语言，
仅凭程序能够编译这一点，我们就能\ **确定**\ 该程序会正确运行（即按照类型描述的规范运行）。

.. Realistically, though,  software relies on state, and many components rely on state machines. For
.. example, they describe network transport protocols like TCP, and
.. implement event-driven systems and regular expression matching. Furthermore,
.. many fundamental resources like network sockets and files are, implicitly,
.. managed by state machines, in that certain operations are only valid on
.. resources in certain states, and those operations can change the states of the
.. underlying resource. For example, it only makes sense to send a message on a
.. connected network socket, and closing a socket changes its state from "open" to
.. "closed". State machines can also encode important security properties. For
.. example, in the software which implements an ATM, it’s important that the ATM
.. dispenses cash only when the machine is in a state where a card has been
.. inserted and the PIN verified.

然而在现实中，软件依赖于状态，很多组件则依赖于状态机。它们可以描述像 TCP 这样的网络传输协议，
实现事件驱动的系统以及正则表达式的匹配。此外，像网络 Socket 和文件这类的很多基础资源，
都由状态机隐式地管理；特定操作只有在资源处于特定状态时才可行，而这些操作也可以改变底层资源的状态。
例如，只有向已建立连接的网络 Socket 发送消息才有意义，关闭 Socket 会将其状态从“打开”切换为“关闭”。
状态机同样可以编码重要的安全性质。比如，在一个 ATM（自动取款机）的软件实现中，
有一点性质非常重要：ATM 只有在卡片插入且通过密码验证的状态下才能吐出钞票。

.. In this tutorial we will introduce the ``Control.ST`` library, which is included
.. with the Idris distribution (currently as part of the ``contrib`` package)
.. and supports programming and reasoning with state and side effects.  This
.. tutorial assumes familiarity with pure programming in Idris, as described in
.. :ref:`tutorial-index`.
.. For further background information, the ``ST`` library is based on ideas
.. discussed in Chapter 13 (available as a free sample chapter) and Chapter 14
.. of `Type-Driven Development with Idris <https://www.manning.com/books/type-driven-development-with-idris>`_.

本教程将介绍 ``Control.ST`` 库，它支持对带有状态和副作用的程序进行编程和推理。
该库已包含在 Idris 发行版当中（目前属于 ``contrib`` 包）。本教程假设读者已经熟悉
:ref:`tutorial-index` 中所述的纯函数式的编程方法。\
``ST`` 库基于\ `《Idris 类型驱动开发》 <https://www.manning.com/books/type-driven-development-with-idris>`_\
一书中第 13 和 14 章所讨论的内容，如需更多背景信息可以参考此书。

.. The ``ST`` library allows us to write programs which are composed of multiple
.. state transition systems. It supports composition in two ways: firstly, we can
.. use several independently implemented state transition systems at once;
.. secondly, we can implement one state transition system in terms of others.

我们可使用 ``ST`` 库编写由多个状态转移系统组合而成的程序。它支持两种组合方式：\
第一，我们能够同时使用数个独立实现的状态转移系统；\
第二，我们能够基于其它状态转移系统实现新的状态转移系统。


示例：需要登录的数据存储
========================

.. Introductory example: a data store requiring a login
.. ====================================================

.. Many software components rely on some form of state, and there may be
.. operations which are only valid in specific states. For example, consider
.. a secure data store in which a user must log in before getting access to
.. some secret data. This system can be in one of two states:

许多软件的组件依赖于状态，有些操作仅在特定状态下才有效。
例如，考虑一个安全的数据存储，它只有在用户登录的情况下才能访问某些机密数据。
该系统可以处于以下两种状态之一:

.. * ``LoggedIn``, in which the user is allowed to read the secret
.. * ``LoggedOut``, in which the user has no access to the secret

* ``已登录``，在此状态下，用户可以访问机密数据
* ``未登录``，在此状态下，用户无法访问机密数据

.. We can provide commands to log in, log out, and read the data, as illustrated
.. in the following diagram:

我们可以提供登录、登出和读取数据命令，如下图所示：

|login|

.. The ``login`` command, if it succeeds, moves the overall system state from
.. ``LoggedOut`` to ``LoggedIn``. The ``logout`` command moves the state from
.. ``LoggedIn`` to ``LoggedOut``. Most importantly, the ``readSecret`` command
.. is only valid when the system is in the ``LoggedIn`` state.

如果 ``登录`` 命令被成功执行，就会使系统状态从 ``未登录`` 转移到 ``已登录``
状态。``登出`` 命令会使系统状态从 ``已登录`` 转移到 ``未登录`` 状态。最重要的是，\
``读取数据`` 命令仅在系统处于 ``已登录`` 状态时才有效。

.. We routinely use type checkers to ensure that variables and arguments are used
.. consistently. However, statically checking that operations are performed only
.. on resources in an appropriate state is not well supported by mainstream type
.. systems. In the data store example, for example, it's important to check that
.. the user is successfully logged in before using ``readSecret``. The
.. ``ST`` library allows us to represent this kind of *protocol* in the type
.. system, and ensure at *compile-time* that the secret is only read when the
.. user is logged in.

我们通常使用类型检查器来确保变量和参数的使用一致性。然而，主流的类型系统并不能\
很好地支持像“某些操作只有在资源处于合适的状态时才被执行”这类性质的静态检查。
例如，在数据存储的示例中，检查“用户在执行 ``读取数据`` 前是否成功登录”这一性质非常重要。
``ST`` 库能让我们在类型系统中表达这类\ **协议**\ ，并且在\ **编译阶段**\
确保机密数据只有在用户处于已登录状态时才能被读取。

大纲
====

.. Outline
.. =======

.. This tutorial starts (:ref:`introst`) by describing how to manipulate
.. individual states, introduces a data type ``STrans`` for describing stateful
.. functions, and ``ST`` which describes top level state transitions.
.. Next (:ref:`smstypes`) it describes how to represent state machines in
.. types, and how to define *interfaces* for describing stateful systems.
.. Then (:ref:`composing`) it describes how to compose systems of multiple
.. state machines. It explains how to implement systems which use several
.. state machines at once, and how to implement a high level stateful system
.. in terms of lower level systems.
.. Finally (:ref:`netexample`) we'll see a specific example of a stateful
.. API in practice, implementing the POSIX network sockets API.

本教程从描述如何操作独立的状态开始（:ref:`introst`），引入了 ``STrans``
数据类型来描述带有状态的函数，以及 ``ST`` 用来描述顶层的状态转移。
接下来的章节（:ref:`smstypes`）描述了如何用类型表示状态机，以及如何定义\ **接口**\
以描述带有状态的系统。之后（:ref:`composing`）描述了如何组合带有多个状态机的系统。
它解释了如何实现同时使用多个状态机的系统，以及如何基于低级系统来实现高级的带有状态的系统。
最后（:ref:`netexample`）我们将看到一个带有状态的 API 应用于真实场景的例子，它实现了
POSIX 网络 Socket API。

.. The ``Control.ST`` library is also described in a draft paper by
.. `Edwin Brady <https://edwinb.wordpress.com/>`_, "State Machines All The Way
.. Down", available `here <https://www.idris-lang.org/drafts/sms.pdf>`_.
.. This paper presents many of the examples from this tutorial, and describes
.. the motivation, design and implementation of the library in more depth.

``Control.ST`` 库在 `Edwin Brady <https://edwinb.wordpress.com/>`_
的一篇文稿 "State Machines All The Way Down" 中亦有提及，你可以从\
`这里 <https://www.idris-lang.org/drafts/sms.pdf>`_\ 获取它。
这篇文章展示了本教程中的很多例子，更加深入地描述了它们的动机、设计、以及实现。

.. |login| image:: ../image/login.png
                   :width: 500px
