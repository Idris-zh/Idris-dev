.. _sect-theorems:

********
定理证明
********

.. ***************
.. Theorem Proving
.. ***************

相等性
========

.. Equality
.. ========

.. Idris allows propositional equalities to be declared, allowing theorems about
.. programs to be stated and proved. Equality is built in, but conceptually
.. has the following definition:

Idris 可以声明命题的相等性，陈述并证明有关程序的定理。
相等性是内建的，不过这里给出其概念上的定义：

.. code-block:: idris

    data (=) : a -> b -> Type where
       Refl : x = x

.. Equalities can be proposed between any values of any types, but the only
.. way to construct a proof of equality is if values actually are equal.
.. For example:

任何类型的任何值之间都可以判断相等性，然而构造相等性证明的唯一方式就是值确实相等。例如：

.. code-block:: idris

    fiveIsFive : 5 = 5
    fiveIsFive = Refl

    twoPlusTwo : 2 + 2 = 4
    twoPlusTwo = Refl

.. _sect-empty:

空类型
======

.. The Empty Type
.. ==============

.. There is an empty type, :math:`\bot`, which has no constructors. It is
.. therefore impossible to construct an element of the empty type, at least
.. without using a partially defined or general recursive function (see
.. Section :ref:`sect-totality` for more details). We can therefore use the
.. empty type to prove that something is impossible, for example zero is
.. never equal to a successor:

存在一个空类型 :math:`\bot`，它没有构造器。因此，在不使用部分定义或一般递归函数的情况下，
无法构造出空类型的元素（详见 :ref:`sect-totality` 一节）。
由此，我们可以用空类型来证明某些命题是不可能的，例如零不可能是任何自然数的后继：

.. code-block:: idris

    disjoint : (n : Nat) -> Z = S n -> Void
    disjoint n p = replace {P = disjointTy} p ()
      where
        disjointTy : Nat -> Type
        disjointTy Z = ()
        disjointTy (S k) = Void

.. There is no need to worry too much about how this function works —
.. essentially, it applies the library function ``replace``, which uses an
.. equality proof to transform a predicate. Here we use it to transform a
.. value of a type which can exist, the empty tuple, to a value of a type
.. which can’t, by using a proof of something which can’t exist.

我们不必太关心该函数如何工作 — 本质上，它应用库函数 ``replace``，
根据相等证明来变换谓词。在本例中，我们利用某些东西无法存在的证明，
将一个可以存在的类型（即空元组）的值，变换成了一个无法存在的类型的值。

.. Once we have an element of the empty type, we can prove anything.
.. ``void`` is defined in the library, to assist with proofs by
.. contradiction.

一旦拥有了空类型的元素，我们就能证明任何东西。``void`` 在库中定义，用于辅助反证法。

.. code-block:: idris

    void : Void -> a

简单定理
========

.. Simple Theorems
.. ===============

.. When type checking dependent types, the type itself gets *normalised*.
.. So imagine we want to prove the following theorem about the reduction
.. behaviour of ``plus``:

当对依赖类型进行类型检查时，该类型就会被 **规范化（Normalized）**。
比如我们想要证明以下关于 ``plus`` 的归约行为（Reduction Behaviour）的定理：

.. code-block:: idris

    plusReduces : (n:Nat) -> plus Z n = n

.. We’ve written down the statement of the theorem as a type, in just the
.. same way as we would write the type of a program. In fact there is no
.. real distinction between proofs and programs. A proof, as far as we are
.. concerned here, is merely a program with a precise enough type to
.. guarantee a particular property of interest.

我们将该定理的陈述写成了类型，就像写出程序的类型一样。实际上证明和程序之间并没有本质的区别。
就我们所关注的而言，证明不过是个程序，只是它的类型精确到足以保证满足我们关心的特殊性质。

.. We won’t go into details here, but the Curry-Howard correspondence [1]_
.. explains this relationship. The proof itself is trivial, because
.. ``plus Z n`` normalises to ``n`` by the definition of ``plus``:

我们不会在这里深入细节，Curry-Howard 同构 [1]_ 解释了这种关系。该证明很平凡，
因为 ``plus`` 的定义将 ``plus Z n`` 规范化成了 ``n``：

.. code-block:: idris

    plusReduces n = Refl

.. It is slightly harder if we try the arguments the other way, because
.. plus is defined by recursion on its first argument. The proof also works
.. by recursion on the first argument to ``plus``, namely ``n``.

如果我们换种方式证明该论点，那就有点难了，因为加法是对第一个参数递归定义的。
该证明同样也可以在 ``plus`` 的第一个参数 ``n`` 上递归：

.. code-block:: idris

    plusReducesZ : (n:Nat) -> n = plus n Z
    plusReducesZ Z = Refl
    plusReducesZ (S k) = cong (plusReducesZ k)

.. ``cong`` is a function defined in the library which states that equality
.. respects function application:

``cong`` 是库中定义的一个函数，它指明相等性也适用于函数应用：

.. code-block:: idris

    cong : {f : t -> u} -> a = b -> f a = f b

.. We can do the same for the reduction behaviour of plus on successors:

我们可以对加法在后继上的递归行为做同样的事情：

.. code-block:: idris

    plusReducesS : (n:Nat) -> (m:Nat) -> S (plus n m) = plus n (S m)
    plusReducesS Z m = Refl
    plusReducesS (S k) m = cong (plusReducesS k m)

.. Even for trivial theorems like these, the proofs are a little tricky to
.. construct in one go. When things get even slightly more complicated, it
.. becomes too much to think about to construct proofs in this “batch
.. mode”.

即便对于如此平凡的定理，一口气构造出证明也有点棘手。当事情变得更复杂时，
需要考虑的就太多了，此时根本无法在这种「批量模式」下构造证明。

.. Idris provides interactive editing capabilities, which can help with
.. building proofs. For more details on building proofs interactively in
.. an editor, see :ref:`proofs-index`.

Idris 提供了交互式编辑的能力，它可以帮助构造证明。关于在编辑器中交互式构造证明的更多详情，
参见 :ref:`proofs-index`。

.. _sect-parity:

实践中的证明
============

.. Theorems in Practice
.. ====================

.. The need to prove theorems can arise naturally in practice. For example,
.. previously (:ref:`sec-views`) we implemented ``natToBin`` using a function
.. ``parity``:

证明定理的需求可在实践中自然产生。例如，在之前的 :ref:`sec-views` 一节中，
我们用函数 ``parity`` 实现了 ``natToBin``：

.. code-block:: idris

    parity : (n:Nat) -> Parity n

.. However, we didn't provide a definition for ``parity``. We might expect it
.. to look something like the following:

然而，我们并未提供 ``parity`` 的定义。我们可能觉得它看起来会像下面这样：

.. code-block:: idris

    parity : (n:Nat) -> Parity n
    parity Z     = Even {n=Z}
    parity (S Z) = Odd {n=Z}
    parity (S (S k)) with (parity k)
      parity (S (S (j + j)))     | Even = Even {n=S j}
      parity (S (S (S (j + j)))) | Odd  = Odd {n=S j}

.. Unfortunately, this fails with a type error:

然而，它会因类型错误而无法编译：

.. ::

..     When checking right hand side of with block in views.parity with expected type
..             Parity (S (S (j + j)))

..     Type mismatch between
..             Parity (S j + S j) (Type of Even)
..     and
..             Parity (S (S (plus j j))) (Expected type)

::

    在按照期望的类型
            Parity (S (S (j + j)))
    检查 views.parity 中 with 块的右侧时

    发现
            Parity (S j + S j) （Even 的类型）
    与
            Parity (S (S (plus j j))) （期望的类型）
    的类型不匹配

.. The problem is that normalising ``S j + S j``, in the type of ``Even``
.. doesn't result in what we need for the type of the right hand side of
.. ``Parity``. We know that ``S (S (plus j j))`` is going to be equal to
.. ``S j + S j``, but we need to explain it to Idris with a proof. We can
.. begin by adding some *holes* (see :ref:`sect-holes`) to the definition:

问题在于，在 ``Even`` 的类型中规范化 ``S j + S j`` 并不能得到我们需要的
``Parity`` 右侧的类型。我们知道 ``S (S (plus j j))`` 等于 ``S j + S j``，
但需要向 Idris 证明它。我们可以先为该定义挖一些 **坑** （见 :ref:`sect-holes`）：

.. code-block:: idris

    parity : (n:Nat) -> Parity n
    parity Z     = Even {n=Z}
    parity (S Z) = Odd {n=Z}
    parity (S (S k)) with (parity k)
      parity (S (S (j + j)))     | Even = let result = Even {n=S j} in
                                              ?helpEven
      parity (S (S (S (j + j)))) | Odd  = let result = Odd {n=S j} in
                                              ?helpOdd

.. Checking the type of ``helpEven`` shows us what we need to prove for the
.. ``Even`` case:

检查 ``helpEven`` 的类型会告诉我们需要为 ``Even`` 的情况证明什么：

::

      j : Nat
      result : Parity (S (plus j (S j)))
    --------------------------------------
    helpEven : Parity (S (S (plus j j)))

.. We can therefore write a helper function to *rewrite* the type to the form
.. we need:

由此我们可以编写一个辅助函数，将它的类型 **重写** 为需要的形式：

.. code-block:: idris

    helpEven : (j : Nat) -> Parity (S j + S j) -> Parity (S (S (plus j j)))
    helpEven j p = rewrite plusSuccRightSucc j j in p

.. The ``rewrite ... in`` syntax allows you to change the required type of an
.. expression by rewriting it according to an equality proof. Here, we have
.. used ``plusSuccRightSucc``, which has the following type:

``rewrite ... in`` 语法允许你根据相等性证明来改写它，以此改变表达式需要的类型。
在这里，我们使用了 ``plusSuccRightSucc``，其类型如下：

.. code-block:: idris

    plusSuccRightSucc : (left : Nat) -> (right : Nat) -> S (left + right) = left + S right

.. We can see the effect of ``rewrite`` by replacing the right hand side of
.. ``helpEven`` with a hole, and working step by step. Beginning with the following:

我们可以在 ``helpEven`` 的右侧挖个坑来看到 ``rewrite`` 的效果，然后一步一步地做。
我们从下面开始：

.. code-block:: idris

    helpEven : (j : Nat) -> Parity (S j + S j) -> Parity (S (S (plus j j)))
    helpEven j p = ?helpEven_rhs

.. We can look at the type of ``helpEven_rhs``:

先查看一下 ``helpEven_rhs`` 的类型：

.. code-block:: idris

      j : Nat
      p : Parity (S (plus j (S j)))
    --------------------------------------
    helpEven_rhs : Parity (S (S (plus j j)))

.. Then we can ``rewrite`` by applying ``plusSuccRightSucc j j``, which gives
.. an equation ``S (j + j) = j + S j``, thus replacing ``S (j + j)`` (or,
.. in this case, ``S (plus j j)`` since ``S (j + j)`` reduces to that) in the
.. type with ``j + S j``:

然后通过应用 ``plusSuccRightSucc j j`` 来进行 ``rewrite`` 重写，
它会给出等式 ``S (j + j) = j + S j``，从而在类型中用 ``j + S j`` 取代
``S (j + j)`` （在这里是 ``S (plus j j)``，它由 ``S (j + j)`` 规约而来 ）：

.. code-block:: idris

    helpEven : (j : Nat) -> Parity (S j + S j) -> Parity (S (S (plus j j)))
    helpEven j p = rewrite plusSuccRightSucc j j in ?helpEven_rhs

.. Checking the type of ``helpEven_rhs`` now shows what has happened, including
.. the type of the equation we just used (as the type of ``_rewrite_rule``):

现在检查 ``helpEven_rhs`` 的类型会告诉我们发生了什么，包括刚才所用的等式的类型
（即 ``_rewrite_rule`` 的类型）：

.. code-block:: idris

      j : Nat
      p : Parity (S (plus j (S j)))
      _rewrite_rule : S (plus j j) = plus j (S j)
    --------------------------------------
    helpEven_rhs : Parity (S (plus j (S j)))

.. Using ``rewrite`` and another helper for the ``Odd`` case, we can complete
.. ``parity`` as follows:

对 ``Odd`` 的情况使用 ``rewrite`` 和另一个辅助函数，我们可以完成 ``parity``：

.. code-block:: idris

    helpEven : (j : Nat) -> Parity (S j + S j) -> Parity (S (S (plus j j)))
    helpEven j p = rewrite plusSuccRightSucc j j in p

    helpOdd : (j : Nat) -> Parity (S (S (j + S j))) -> Parity (S (S (S (j + j))))
    helpOdd j p = rewrite plusSuccRightSucc j j in p

    parity : (n:Nat) -> Parity n
    parity Z     = Even {n=Z}
    parity (S Z) = Odd {n=Z}
    parity (S (S k)) with (parity k)
      parity (S (S (j + j)))     | Even = helpEven j (Even {n = S j})
      parity (S (S (S (j + j)))) | Odd  = helpOdd j (Odd {n = S j})

.. Full details of ``rewrite`` are beyond the scope of this introductory tutorial,
.. but it is covered in the theorem proving tutorial (see :ref:`proofs-index`).

``rewrite`` 的完整细节超出了本入门教程的范围，不过定理证明教程
（见 :ref:`proofs-index`）中覆盖了它。

.. _sect-totality:

完全性检查
==========

.. Totality Checking
.. =================

.. If we really want to trust our proofs, it is important that they are
.. defined by *total* functions — that is, a function which is defined for
.. all possible inputs and is guaranteed to terminate. Otherwise we could
.. construct an element of the empty type, from which we could prove
.. anything:

如果我们真的想要信任我们的证明，它们定义为 **全** 函数是十分重要的 — 也就是说，
一个函数为所有可能的输入情况定义，并且保证会终止。不然我们就能构造出一个空类型的元素，
以它开始我们可以证明任何东西：

.. .. code-block:: idris

..     -- making use of 'hd' being partially defined
..     empty1 : Void
..     empty1 = hd [] where
..         hd : List a -> a
..         hd (x :: xs) = x

..     -- not terminating
..     empty2 : Void
..     empty2 = empty2

.. code-block:: idris

    -- 利用部分定义的「hd」
    empty1 : Void
    empty1 = hd [] where
        hd : List a -> a
        hd (x :: xs) = x

    -- 不会终止
    empty2 : Void
    empty2 = empty2

.. Internally, Idris checks every definition for totality, and we can check at
.. the prompt with the ``:total`` command. We see that neither of the above
.. definitions is total:

Idris 会在内部检查所有函数的完全性，我们可在提示符中用 ``:total`` 命令来检查。
我们会看到上面的两个定义都是不完全的：

.. ::

..     *Theorems> :total empty1
..     possibly not total due to: empty1#hd
..         not total as there are missing cases
..     *Theorems> :total empty2
..     possibly not total due to recursive path empty2

::

    *Theorems> :total empty1
    可能不完全，由于： empty1#hd
        不完全，因为有遗漏的情况
    *Theorems> :total empty2
    可能不完全，由于递归路径 empty2

.. Note the use of the word “possibly” — a totality check can, of course,
.. never be certain due to the undecidability of the halting problem. The
.. check is, therefore, conservative. It is also possible (and indeed
.. advisable, in the case of proofs) to mark functions as total so that it
.. will be a compile time error for the totality check to fail:

注意这里用了「可能」一词 — 由于停机问题的不可判定性，完全性检查当然永远无法确定。
因此，该检查是保守的。我们也可以将函数标记为完全的，使其在完全性检查失败时产生编译期错误：

.. code-block:: idris

    total empty2 : Void
    empty2 = empty2

.. ::

..     Type checking ./theorems.idr
..     theorems.idr:25:empty2 is possibly not total due to recursive path empty2

::

    对 ./theorems.idr 进行类型检查
    theorems.idr:25:empty2 可能不完全，由于递归路径 empty2

.. Reassuringly, our proof in Section :ref:`sect-empty` that the zero and
.. successor constructors are disjoint is total:

令人欣慰的是，我们在 :ref:`sect-empty` 一节中对零与后继构造器分立的证明是完全的：

.. code-block:: idris

    *theorems> :total disjoint
    Total

.. The totality check is, necessarily, conservative. To be recorded as
.. total, a function ``f`` must:

.. -  Cover all possible inputs

.. -  Be *well-founded* — i.e. by the time a sequence of (possibly
..    mutually) recursive calls reaches ``f`` again, it must be possible to
..    show that one of its arguments has decreased.

.. -  Not use any data types which are not *strictly positive*

.. -  Not call any non-total functions

完全性检查必然是保守的。要被记录为完全的，函数 ``f`` 必须：

-  覆盖所有可能的输入

-  是 **良基** 的 — 即，当一系列（可能互相）递归的调用再次到达 ``f``
   时，它必须能够表明其参数之一已经递减。

-  使用的数据类型必须 **严格为正（strictly positive）**

-  没有调用任何不完全的函数


完全性的指令与编译器参数
------------------------

.. Directives and Compiler Flags for Totality
.. ------------------------------------------

.. By default, Idris allows all well-typed definitions, whether total or not.
.. However, it is desirable for functions to be total as far as possible, as this
.. provides a guarantee that they provide a result for all possible inputs, in
.. finite time. It is possible to make total functions a requirement, either:

默认情况下，Idris 允许所有良类型的定义，无论是否完全。然而在理想情况下，
函数总是要尽可能地完全，因为这能保证它们可以在有限时间内，
对于所有可能的输入提供一个结果。我们可以要求函数是完全的，通过以下两种方式之一：

.. -  By using the ``--total`` compiler flag.

.. -  By adding a ``%default total`` directive to a source file. All
..    definitions after this will be required to be total, unless
..    explicitly flagged as ``partial``.

-  使用 ``--total`` 编译器参数。

-  为源文件添加 ``%default total`` 指令。在这之后的所有定义都会要求为完全的，
   除非显式地标记为 ``partial``。

.. All functions *after* a ``%default total`` declaration are required to
.. be total. Correspondingly, after a ``%default partial`` declaration, the
.. requirement is relaxed.

在 ``%default total`` 声明 **之后** 的所有函数都会被要求是完全的。与此相应，
``%default partial`` 声明之后的要求则被放宽。

.. Finally, the compiler flag ``--warnpartial`` causes to print a warning
.. for any undeclared partial function.

最后，编译器参数 ``--warnpartial`` 会为任何未声明完全性的偏函数打印一个警告。

完全性检查的问题
----------------

.. Totality checking issues
.. ------------------------

.. Please note that the totality checker is not perfect! Firstly, it is
.. necessarily conservative due to the undecidability of the halting
.. problem, so many programs which *are* total will not be detected as
.. such. Secondly, the current implementation has had limited effort put
.. into it so far, so there may still be cases where it believes a function
.. is total which is not. Do not rely on it for your proofs yet!

请注意，完全性检查器并不完美！首先，由于停机问题的不可判定性，它必然是保守的，
因此一些 **确实完全** 的程序不会被检测为完全的。其次，当前实现投入的精力有限，
因此它仍然有可能将不完全的函数当作完全的。你的证明请先不要依赖它！

完全性的提示
------------

.. Hints for totality
.. ------------------

.. In cases where you believe a program is total, but Idris does not agree, it is
.. possible to give hints to the checker to give more detail for a termination
.. argument. The checker works by ensuring that all chains of recursive calls
.. eventually lead to one of the arguments decreasing towards a base case, but
.. sometimes this is hard to spot. For example, the following definition cannot be
.. checked as ``total`` because the checker cannot decide that ``filter (< x) xs``
.. will always be smaller than ``(x :: xs)``:

有时你确信一个程序是完全的，但 Idris 不这么认为，此时可以对检查器给出提示，
以此来给出终止参数的详情。检查器会确保所有的递归调用链最终都能导致其中一个参数递减到基本情况，
然而有些情况很难辨别。例如，以下定义无法被检查为 ``total``，因为检查器无法确定
``filter (< x) xs`` 一定小于 ``(x :: xs)``：

.. code-block:: idris

    qsort : Ord a => List a -> List a
    qsort [] = []
    qsort (x :: xs)
       = qsort (filter (< x) xs) ++
          (x :: qsort (filter (>= x) xs))

.. The function ``assert_smaller``, defined in the prelude, is intended to
.. address this problem:

prelude 中定义的 ``assert_smaller`` 旨在解决这个问题：

.. code-block:: idris

    assert_smaller : a -> a -> a
    assert_smaller x y = y

.. It simply evaluates to its second argument, but also asserts to the
.. totality checker that ``y`` is structurally smaller than ``x``. This can
.. be used to explain the reasoning for totality if the checker cannot work
.. it out itself. The above example can now be written as:

它简单地求值成第二个参数，但也会向完全性检查器断言 ``y`` 在结构上小于 ``x``。
当检查器自己无法解决时，它可用于解释完全性的推理。现在上面的例子可重写为：

.. code-block:: idris

    total
    qsort : Ord a => List a -> List a
    qsort [] = []
    qsort (x :: xs)
       = qsort (assert_smaller (x :: xs) (filter (< x) xs)) ++
          (x :: qsort (assert_smaller (x :: xs) (filter (>= x) xs)))

.. The expression ``assert_smaller (x :: xs) (filter (<= x) xs)`` asserts
.. that the result of the filter will always be smaller than the pattern
.. ``(x :: xs)``.

表达式 ``assert_smaller (x :: xs) (filter (<= x) xs)`` 断言 ``filter``
的结果总是小于模式 ``(x :: xs)`` 。

.. In more extreme cases, the function ``assert_total`` marks a
.. subexpression as always being total:

在更极端的情况下，函数 ``assert_total`` 能将一个表达式标为总是完全的：

.. code-block:: idris

    assert_total : a -> a
    assert_total x = x

.. In general, this function should be avoided, but it can be very useful
.. when reasoning about primitives or externally defined functions (for
.. example from a C library) where totality can be shown by an external
.. argument.

通常，你应当避免使用该函数，不过在对原语进行推理，或者在对外部定义的，
完全性可被外部参数展示的函数（例如 C 库中的）进行推理时，它会非常有用。


.. [1] Timothy G. Griffin. 1989. A formulae-as-type notion of
       control. In Proceedings of the 17th ACM SIGPLAN-SIGACT
       symposium on Principles of programming languages (POPL
       '90). ACM, New York, NY, USA, 47-58. DOI=10.1145/96709.96714
       http://doi.acm.org/10.1145/96709.96714
