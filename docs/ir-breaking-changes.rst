
.. index: ir breaking changes

.. _ir-breaking-changes:

*************************************
基于 Solidity 中间表征的 Codegen 变化
*************************************

Solidity 可以通过两种不同的方式生成 EVM 字节码：
要么直接从 Solidity 到 EVM 操作码（“旧编码”），
要么通过在 Yul 中的中间表示法（“IR”）（“新编码” 或 “基于IR的编码”）。

引入基于 IR 的代码生成器的目的是，不仅使代码生成更加透明和可审计，
而且能够实现更强大的跨函数的优化通道。

<<<<<<< HEAD
您可以在命令行中使用 ``--via-ir``
或在 standard-json 中使用 ``{"viaIR": true}`` 选项来启用它，
我们鼓励大家尝试一下！

由于一些原因，旧的和基于 IR 的代码生成器之间存在着微小的语义差异，
主要是在那些我们无论如何也不会期望人们依赖这种行为的领域。
本节强调了旧的和基于IR的代码生成器之间的主要区别。
=======
You can enable it on the command-line using ``--via-ir``
or with the option ``{"viaIR": true}`` in standard-json and we
encourage everyone to try it out!

For several reasons, there are tiny semantic differences between the old
and the IR-based code generator, mostly in areas where we would not
expect people to rely on this behavior anyway.
This section highlights the main differences between the old and the IR-based codegen.
>>>>>>> english/develop

仅有语义上的变化
=====================

本节列出了仅有语义的变化，从而有可能在现有的代码中隐藏新的和不同的行为。

<<<<<<< HEAD
- 在继承的情况下，状态变量初始化的顺序已经改变。
=======
.. _state-variable-initialization-order:

- The order of state variable initialization has changed in case of inheritance.
>>>>>>> english/develop

  以前的顺序是：

  - 所有的状态变量在开始时都被零初始化。
  - 从最终派生合约到最基础的合约评估基础构造函数参数。
  - 从最基础的继承关系到最终派生的继承关系初始化整个继承层次结构中的所有状态变量。
  - 如果存在，在线性化层次结构中从最基础的合约到最终派生的合约依次运行构造函数。

  新的顺序：

  - 所有的状态变量在开始时都被零初始化。
  - 从最终派生合约到最基础的合约评估基础构造函数参数。
  - 对于每一个合约，按照从最基础到最终派生的合约的线性化层次结构的顺序执行：

    1. 初始化状态变量。
    2. 运行构造函数（如果存在）。

  这导致了合约中的差异，即一个状态变量的初始值依赖于另一个合约中构造函数的结果：

  .. code-block:: solidity

      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >=0.7.1;

      contract A {
          uint x;
          constructor() {
              x = 42;
          }
          function f() public view returns(uint256) {
              return x;
          }
      }
      contract B is A {
          uint public y = f();
      }

  以前， ``y`` 会被设置为0。这是由于我们会先初始化状态变量：首先， ``x`` 被设置为0，当初始化 ``y`` 时， ``f()`` 将返回0，导致 ``y`` 也为0。
  在新的规则下， ``y`` 将被设置为42。我们首先将 ``x`` 初始化为0，然后调用 A 的构造函数，将 ``x`` 设置为42。最后，在初始化 ``y`` 时， ``f()`` 返回42，导致 ``y`` 为42。

- 当存储结构被删除时，包含该结构成员的每个存储槽都被完全设置为零。
  以前，填充空间是不被触动的。
  因此，如果结构中的填充空间被用来存储数据（例如在合约升级的背景下），
  您必须注意， ``delete`` 现在也会清除添加的成员（而在过去不会被清除）。

  .. code-block:: solidity

      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >=0.7.1;

      contract C {
          struct S {
              uint64 y;
              uint64 z;
          }
          S s;
          function f() public {
              // ...
              delete s;
              // s只占用了32个字节槽的前16个字节
              // delete 将把零写到完整的插槽中
          }
      }

  我们对隐式删除也有同样的行为，例如当结构体的数组被缩短时。

- 关于函数参数和返回变量，函数修改器的实现方式略有不同。
  如果占位符 ``_;`` 在一个修饰符中被多次使用，这尤其有影响。
  在旧的代码生成器中，每个函数参数和返回变量在堆栈中都有一个固定的槽。
  如果因为多次使用 ``_;`` 而使函数运行多次，或者在一个循环中使用，
  那么函数参数或返回变量的值的变化在函数的下一次执行中是可见的。
  新的代码生成器使用实际的函数来实现修改器，并将函数参数传递下去。
  这意味着对一个函数主体的多次使用将得到相同的参数值，而对返回变量的影响是，
  它们在每次执行时都被重置为其默认值（零）。

  .. code-block:: solidity

      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >=0.7.0;
      contract C {
          function f(uint a) public pure mod() returns (uint r) {
              r = a++;
          }
          modifier mod() { _; _; }
      }

  如果您在旧的代码生成器中执行 ``f(0)``，它将返回 ``1``，
  而在使用新的代码生成器时，它将返回 ``0``。

  .. code-block:: solidity

      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >=0.7.1 <0.9.0;

      contract C {
          bool active = true;
          modifier mod()
          {
              _;
              active = false;
              _;
          }
          function foo() external mod() returns (uint ret)
          {
              if (active)
                  ret = 1; // 与 ``return 1`` 相同
          }
      }

  函数 ``C.foo()`` 返回以下值：

  - 旧的代码生成器： ``1`` 作为返回变量在第一次 ``_;`` 使用前只被初始化为 ``0``，
    然后被 ``return 1;`` 覆盖。在第二次 ``_;`` 使用时，它没有被再次初始化，
    而且 ``foo()`` 也没有明确地分配给它（由于 ``active == false``），因此它保持了它的第一个值。
  - 新的代码生成器： ``0`` 作为所有参数，包括返回参数，将在每次 ``_;`` 使用前被重新初始化。

  .. index:: ! evaluation order; expression

- 对于旧的代码生成器，表达式的评估顺序是没有规定的。
  对于新的代码生成器，我们试图按照源顺序（从左到右）进行评估，但并不保证这一点。
  这可能会导致语义上的差异。

  例如：

  .. code-block:: solidity

      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >=0.8.1;
      contract C {
          function preincr_u8(uint8 a) public pure returns (uint8) {
              return ++a + a;
          }
      }

  函数 ``preincr_u8(1)`` 返回以下值：

  - 旧的代码生成器： ``3`` ( ``1 + 2`` )，但一般情况下返回值是不指定的
  - 新的代码生成器： ``4`` ( ``2 + 2`` )，但不能保证返回值

  .. index:: ! evaluation order; function arguments

  另一方面，除了全局函数 ``addmod`` 和 ``mulmod`` 外，两个代码生成器对函数参数表达式的评估顺序是一样的。
  例如：

  .. code-block:: solidity

      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >=0.8.1;
      contract C {
          function add(uint8 a, uint8 b) public pure returns (uint8) {
              return a + b;
          }
          function g(uint8 a, uint8 b) public pure returns (uint8) {
              return add(++a + ++b, a + b);
          }
      }

  函数 ``g(1, 2)`` 返回以下值：

  - 旧的代码生成器： ``10`` ( ``add(2+3, 2+3)`` )，但返回值一般不指定。
  - 新的代码生成器： ``10``，但不能保证返回值

  全局函数 ``addmod`` 和 ``mulmod`` 的参数由旧代码生成器从右向左评估，新代码生成器从左向右评估。
  例如：

  .. code-block:: solidity

      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >=0.8.1;
      contract C {
          function f() public pure returns (uint256 aMod, uint256 mMod) {
              uint256 x = 3;
              // 旧的代码生成器： add/mulmod(5, 4, 3)
              // 新的代码生成器： add/mulmod(4, 5, 5)
              aMod = addmod(++x, ++x, x);
              mMod = mulmod(++x, ++x, x);
          }
      }

  函数 ``f()`` 返回以下值：

  - 旧的代码生成器： ``aMod = 0`` 和 ``mMod = 2``
  - 新的代码生成器： ``aMod = 4`` 和 ``mMod = 0``

- 新的代码生成器对自由内存指针施加了一个硬性限制 ``type(uint64).max``
  （ ``0xffffffffffffffff``）。其增加值超过这个限制的分配会被恢复。
  旧的代码生成器没有这个限制。

  例如：

  .. code-block:: solidity
      :force:

      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >0.8.0;
      contract C {
          function f() public {
              uint[] memory arr;
              // 分配空间： 576460752303423481
              // 假设freeMemPtr最初指向0x80
              uint solYulMaxAllocationBeforeMemPtrOverflow = (type(uint64).max - 0x80 - 31) / 32;
              // freeMemPtr 因 UINT64_MAX 限制溢出
              arr = new uint[](solYulMaxAllocationBeforeMemPtrOverflow);
          }
      }

  函数 ``f()`` 的作用如下：

  - 旧的代码生成器：在大内存分配后对数组内容进行清零时耗尽了gas
  - 新的代码生成器：由于自由内存指针溢出而还原（不会耗尽gas）。


内部结构
=========

内部函数指针
--------------------------

.. index:: function pointers

旧的代码生成器对内部函数指针的值使用代码偏移量或标签。
这一点特别复杂，因为这些偏移量在构造时和部署后是不同的，而且这些值可以通过存储跨越这个边界。
正因为如此，这两个偏移量在构造时被编码为同一个值（进入不同的字节）。

在新的代码生成器中，函数指针使用依次分配的内部ID。
由于通过跳转的调用是不可能的，通过函数指针的调用总是要使用内部调度函数，
使用 ``switch`` 语句来选择正确的函数。

ID ``0`` 是为未初始化的函数指针保留的，这些指针在被调用时，会引起调度函数的panic错误。

在旧的代码生成器中，内部函数指针是用一个特殊的函数初始化的，它总是引起panic错误。
这导致在构造时对存储中的内部函数指针进行存储写入。

清理
-------

.. index:: cleanup, dirty bits

旧的代码生成器只在操作前执行清理，而操作的结果可能会受到脏位值的影响。
新的代码生成器在任何可能导致脏位的操作之后执行清理。
我们希望优化器能够强大到足以消除多余的清理操作。

例如：

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.1;
    contract C {
        function f(uint8 a) public pure returns (uint r1, uint r2)
        {
            a = ~a;
            assembly {
                r1 := a
            }
            r2 = a;
        }
    }

函数 ``f(1)`` 返回以下值：

- 旧的代码生成器：（ ``fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe``, ``00000000000000000000000000000000000000000000000000000000000000fe``）
- 新的代码生成器：（ ``00000000000000000000000000000000000000000000000000000000000000fe``, ``00000000000000000000000000000000000000000000000000000000000000fe``）

请注意，与新的代码生成器不同，旧的代码生成器在位取反赋值（ ``a = ~a`` ）后没有进行清理。
这导致新旧代码生成器之间对返回值 ``r1`` 的赋值（在内联汇编块内）不同。
然而，两个代码生成器在 ``a`` 的新值被分配到 ``r2`` 之前都进行了清理。
