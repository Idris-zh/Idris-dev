.. _sec-views:

*****************************
视角与「``with``」规则
*****************************

.. *****************************
.. Views and the “``with``” rule
.. *****************************

依赖模式匹配
============

.. Dependent pattern matching
.. ==========================

.. Since types can depend on values, the form of some arguments can be
.. determined by the value of others. For example, if we were to write
.. down the implicit length arguments to ``(++)``, we’d see that the form
.. of the length argument was determined by whether the vector was empty
.. or not:

由于类型可以依赖于值，因此某些参数的形式可根据其它参数的值来确定。例如，
如果我们写出 ``(++)`` 的长度隐式参数，就会看出它的形式由向量是否为空来决定：

.. code-block:: idris

    (++) : Vect n a -> Vect m a -> Vect (n + m) a
    (++) {n=Z}   []        ys = ys
    (++) {n=S k} (x :: xs) ys = x :: xs ++ ys

.. If ``n`` was a successor in the ``[]`` case, or zero in the ``::``
.. case, the definition would not be well typed.

假如 ``n`` 在 ``[]`` 的情况下是一个后继，或者在 ``::`` 的情况下为零，
那么它的定义就不是良类型的。

.. _sect-nattobin:

``with`` 规则：匹配中间值
=========================

.. The ``with`` rule — matching intermediate values
.. ================================================

.. Very often, we need to match on the result of an intermediate
.. computation. Idris provides a construct for this, the ``with``
.. rule, inspired by views in ``Epigram`` [1]_, which takes account of
.. the fact that matching on a value in a dependently typed language can
.. affect what we know about the forms of other values. In its simplest
.. form, the ``with`` rule adds another argument to the function being
.. defined.

我们经常需要匹配中间计算（Intermediate Computation）的结果。Idris 受到
``Epigram`` [1]_ 中视角（View）的启发，为此提供了一种构造，即 ``with`` 规则，
它考虑到在依赖型的语言中匹配值会影响我们对其它值的形式的了解。在最简单的形式中，
``with`` 规则会为正在定义的函数附加一个额外的参数。

.. We have already seen a vector filter function. This time, we define it
.. using ``with`` as follows:

我们已经见过向量过滤函数了。这次我们用 ``with`` 来定义它：

.. code-block:: idris

    filter : (a -> Bool) -> Vect n a -> (p ** Vect p a)
    filter p [] = ( _ ** [] )
    filter p (x :: xs) with (filter p xs)
      filter p (x :: xs) | ( _ ** xs' ) = if (p x) then ( _ ** x :: xs' ) else ( _ ** xs' )

.. Here, the ``with`` clause allows us to deconstruct the result of
.. ``filter p xs``. The view refined argument pattern ``filter p (x ::
.. xs)`` goes beneath the ``with`` clause, followed by a vertical bar
.. ``|``, followed by the deconstructed intermediate result ``( _ ** xs'
.. )``. If the view refined argument pattern is unchanged from the
.. original function argument pattern, then the left side of ``|`` is
.. extraneous and may be omitted:

在这里，``with`` 从句能让我们解构（deconstruct）``filter p xs`` 的结果。
该视角精化的参数模式 ``filter p (x :: xs)`` 位于 ``with`` 从句的下方，
之后是一条竖线 ``|`` 后面跟着解构的中间结果 ``( _ ** xs' )``。
如果该视角精化的参数模式与原函数的参数模式相同，那么 ``|`` 左侧的就是多余的，可以省略：

.. code-block:: idris

    filter p (x :: xs) with (filter p xs)
      | ( _ ** xs' ) = if (p x) then ( _ ** x :: xs' ) else ( _ ** xs' )

.. hint::

    这个例子并不好，它省去了大量的细节，包括什么是视角，如何用视角来解构数据，
    以及 with 从句帮你做了什么。参数 ``p`` 与结果 ``(p ** Vect p a)``
    中的 ``p`` 也毫无关系，这点可能会对初学者造成困扰。有条件的读者可参考
    Type-Driven Development with Idris 第二部分中的
    Views: extending pattern matching 一章，或者重新思考接下来这个例子。

.. ``with`` clauses can also be nested:

``with`` 从句还可以嵌套：

.. code-block:: idris

    foo : Int -> Int -> Bool
    foo n m with (succ n)
      foo _ m | 2 with (succ m)
        foo _ _ | 2 | 3 = True
        foo _ _ | 2 | _ = False
      foo _ _ | _ = False

.. If the intermediate computation itself has a dependent type, then the
.. result can affect the forms of other arguments — we can learn the form
.. of one value by testing another. In these cases, view refined argument
.. patterns must be explicit. For example, a ``Nat`` is either even or
.. odd. If it is even it will be the sum of two equal ``Nat``.
.. Otherwise, it is the sum of two equal ``Nat`` plus one:

如果中间计算本身具有依赖类型，那么其结果会影响其它参数的形式 — 我们可以通过测试
附加参数来了解一个值的形式。在这类情况下，视角精化的参数模式必须是显式的。
例如，一个 ``Nat`` 不是偶数就是奇数。如果它是偶数，那么它就等于两个 ``Nat`` 之和。
否则，它就是两个 ``Nat`` 之和再加一：

.. code-block:: idris

    data Parity : Nat -> Type where
       Even : Parity (n + n)
       Odd  : Parity (S (n + n))

.. We say ``Parity`` is a *view* of ``Nat``. It has a *covering function*
.. which tests whether it is even or odd and constructs the predicate
.. accordingly.

我们将 ``Parity`` 称为 ``Nat`` 的一个 **视角（View）**。它拥有一个
**覆盖函数（Covering Function）** 来测试 ``Nat`` 的奇偶性并构造相应的谓词：

.. code-block:: idris

    parity : (n:Nat) -> Parity n

.. We’ll come back to the definition of ``parity`` shortly. We can use it
.. to write a function which converts a natural number to a list of
.. binary digits (least significant first) as follows, using the
.. rule:

我们之后再看 ``parity`` 的定义。我们可以用它配合 ``with`` 规则来编写一个函数，
它将一个自然数转换成一系列二进制数字（False 为 0，True 为 1，低位在前）：

.. code-block:: idris

    natToBin : Nat -> List Bool
    natToBin Z = Nil
    natToBin k with (parity k)
       natToBin (j + j)     | Even = False :: natToBin j
       natToBin (S (j + j)) | Odd  = True  :: natToBin j

.. The value of ``parity k`` affects the form of ``k``, because the
.. result of ``parity k`` depends on ``k``. So, as well as the patterns
.. for the result of the intermediate computation (``Even`` and ``Odd``)
.. right of the ``|``, we also write how the results affect the other
.. patterns left of the ``|``. That is:

``parity k`` 的值影响了 ``k`` 的形式，因为 ``parity k`` 的结果取决于 ``k``。
因此，除了 ``|`` 右侧的中间计算结果（``Even`` 和 ``Odd``）的模式外，
我们还可以写出该结果如何影响 ``|`` 左侧的其它模式。即：

.. - When ``parity k`` evaluates to ``Even``, we can refine the original
..   argument ``k`` to a refined pattern ``(j + j)`` according to
..   ``Parity (n + n)`` from the ``Even`` constructor definition. So
..   ``(j + j)`` replaces ``k`` on the left side of ``|``, and the
..   ``Even`` constructor appears on the right side. The natural number
..   ``j`` in the refined pattern can be used on the ride side of the
..   ``=`` sign.

- 当 ``parity k`` 求值为 ``Even`` 时，我们可以根据 ``Even`` 构造器的定义
  ``Parity (n + n)``，将原始参数 ``k`` 精化为模式 ``(j + j)``。这样 ``(j + j)``
  就代替了 ``|`` 左侧的 ``k``，而 ``Even`` 构造器则出现在右侧。精化模式中的自然数
  ``j`` 会被用在 ``=`` 符号的两侧。

.. - Otherwise, when ``parity k`` evaluates to ``Odd``, the original
..   argument ``k`` is refined to ``S (j + j)`` according to ``Parity (S
..   (n + n))`` from the ``Odd`` constructor definition, and ``Odd`` now
..   appears on the ride side of ``|``, again with the natural number
..   ``j`` used on the ride side of the ``=`` sign.

- 否则，当 ``parity k`` 求值为 ``Odd`` 时，根据 ``Odd`` 构造器的定义
  ``Parity (S (n + n))``，原始参数 ``k`` 会被精化为模式 ``S (j + j)``，
  它和 ``Odd`` 会出现在 ``|`` 的两侧，同样自然数 ``j`` 会被用在 ``=`` 符号的两侧。

.. Note that there is a function in the patterns (``+``) and repeated
.. occurrences of ``j`` - this is allowed because another argument has
.. determined the form of these patterns.

注意，在精化模式的两个 ``j`` 之间有一个函数 (``+``)，它被允许是因为附加参数
已经确定了此模式的形式。

.. We will return to this function in the next section :ref:`sect-parity` to
.. complete the definition of ``parity``.

我们会在下一节 :ref:`sect-parity` 中回到 ``parity`` 上来完成它的定义。

with 与证明
===============

.. With and proofs
.. ===============

.. To use a dependent pattern match for theorem proving, it is sometimes necessary
.. to explicitly construct the proof resulting from the pattern match.
.. To do this, you can postfix the with clause with ``proof p`` and the proof
.. generated by the pattern match will be in scope and named ``p``. For example:

要使用依赖模式匹配进行定理证明，有时必须根据匹配模式显式地构造出证明结果。为此，你可以为
with 从句加上 ``proof p`` 后缀，由模式匹配生成的证明会被命名为 ``p`` 并加入到作用域中。
例如：

.. code-block:: idris

    data Foo = FInt Int | FBool Bool

    optional : Foo -> Maybe Int
    optional (FInt x) = Just x
    optional (FBool b) = Nothing

    isFInt : (foo:Foo) -> Maybe (x : Int ** (optional foo = Just x))
    isFInt foo with (optional foo) proof p
      isFInt foo | Nothing = Nothing           -- here, p : Nothing = optional foo
      isFInt foo | (Just x) = Just (x ** Refl) -- here, p : Just x = optional foo


.. [1] Conor McBride and James McKinna. 2004. The view from the
       left. J. Funct. Program. 14, 1 (January 2004),
       69-111. DOI=10.1017/S0956796803004829
       http://dx.doi.org/10.1017/S0956796803004829ñ
