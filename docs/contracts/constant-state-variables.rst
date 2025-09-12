.. index:: ! constant

.. _constants:

**************************************
常量和不可变状态变量
**************************************

状态变量可以被声明为 ``constant`` 或 ``immutable``。
在这两种情况下，变量在合约构建完成后不能被修改。
对于 ``constant`` 变量，其值必须在编译时固定，
而对于 ``immutable`` 变量，仍然可以在构造时分配。

也可以在文件级别定义 ``constant`` 变量。

<<<<<<< HEAD
编译器并没有为这些变量预留存储，它们的每次出现都会被替换为相应的常量表达式。
=======
Every occurrence of such a variable in the source is replaced by its underlying value
and the compiler does not reserve a storage slot for it.
It cannot be assigned a slot in transient storage using the ``transient`` keyword either.
>>>>>>> english/develop

与普通的状态变量相比，常量变量（constant）和不可改变的变量（immutable）的燃料成本要低得多。
对于常量变量，分配给它的表达式被复制到所有访问它的地方，并且每次都要重新评估，
这使得局部优化成为可能。不可变的变量在构造时被评估一次，其值被复制到代码中所有被访问的地方。
对于这些值，要保留32个字节，即使它们可以装入更少的字节。由于这个原因，常量值有时会比不可变的值更便宜。

目前，并非所有的常量和不可变量的类型都已实现。
唯一支持的类型是 :ref:`字符串类型 <strings>` （仅用于常量）和 :ref:`值类型 <value-types>`。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.21;

    uint constant X = 32**22 + 8;

    contract C {
        string constant TEXT = "abc";
        bytes32 constant MY_HASH = keccak256("abc");
        uint immutable decimals = 18;
        uint immutable maxBalance;
        address immutable owner = msg.sender;

        constructor(uint decimals_, address ref) {
            if (decimals_ != 0)
                // immutable变量只有在部署时才是不可变的。
                // 在构建时，它们可以被分配任意次数。
                decimals = decimals_;

            // 对immutable变量的赋值甚至可以访问环境变量。
            maxBalance = ref.balance;
        }

        function isBalanceTooHigh(address other) public view returns (bool) {
            return other.balance > maxBalance;
        }
    }


Constant
========

对于 ``constant`` 变量，其值在编译时必须是一个常量，并且必须在变量声明的地方分配。
任何访问存储、区块链数据（例如： ``block.timestamp``, ``address(this).balance`` 或 ``block.number``）
或执行数据（ ``msg.value`` 或 ``gasleft()``）或者调用外部合约的表达式都是不允许的。
但可能对内存分配产生副作用的表达式是允许的，但那些可能对其他内存对象产生副作用的表达式是不允许的。
内置函数 ``keccak256``， ``sha256``， ``ripemd160``， ``ecrecover``， ``addmod`` 和 ``mulmod``
是允许的（尽管除了 ``keccak256``，它们确实调用了外部合约）。

允许在内存分配器上产生副作用的原因是，
它应该可以构建复杂的对象，比如说查找表。
这个功能现在还不能完全使用。

Immutable
=========

声明为 ``immutable`` 的变量比声明为 ``constant`` 的变量受到的限制要少一些：
不可变（immutable）变量可以在构造时赋值。
该值在部署前的任何时候都可以更改，然后成为永久值。

另外一个限制是，不可变变量只能分配给创建后不可能被执行的表达式内部。
这就排除了所有修改器定义和构造函数以外的函数。

读取不可变变量没有任何限制。
读取甚至可以在变量第一次被写入之前进行，
因为Solidity中的变量总是有一个定义明确的初始值。
因此，也允许永远不对不可变变量显式赋值。

.. warning::
    在构造时访问不可变变量时，请牢记 :ref:`初始化顺序 <state-variable-initialization-order>`。
    即使您提供了显式初始化器，某些表达式可能最终会在初始化器之前被求值，
    尤其是当它们处于继承层次结构的不同层级时。

.. note::
    在 Solidity 0.8.21 之前，不可变变量的初始化限制较多。
    这些变量必须在构造时被初始化一次，而且在此之前不能被读取。

编译器生成的合约创建代码会在返回合约之前修改合约的运行时代码，
并用分配给它们的值替换所有对不可变变量的引用。
如果您要比较编译器生成的运行时代码和实际存储在区块链中的运行时代码，这一点非常重要。
编译器会在 :ref:`编译器JSON标准输出 <compiler-api>` 的 ``immutableReferences`` 字段中
输出这些不可变变量在部署字节码中的位置。
