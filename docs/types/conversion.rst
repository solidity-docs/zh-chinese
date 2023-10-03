.. index:: ! type;conversion, ! cast

.. _types-conversion-elementary-types:

基本类型之间的转换
==================

隐式转换
-----------

在某些情况下，在赋值过程中，在向函数传递参数和应用运算符时，
编译器会自动应用隐式类型转换。一般来说，如果在语义上有意义，
并且不会丢失信息，那么值-类型之间的隐式转换是可能的。

例如， ``uint8`` 可以转换为 ``uint16``， ``int128`` 可以转换为 ``int256``，
但是 ``int8`` 不能转换为 ``uint256``，因为 ``uint256`` 不能容纳 ``-1`` 这样的值。

如果一个运算符被应用于不同的类型，
编译器会尝试将其中一个操作数隐含地转换为另一个的类型（对于赋值也是如此）。
这意味着操作总是以其中一个操作数的类型进行。

关于哪些隐式转换是可能的，请参考关于类型本身的章节。

在下面的例子中， ``y`` 和 ``z``，即加法的操作数，没有相同的类型，
但是 ``uint8`` 可以隐式转换为 ``uint16``，反之则不行。正因为如此，
``y`` 被转换为 ``z`` 的类型，然后在 ``uint16`` 类型中进行加法。
结果表达式 ``y + z`` 的类型是 ``uint16``。
因为它被分配到一个 ``uint32`` 类型的变量中，所以在加法后又进行了一次隐式转换。

.. code-block:: solidity

    uint8 y;
    uint16 z;
    uint32 x = y + z;


显式转换
---------

<<<<<<< HEAD
如果编译器不允许隐式转换，但您确信转换会成功，
有时可以进行显式类型转换。
这可能会导致意想不到的行为，并使您绕过编译器的一些安全特性，
所以一定要测试结果是否是您想要的和期望的!
=======
If the compiler does not allow implicit conversion but you are confident a conversion will work,
an explicit type conversion is sometimes possible. This may
result in unexpected behavior and allows you to bypass some security
features of the compiler, so be sure to test that the
result is what you want and expect!
>>>>>>> english/develop

以下面的例子为例，将一个负的 ``int`` 转换为 ``uint``：

.. code-block:: solidity

    int  y = -3;
    uint x = uint(y);

在这个代码片断的最后， ``x`` 变成 ``0xfffff..fd`` 的值（64个十六进制字符），
这在256位的二进制补码中表示是-3。

如果一个整数被明确地转换为一个较小的类型，高阶位就会被切断：

.. code-block:: solidity

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // b 现在会是 0x5678

如果一个整数被明确地转换为一个更大的类型，它将在左边被填充（即在高阶的一端）。
转换的结果将与原整数比较相等：

.. code-block:: solidity

    uint16 a = 0x1234;
    uint32 b = uint32(a); // b 现在会是 0x00001234
    assert(a == b);

固定大小的字节类型在转换过程中的行为是不同的。
它们可以被认为是单个字节的序列，转换到一个较小的类型将切断序列：

.. code-block:: solidity

    bytes2 a = 0x1234;
    bytes1 b = bytes1(a); // b 现在会是 0x12

如果一个固定大小的字节类型被明确地转换为一个更大的类型，它将在右边被填充。
访问固定索引的字节将导致转换前后的数值相同（如果索引仍在范围内）：

.. code-block:: solidity

    bytes2 a = 0x1234;
    bytes4 b = bytes4(a); // b 现在会是 0x12340000
    assert(a[0] == b[0]);
    assert(a[1] == b[1]);

于整数和固定大小的字节数组在截断或填充时表现不同，
只有在整数和固定大小的字节数组具有相同大小的情况下，才允许在两者之间进行显式转换。
如果您想在不同大小的整数和固定大小的字节数组之间进行转换，您必须使用中间转换，
使所需的截断和填充规则明确：

.. code-block:: solidity

    bytes2 a = 0x1234;
    uint32 b = uint16(a); // b 将会是 0x00001234
    uint32 c = uint32(bytes4(a)); // c 将会是 0x12340000
    uint8 d = uint8(uint16(a)); // d 将会是 0x34
    uint8 e = uint8(bytes1(a)); // e 将会是 0x12

``bytes`` 数组和 ``bytes`` calldata 切片可以明确转换为固定字节类型（ ``bytes1`` /.../ ``bytes32``）。
如果数组比目标的固定字节类型长，在末端会发生截断的情况。如果数组比目标类型短，它将在末尾被填充零。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.5;

    contract C {
        bytes s = "abcdefgh";
        function f(bytes calldata c, bytes memory m) public view returns (bytes16, bytes3) {
            require(c.length == 16, "");
            bytes16 b = bytes16(m);  // 如果m的长度大于16，将发生截断。
            b = bytes16(s);  // 右边进行填充，所以结果是 "abcdefgh\0\0\0\0\0\0\0\0"
            bytes3 b1 = bytes3(s); // 发生截断, b1 相当于 "abc"
            b = bytes16(c[:8]);  // 同样用0进行填充
            return (b, b1);
        }
    }

.. index:: ! literal;conversion, literal;rational, literal;hexadecimal number
.. _types-conversion-literals:

字面常数和基本类型之间的转换
============================

整数类型
-------------

十进制和十六进制的数字字面常数可以隐含地转换为任何足够大的整数类型去表示它而不被截断：

.. code-block:: solidity

    uint8 a = 12; // 可行
    uint32 b = 1234; // 可行
    uint16 c = 0x123456; // 报错, 因为这将会截断成 0x3456

.. note::
    在0.8.0版本之前，任何十进制或十六进制的数字字面常数都可以显式转换为整数类型。
    从0.8.0开始，这种显式转换和隐式转换一样严格，也就是说，只有当字面意义符合所产生的范围时，才允许转换。

.. index:: literal;string, literal;hexadecimal

固定大小的字节数组
----------------------

十进制数字字面常数不能被隐含地转换为固定大小的字节数组。
十六进制数字字面常数是可以的，但只有当十六进制数字的数量正好符合字节类型的大小时才可以。
但是有一个例外，数值为0的十进制和十六进制数字字面常数都可以被转换为任何固定大小的字节类型：

.. code-block:: solidity

    bytes2 a = 54321; // 不允许
    bytes2 b = 0x12; // 不允许
    bytes2 c = 0x123; // 不允许
    bytes2 d = 0x1234; // 可行
    bytes2 e = 0x0012; // 可行
    bytes4 f = 0; // 可行
    bytes4 g = 0x0; // 可行

字符串和十六进制字符串字面常数可以被隐含地转换为固定大小的字节数组，
如果它们的字符数与字节类型的大小相匹配：

.. code-block:: solidity

    bytes2 a = hex"1234"; // 可行
    bytes2 b = "xy"; // 可行
    bytes2 c = hex"12"; // 不允许
    bytes2 d = hex"123"; // 不允许
    bytes2 e = "x"; // 不允许
    bytes2 f = "xyz"; // 不允许

.. index:: literal;address

地址类型
---------

正如在 :ref:`address_literals` 中所描述的那样，正确大小并通过校验测试的十六进制字是 ``address`` 类型。
其他字面常数不能隐含地转换为 ``address`` 类型。

只允许从 ``bytes20`` 和 ``uint160`` 显式转换到 ``address``。

``address a`` 可以通过 ``payable(a)`` 显式转换为 ``address payable``。

.. note::
    在 0.8.0 版本之前，可以显式地从任何整数类型（任何大小，有符号或无符号）转换为 ``address`` 或 ``address payable`` 类型。
    从 0.8.0 开始，只允许从 ``uint160`` 转换。
