********************************
Solidity 0.6.0 版本突破性变化
********************************

本节强调了 Solidity 0.6.0 版本中引入的主要突破性变化，以及这些变化背后的原因和如何更新受影响的代码。
对于完整的列表，请查看 `版本更新日志 <https://github.com/ethereum/solidity/releases/tag/v0.6.0>`_。


编译器可能不会发出警告的变化
=========================================

<<<<<<< HEAD
本节列出了一些变化，在这些变化中，您的代码的行为可能会发生变化，而编译器不会告诉您。
=======
This section lists changes where the behavior of your code might
change without the compiler telling you about it.
>>>>>>> english/develop

* 指数运算的结果类型是基数的类型。
  就像对称运算一样，它曾经是可以同时容纳基数类型和指数类型的最小类型。
  此外，指数化的基数允许是有符号的类型。

显性要求
=========================

这一节列出了代码现在需要更显式的更改，但是语义没有改变。
对于大多数主题，编译器会提供建议。

* 现在，只有当函数被标记为 ``virtual`` 关键字或被定义在一个接口中时，才能被重载。
  在接口之外没有实现的函数必须被标记为 ``virtual``。
  当重载一个函数或修改器时，必须使用新的关键字 ``override``。
  当重载一个定义在多个并行基类的函数或修改器时，所有基必须在关键字后面的括号中列出，
  像这样： ``override(Base1, Base2)``。

* 成员对数组的 ``length`` 的访问现在总是只读的，即使是存储数组。
  不再可能通过给存储数组的长度分配一个新值来调整其大小。
  使用 ``push()``， ``push(value)`` 或 ``pop()`` 代替，
  或者分配一个完整的数组，当然这将覆盖现有内容。
  这背后的原因是为了防止巨大的存储阵列的存储碰撞。

* 新的关键字 ``abstract`` 可以用来标记合约为抽象的。
  如果一个合约没有实现它的所有功能，就必须使用这个关键字。抽象合约不能用 ``new`` 操作符创建，
  在编译过程中也不可能为其生成字节码。

* 库合约必须实现其所有功能，而不仅仅是内部功能。

* 在内联汇编中声明的变量名称不能再以 ``_slot`` 或 ``_offset`` 结尾。

* 内联汇编中的变量声明不能再影射内联汇编块外的任何声明。
  如果变量名称中包含一个点，那么它的前缀到点的部分不能与内联汇编块外的任何声明冲突。

* 在内联汇编中，不带参数的操作码现在表示为“内置函数”而不是独立标识符。所以 ``gas`` 现在是 ``gas()``。

* 现在不允许影子状态变量。 一个派生合约只能声明一个状态变量 ``x``，
  如果在它的任何基类合约中都没有同名的可见状态变量。


语义和语法变化
==============================

这一部分列出了您必须修改您的代码，而之后它又做了一些别的事情的变化。

* 现在不允许从外部函数类型转换为 ``address`` 类型。
  相反，外部函数类型有一个叫做 ``address`` 的成员，类似于现有的 ``selector`` 成员。

* 动态存储数组的函数 ``push(value)`` 不再返回新的长度（它什么也不返回）。

* 通常被称为 "回退函数" 的无名函数被拆分为一个新的回退函数，该函数使用 ``fallback`` 关键字定义，
  并使用 ``receive`` 关键字定义一个接收以太的函数。

  * 如果存在，每当调用数据为空时（无论是否收到以太），
    都会调用接收以太函数 receive。此函数是隐式的 ``payable``。

  * 当没有其他函数匹配时，就会调用新的回退函数（如果接收以太的函数receive不存在，则包括调用数据为空的调用）。
    您可以让这个函数是 ``payable`` 函数或不是。如果它不是 ``payable`` 函数，
    那么不匹配任何其他发送价值的函数的交易将恢复。
    只有在采用升级或代理模式时，才需要实现新的回退函数。


新功能
============

本节列出了在Solidity 0.6.0之前不可能实现或很难实现的事情。

* :ref:`try/catch语句 <try-catch>` 允许您对失败的外部调用做出反应。
* ``struct`` 和 ``enum`` 类型可以在文件级别声明。
* 数组切片可以用于calldata数组，例如 ``abi.decode(msg.data[4:], (uint, uint))``
  是对函数调用有效负载进行解码的低级方法。
* Natspec在开发者文档中支持多个返回参数，执行与 ``@param`` 相同的命名检查。
* Yul 和内联汇编有一个新的语句，叫做 ``leave``，可以退出当前函数。
* 现在可以通过 ``payable(x)`` 将 ``address`` 转换为 ``address payable``，
  其中 ``x`` 必须是 ``address`` 类型。


接口变化
=================

<<<<<<< HEAD
本节列出与语言本身无关但对编译器接口有影响的更改。
这些可能会改变您在命令行上使用编译器的方式，例如，您如何使用它的可编程接口，
或者您如何分析它产生的输出。
=======
This section lists changes that are unrelated to the language itself, but that have an effect on the interfaces of
the compiler. These may change the way how you use the compiler on the command-line, how you use its programmable
interface, or how you analyze the output produced by it.
>>>>>>> english/develop

新的错误报告器
~~~~~~~~~~~~~~~~~~

<<<<<<< HEAD
引入一个新的错误报告器，其目的是在命令行上产生更易访问的错误消息。
它在默认情况下是启用的，但是通过 ``--old-reporter`` 可以返回到弃用的旧错误报告器。
=======
A new error reporter was introduced, which aims at producing more accessible error messages on the command-line.
It is enabled by default, but passing ``--old-reporter`` falls back to the deprecated old error reporter.
>>>>>>> english/develop

元数据哈希选项
~~~~~~~~~~~~~~~~~~~~~

<<<<<<< HEAD
编译器现在默认将元数据文件的 `IPFS <https://ipfs.io/>`_ 哈希值附加到字节码的末尾
（详见 :doc:`合约元数据 <metadata>` ） 文档。在0.6.0之前，
编译器默认附加了 `Swarm <https://ethersphere.github.io/swarm-home/>`_ 哈希值，
为了仍然支持这种行为，引入了新的命令行选项 ``--metadata-hash``。
它允许您通过传递 ``--metadata-hash`` 命令行选项的 ``ipfs`` 或 ``swarm`` 值来选择要产生和附加的哈希。
传递 ``none`` 则可以完全删除哈希。
=======
The compiler now appends the `IPFS <https://ipfs.io/>`_ hash of the metadata file to the end of the bytecode by default
(for details, see documentation on :doc:`contract metadata <metadata>`). Before 0.6.0, the compiler appended the
`Swarm <https://ethersphere.github.io/swarm-home/>`_ hash by default, and in order to still support this behavior,
the new command-line option ``--metadata-hash`` was introduced. It allows you to select the hash to be produced and
appended, by passing either ``ipfs`` or ``swarm`` as value to the ``--metadata-hash`` command-line option.
Passing the value ``none`` completely removes the hash.
>>>>>>> english/develop

这些变化也可以通过 :ref:`标准JSON接口 <compiler-api>` 使用，并影响编译器生成的元数据JSON。

读取元数据的推荐方法是读取最后两个字节，以确定CBOR编码的长度，并对该数据块进行适当的解码，
这在 :ref:`元数据部分 <encoding-of-the-metadata-hash-in-the-bytecode>` 中有所解释。

Yul 优化器
~~~~~~~~~~~~~

与传统的字节码优化器一起，:doc:`Yul <yul>` 优化器现在在用 ``--optimize`` 参数调用编译器时默认启用。
可以通过 ``--no-optimize-yul`` 参数在调用编译器时禁用它。这主要影响到使用ABI coder v2的代码。

C API 变化
~~~~~~~~~~~~~

使用 ``libsolc`` 的C API的客户端代码现在可以控制编译器使用的内存。
为了使这一变化保持一致， ``solidity_free`` 被重新命名为 ``solidity_reset``，
增加了函数 ``solidity_alloc`` 和 ``solidity_free``，
``solidity_compile`` 现在返回一个必须通过 ``solidity_free()`` 显式释放的字符串。


如何更新您的代码
=======================

本节详细说明了如何为每一个突破性变化更新先前的代码。

* 将 ``address(f)``  改为 ``f.address``，因为 ``f`` 是外部函数类型。

* 用 ``receive() external payable { ... }``， ``fallback() external [payable] { ... }``
  或这两个函数一起来替换 ``function () external [payable] { ... }``。
  只要有可能，最好只使用 ``receive`` 函数。

* 将 ``uint length = array.push(value)`` 改为 ``array.push(value);``。
  新的长度可以通过 ``array.length`` 访问。

* 将 ``array.length++`` 改为 ``array.push()`` 来增加数组长度，使用 ``pop()`` 来减少存储数组的长度。

* 在一个函数的 ``@dev`` 文档中，为每个命名的返回参数定义一个 ``@return`` 条目，
  将参数的名称作为第一个词。例如，
  如果您有定义为 ``function f() public returns (uint value)`` 的函数 ``f()``，
  并且有 ``@dev`` 注释，那么记录它的返回参数如下： ``@return value The return value.``。
  您可以混合使用命名的和未命名的返回参数文档，只要这些声明是按照它们在元组返回类型中出现的顺序即可。

* 为内联汇编中的变量声明选择唯一的标识符，不与内联汇编块外的声明冲突。

* 在每一个您打算重载的非接口函数上添加 ``virtual``。
  在所有没有具体实现的接口之外的函数上添加 ``virtual``。
  对于单继承，在每个重载的函数上添加 ``override``。对于多重继承，添加 ``override(A, B, ..)``，
  在括号中列出所有定义了重载函数的合约。当多个基类定义同一个函数时，继承的合约必须重载所有冲突的函数。

* 在内联汇编中，将 ``()`` 添加到所有不接受参数的操作码后。
  例如，将 ``pc`` 更改为 ``pc()``，将 ``gas`` 更改为 ``gas()``。
