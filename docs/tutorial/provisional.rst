.. _sect-provisional:

********
临时定义
********

.. ***********************
.. Provisional Definitions
.. ***********************

.. Sometimes when programming with dependent types, the type required by
.. the type checker and the type of the program we have written will be
.. different (in that they do not have the same normal form), but
.. nevertheless provably equal. For example, recall the ``parity``
.. function:

在依赖类型编程中，有时类型检查器需要的类型会和我们编写的程序的类型不同
（在这里是它们的形式不同），但尽管如此它们在证明上是等价的。例如，回想一下
``parity`` 函数：

.. code-block:: idris

    data Parity : Nat -> Type where
       Even : Parity (n + n)
       Odd  : Parity (S (n + n))

.. We’d like to implement this as follows:

我们想要把它实现为：

.. code-block:: idris

    parity : (n:Nat) -> Parity n
    parity Z     = Even {n=Z}
    parity (S Z) = Odd {n=Z}
    parity (S (S k)) with (parity k)
      parity (S (S (j + j)))     | Even = Even {n=S j}
      parity (S (S (S (j + j)))) | Odd  = Odd {n=S j}

.. This simply states that zero is even, one is odd, and recursively, the
.. parity of ``k+2`` is the same as the parity of ``k``. Explicitly marking
.. the value of ``n`` is even and odd is necessary to help type inference.
.. Unfortunately, the type checker rejects this:

它简单地描述了零为偶数，一为奇数，并递归地描述了 ``k+2`` 的奇偶性与 ``k`` 相同。
显式地标出 ``n`` 是奇数还是偶数对于类型推断来说是必须的。不幸的是，类型检查器却拒绝了它：

.. ::

    .. viewsbroken.idr:12:10:When elaborating right hand side of ViewsBroken.parity:
    .. Type mismatch between
    ..     Parity (plus (S j) (S j))
    .. and
    ..     Parity (S (S (plus j j)))

    .. Specifically:
    ..     Type mismatch between
    ..         plus (S j) (S j)
    ..     and
    ..         S (S (plus j j))

::

    viewsbroken.idr:12:10: 在解析 ViewsBroken.parity 的右侧时：
        Parity (plus (S j) (S j))
    与
        Parity (S (S (plus j j)))
    的类型不匹配

    具体为：
            plus (S j) (S j)
        与
            S (S (plus j j))
        的类型不匹配

.. The type checker is telling us that ``(j+1)+(j+1)`` and ``2+j+j`` do not
.. normalise to the same value. This is because ``plus`` is defined by
.. recursion on its first argument, and in the second value, there is a
.. successor symbol on the second argument, so this will not help with
.. reduction. These values are obviously equal — how can we rewrite the
.. program to fix this problem?

类型检查器告诉我们 ``(j+1)+(j+1)`` 和 ``2+j+j`` 无法规范化为相同的值。这是因为
``plus`` 是在第一个参数上递归定义的，而在第二个值中，有一个后继符号作用在第二个参数上，
因此它无法帮助归约。这些值明显相等 — 不过我们要如何重写程序来修复此问题？

临时定义
========

.. Provisional definitions
.. =======================

.. *Provisional definitions* help with this problem by allowing us to defer
.. the proof details until a later point. There are two main reasons why
.. they are useful.

**临时定义** 通过允许我们将证明细节推迟到后面来帮助解决此问题。它主要有两个作用：

.. -  When *prototyping*, it is useful to be able to test programs before
..    finishing all the details of proofs.

.. -  When *reading* a program, it is often much clearer to defer the proof
..    details so that they do not distract the reader from the underlying
..    algorithm.

-  在 **成型** 时，它可用于在结束证明的所有细节前测试程序。

-  在 **阅读** 程序时，推迟证明的详情通常会让过程更清晰，避免读者从底层算法中分心。

.. Provisional definitions are written in the same way as ordinary
.. definitions, except that they introduce the right hand side with a
.. ``?=`` rather than ``=``. We define ``parity`` as follows:

临时定义的写法和普通定义相同，只是它以 ``?=`` 而非 ``=`` 引入右侧。我们将 ``parity``
定义为：

.. code-block:: idris

    parity : (n:Nat) -> Parity n
    parity Z = Even {n=Z}
    parity (S Z) = Odd {n=Z}
    parity (S (S k)) with (parity k)
      parity (S (S (j + j))) | Even ?= Even {n=S j}
      parity (S (S (S (j + j)))) | Odd ?= Odd {n=S j}

.. When written in this form, instead of reporting a type error, Idris
.. will insert a hole standing for a theorem which will correct the
.. type error. Idris tells us we have two proof obligations, with names
.. generated from the module and function names:

当以这种形式编写时，Idris 不会报告类型错误，而是为定理插入一个坑，以此来修正类型错误。
Idris 会告诉我们有两个证明义务，其名字根据模块和函数名生成：

.. code-block:: idris

    *views> :m
    Global holes:
            [views.parity_lemma_2,views.parity_lemma_1]

.. The first of these has the following type:

其中第一个类型如下：

.. code-block:: idris

    *views> :p views.parity_lemma_1

    ---------------------------------- (views.parity_lemma_1) --------
    {hole0} : (j : Nat) -> (Parity (plus (S j) (S j))) -> Parity (S (S (plus j j)))

    -views.parity_lemma_1>

.. The two arguments are ``j``, the variable in scope from the pattern
.. match, and ``value``, which is the value we gave in the right hand side
.. of the provisional definition. Our goal is to rewrite the type so that
.. we can use this value. We can achieve this using the following theorem
.. from the prelude:

它的两个参数为 ``j``，一个是模式匹配作用域中的变量，另一个是 ``value``，
它是我们在临时定义右侧给出的值。我们的目标是重写类型以便让我们能使用该值。
我们可以使用 prelude 中的以下定理来达到这个目的：

.. code-block:: idris

    plusSuccRightSucc : (left : Nat) -> (right : Nat) ->
      S (left + right) = left + (S right)

.. We need to use ``compute`` again to unfold the definition of ``plus``:

我们需要再使用 ``compute`` 来展开 ``plus`` 的定义：

.. code-block:: idris

    -views.parity_lemma_1> compute


    ---------------------------------- (views.parity_lemma_1) --------
    {hole0} : (j : Nat) -> (Parity (S (plus j (S j)))) -> Parity (S (S (plus j j)))

.. After applying ``intros`` we have:

在应用 ``intros`` 之后，我们有：

.. code-block:: idris

    -views.parity_lemma_1> intros

      j : Nat
      value : Parity (S (plus j (S j)))
    ---------------------------------- (views.parity_lemma_1) --------
    {hole2} : Parity (S (S (plus j j)))

.. Then we apply the ``plusSuccRightSucc`` rewrite rule, symmetrically, to
.. ``j`` and ``j``, giving:

接着，我们对称地对 ``j`` 和 ``j`` 应用 ``plusSuccRightSucc`` 重写规则，它会给出：

.. code-block:: idris

    -views.parity_lemma_1> rewrite sym (plusSuccRightSucc j j)

      j : Nat
      value : Parity (S (plus j (S j)))
    ---------------------------------- (views.parity_lemma_1) --------
    {hole3} : Parity (S (plus j (S j)))

.. ``sym`` is a function, defined in the library, which reverses the order
.. of the rewrite:

``sym`` 是一个在库中定义的函数，它用于反转重写的顺序：

.. code-block:: idris

    sym : l = r -> r = l
    sym Refl = Refl

.. We can complete this proof using the ``trivial`` tactic, which finds
.. ``value`` in the premises. The proof of the second lemma proceeds in
.. exactly the same way.

我们可以用 ``trivial`` 策略来完成此证明，它会在前提中找到 ``value``。
第二个引理的证明以完全相同的方式进行。

.. We can now test the ``natToBin`` function from Section :ref:`sect-nattobin`
.. at the prompt. The number 42 is 101010 in binary. The binary digits are
.. reversed:

现在我们可以在提示符中测试 :ref:`sect-nattobin` 一节中的 ``natToBin`` 了。数字
42 的二进制为 101010。其二进制数字以逆序表示：

.. code-block:: idris

    *views> show (natToBin 42)
    "[False, True, False, True, False, True]" : String

暂且相信
========

.. Suspension of Disbelief
.. =======================

.. Idris requires that proofs be complete before compiling programs
.. (although evaluation at the prompt is possible without proof details).
.. Sometimes, especially when prototyping, it is easier not to have to do
.. this. It might even be beneficial to test programs before attempting to
.. prove things about them — if testing finds an error, you know you had
.. better not waste your time proving something!

Idris 在编译程序前需要完成证明（尽管在提示符中求值可以无需详细证明）。然而有时候，
特别在成型时，不去完成证明反而更容易。在尝试证明它们前测试程序甚至可能会更好，
如果测试找到了一个错误，你就会知道最好不要花时间去证明某些事了！

.. Therefore, Idris provides a built-in coercion function, which allows
.. you to use a value of the incorrect types:

因此，Idris 提供了一个内建的强迫（coercion）函数，它允许我们使用类型错误的值：

.. code-block:: idris

    believe_me : a -> b

.. Obviously, this should be used with extreme caution. It is useful when
.. prototyping, and can also be appropriate when asserting properties of
.. external code (perhaps in an external C library). The “proof” of
.. ``views.parity_lemma_1`` using this is:

很明显，它必须非常小心地使用。在确定原型时它非常有用，在断言外部代码（可能在外部的
C 库中）的属性时也是可以用的。使用了它的 ``views.parity_lemma_1`` 「证明」为：

.. code-block:: idris

    views.parity_lemma_2 = proof {
        intro;
        intro;
        exact believe_me value;
    }

.. The ``exact`` tactic allows us to provide an exact value for the proof.
.. In this case, we assert that the value we gave was correct.

``exact`` 策略允许我们为该证明提供一个确切的值。在本例中，我们断言给出的值是正确的。

示例：二进制数
==============

.. Example: Binary numbers
.. =======================

.. Previously, we implemented conversion to binary numbers using the
.. ``Parity`` view. Here, we show how to use the same view to implement a
.. verified conversion to binary. We begin by indexing binary numbers over
.. their ``Nat`` equivalent. This is a common pattern, linking a
.. representation (in this case ``Binary``) with a meaning (in this case
.. ``Nat``):

我们在前面通过 ``Parity`` 视角实现了到二进制数的转换。在这里，
我们会展示如何用同样的视角来实现验证过的到二进制的转换。我们先从与其等价的 ``Nat``
上索引二进制数开始。这是一种通用的模式，将一种表示（这里为 ``Binary``）与其含义
（这里为 ``Nat``）关联起来：

.. code-block:: idris

    data Binary : Nat -> Type where
       BEnd : Binary Z
       BO : Binary n -> Binary (n + n)
       BI : Binary n -> Binary (S (n + n))

.. ``BO`` and ``BI`` take a binary number as an argument and effectively
.. shift it one bit left, adding either a zero or one as the new least
.. significant bit. The index, ``n + n`` or ``S (n + n)`` states the result
.. that this left shift then add will have to the meaning of the number.
.. This will result in a representation with the least significant bit at
.. the front.

``BO`` 和 ``BI`` 接受一个二进制数作为其参数并立即将它左移一位，
然后再加零或一作为新的最低位。索引 ``n + n`` 或 ``S (n + n)``
描述了左移后再相加的结果与该数值的意义相同。它会产生低位在前的表示。

.. Now a function which converts a Nat to binary will state, in the type,
.. that the resulting binary number is a faithful representation of the
.. original Nat:

现在，将 Nat 转换为二进制的函数在类型中描述了其结果二进制数为原始 Nat 的忠实表示：

.. code-block:: idris

    natToBin : (n:Nat) -> Binary n

.. The ``Parity`` view makes the definition fairly simple — halving the
.. number is effectively a right shift after all — although we need to use
.. a provisional definition in the Odd case:

``Parity`` 视角让定义变得相当简单：把数除以二其实就是进行一次右移，尽管我们需要在
Odd 的情况中使用临时定义：

.. code-block:: idris

    natToBin : (n:Nat) -> Binary n
    natToBin Z = BEnd
    natToBin (S k) with (parity k)
       natToBin (S (j + j))     | Even  = BI (natToBin j)
       natToBin (S (S (j + j))) | Odd  ?= BO (natToBin (S j))

.. The problem with the Odd case is the same as in the definition of
.. ``parity``, and the proof proceeds in the same way:

Odd 情况的问题与 ``parity`` 定义中的相同，其证明过程也一样：

.. code-block:: idris

    natToBin_lemma_1 = proof {
        intro;
        intro;
        rewrite sym (plusSuccRightSucc j j);
        trivial;
    }

.. To finish, we’ll implement a main program which reads an integer from
.. the user and outputs it in binary.

最后，我们来实现一个主程序，它读取用户输入的整数并按照二进制输出：

.. code-block:: idris

    main : IO ()
    main = do putStr "Enter a number: "
              x <- getLine
              print (natToBin (fromInteger (cast x)))

.. For this to work, of course, we need a ``Show`` implementation for
.. ``Binary n``:

当然，为了让它能够工作，我们需要为 ``Binary n`` 实现 ``Show``：

.. code-block:: idris

    Show (Binary n) where
        show (BO x) = show x ++ "0"
        show (BI x) = show x ++ "1"
        show BEnd = ""
