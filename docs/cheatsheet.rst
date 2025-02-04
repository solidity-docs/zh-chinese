**********
速查表
**********

.. index:: operator;precedence

操作符的优先顺序
================================

.. include:: types/operator-precedence-table.rst

.. index:: abi;decode, abi;encode, abi;encodePacked, abi;encodeWithSelector, abi;encodeCall, abi;encodeWithSignature

ABI 编码和解码函数
===================================

- ``abi.decode(bytes memory encodedData, (...)) returns (...)``： :ref:`ABI <ABI>` - 对提供的数据进行解码。类型在括号中作为第二个参数给出。
  示例： ``(uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))``
- ``abi.encode(...) returns (bytes memory)``： :ref:`ABI <ABI>` - 对给定的参数进行编码。
- ``abi.encodePacked(...) returns (bytes memory)``： 对给定的参数执行 :ref:`紧密打包 <abi_packed_mode>`。
  请注意，这种编码可能是不明确的!
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory)``： :ref:`ABI <ABI>` - 对给定参数进行编码，
  并以给定的函数选择器作为起始的 4 字节数据一起返回
- ``abi.encodeCall(function functionPointer, (...)) returns (bytes memory)``： 对 ``functionPointer`` 的调用进行ABI编码，
  参数在元组中找到。执行全面的类型检查，确保类型与函数签名相符。结果等于 ``abi.encodeWithSelector(functionPointer.selector(..))``。
- ``abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)``： 等价于
  ``abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), ...)``

.. index:: bytes;concat, string;concat

``bytes`` 和  ``string`` 的成员方法
====================================

- ``bytes.concat(...) returns (bytes memory)``： :ref:`将可变数量的参数连接成一个字节数组 <bytes-concat>`。

- ``string.concat(...) returns (string memory)``： :ref:`将可变数量的参数连接成一个字符串数组 <string-concat>`。


.. index:: address;balance, address;codehash, address;send, address;code, address;transfer

``address`` 的成员方法
======================

- ``<address>.balance`` (``uint256``)： :ref:`address` 的余额，以 Wei 为单位
- ``<address>.code`` (``bytes memory``)： 在 :ref:`address` 的代码（可以是空的）。
- ``<address>.codehash`` (``bytes32``)： :ref:`address` 的代码哈希值。
- ``<address>.call(bytes memory) returns (bool, bytes memory)``： 用给定的数据执行低阶 ``CALL``，
  返回执行结果和执行后返回的数据
- ``<address>.delegatecall(bytes memory) returns (bool, bytes memory)``: 用给定的数据执行低阶 ``DELEGATECALL``,
  返回执行结果和执行后返回的数据
- ``<address>.staticcall(bytes memory) returns (bool, bytes memory)``: 用给定的数据执行低阶 ``STATICCALL``,
  返回执行结果和执行后返回的数据
- ``<address payable>.send(uint256 amount) returns (bool)``： 向 :ref:`address` 发送给定数量的 Wei，失败时返回 ``false``
- ``<address payable>.transfer(uint256 amount)``： 向 :ref:`address` 发送给定数量的 Wei，失败时会抛出错误

.. index:: blockhash, blobhash, block, block;basefee, block;blobbasefee, block;chainid, block;coinbase, block;difficulty, block;gaslimit, block;number, block;prevrandao, block;timestamp
.. index:: gasleft, msg;data, msg;sender, msg;sig, msg;value, tx;gasprice, tx;origin

区块和交易属性
================================

- ``blockhash(uint blockNumber) returns (bytes32)``： 给定区块的哈希值 - 只对最近的256个区块有效
- ``blobhash(uint index) returns (bytes32)``： 与当前交易相关联的第 ``index`` 个blob。
  此带版本的哈希值是由一个表示版本的单字节（当前为 ``0x01`` ）和紧随其后的KZG证明的SHA256哈希的最后31个字节组成。
  （ `EIP-4844 <https://eips.ethereum.org/EIPS/eip-4844>`_ ）。
- ``block.basefee`` (``uint``)： 当前区块的基本费用 （ `EIP-3198 <https://eips.ethereum.org/EIPS/eip-3198>`_ 和 `EIP-1559 <https://eips.ethereum.org/EIPS/eip-1559>`_ ）
- ``block.blobbasefee`` (``uint``): 当前区块的blob基础费用（ `EIP-7516 <https://eips.ethereum.org/EIPS/eip-7516>`_ 和 `EIP-4844 <https://eips.ethereum.org/EIPS/eip-4844>`_）
- ``block.chainid`` (``uint``)： 当前链的ID
- ``block.coinbase`` (``address payable``)： 当前区块矿工的地址
- ``block.difficulty`` (``uint``)： 当前区块的难度值（ ``EVM < Paris`` ）。对于其他EVM版本，它是 ``block.prevrandao`` 的一个废弃的别名，将在下一个重大改变版本中被删除。
- ``block.gaslimit`` (``uint``)： 当前区块的燃料上限
- ``block.number`` (``uint``)： 当前区块的区块号
- ``block.prevrandao`` (``uint``)： 由信标链提供的随机数（ ``EVM >= Paris`` ）（见 `EIP-4399 <https://eips.ethereum.org/EIPS/eip-4399>`_ ）。
- ``block.timestamp`` (``uint``)： 当前区块的时间戳，自Unix epoch以来的秒数
- ``gasleft() returns (uint256)``： 剩余燃料
- ``msg.data`` (``bytes``)： 完整的调用数据
- ``msg.sender`` (``address``)： 消息发送方（当前调用）
- ``msg.sig`` (``bytes4``)： 调用数据的前四个字节（即函数标识符）。
- ``msg.value`` (``uint``)： 随消息发送的 wei 的数量
- ``tx.gasprice`` (``uint``)： 交易的燃料价格
- ``tx.origin`` (``address``)： 交易发送方（完整调用链上的原始发送方）

.. index:: assert, require, revert

验证和断言
==========================

- ``assert(bool condition)``： 如果条件为 ``false``，则中止执行并恢复状态变化（用于内部错误）。
- ``require(bool condition)``： 如果条件为 ``false``，则中止执行并恢复状态变化（用于错误的输入或外部组件的错误）。
- ``require(bool condition, string memory message)``： 如果条件为 ``false``，则中止执行并恢复状态变化（用于错误的输入或外部组件的错误）。同时提供错误信息。
- ``revert()``： 中止执行并恢复状态变化
- ``revert(string memory message)``： 中止执行并恢复状态变化，提供一个解释性的字符串

.. index:: cryptography, keccak256, sha256, ripemd160, ecrecover, addmod, mulmod

数学和密码方法
========================================

- ``keccak256(bytes memory) returns (bytes32)``： 计算输入的Keccak-256哈希值
- ``sha256(bytes memory) returns (bytes32)``： 计算输入的SHA-256哈希值
- ``ripemd160(bytes memory) returns (bytes20)``： 计算输入的RIPEMD-160的哈希值
- ``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``： 从椭圆曲线签名中恢复与公钥相关的地址，错误时返回0
- ``addmod(uint x, uint y, uint k) returns (uint)``： 计算 ``(x + y) % k`` 的值，其中加法的结果即使超过 ``2**256`` 也不会被截取。
  从 0.5.0 版本开始会加入对 ``k != 0`` 的 assert（即会在此函数开头执行 ``assert(k != 0);`` 作为参数检查，译者注）。
- ``mulmod(uint x, uint y, uint k) returns (uint)``： 计算 ``(x * y) % k`` 的值，其中乘法的结果即使超过 ``2**256`` 也不会被截取。
  从 0.5.0 版本开始会加入对 ``k != 0`` 的 assert（即会在此函数开头执行 ``assert(k != 0);`` 作为参数检查，译者注）。

.. index:: this, super, selfdestruct

合约相关方法
================

- ``this`` （当前合约的类型）： 当前合约，可明确转换为 ``address`` 或 ``address payable``。
- ``super``： 继承层次中高一级的合约
- ``selfdestruct(address payable recipient)``： 销毁当前合约，将其资金发送到给定的地址。
  
.. index:: type;name, type;creationCode, type;runtimeCode, type;interfaceId, type;min, type;max

类型相关信息
================

- ``type(C).name`` (``string``)： 合约的名称
- ``type(C).creationCode`` (``bytes memory``)： 给定合约的创建字节码，参见 :ref:`类型信息 <meta-type>`。
- ``type(C).runtimeCode`` (``bytes memory``)： 给定合约的运行时字节码，参见 :ref:`类型信息 <meta-type>`。
- ``type(I).interfaceId`` (``bytes4``)： 包含给定接口的EIP-165接口标识符的值，参见 :ref:`类型信息 <meta-type>`。
- ``type(T).min`` (``T``)： 整数类型 ``T`` 所能代表的最小值，参见 :ref:`类型信息 <meta-type>`。
- ``type(T).max`` (``T``)： 整数类型 ``T`` 所能代表的最大值，参见 :ref:`类型信息 <meta-type>`。


.. index:: visibility, public, private, external, internal

函数可见性说明符
==============================

.. code-block:: solidity
    :force:

    function myFunction() <visibility specifier> returns (bool) {
        return true;
    }

- ``public``： 内部、外部均可见（参考为存储/状态变量创建 :ref:`getter function<getter-functions>` 函数）
- ``private``： 仅在当前合约内可见
- ``external``： 仅在外部可见（仅可修饰函数）——就是说，仅可用于消息调用（即使在合约内调用，也只能通过 ``this.func`` 的方式）
- ``internal``： 仅在内部可见（也就是在当前 Solidity 源代码文件内均可见，不仅限于当前合约内，译者注）


.. index:: modifiers, pure, view, payable, constant, anonymous, indexed

修饰器
=========

- ``pure`` 修饰函数时：不允许修改或访问状态变量。
- ``view`` 修饰函数时：不允许修改状态变量。
- ``payable`` 修饰函数时：允许从调用中接收以太币。
- ``constant`` 修饰状态变量时：不允许赋值（除初始化以外），不会占据存储插槽（storage slot）。
- ``immutable`` 修饰状态变量时：允许在构造时分配并在部署时保持不变。存储在代码中。
- ``anonymous`` 修饰事件时：不把事件签名作为 topic 存储。
- ``indexed`` 修饰事件参数时：将参数作为 topic 存储。
- ``virtual`` 修饰函数和修改时：允许在派生合约中改变函数或修改器的行为。
- ``override`` 表示该函数、修改器或公共状态变量改变了基类合约中的函数或修改器的行为。

