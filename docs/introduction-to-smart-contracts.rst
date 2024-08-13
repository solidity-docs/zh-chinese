###############################
智能合约概述
###############################

.. _simple-smart-contract:

***********************
简单的智能合约
***********************

让我们从一个基本的例子开始，这个例子设置了一个变量的值，并将其暴露给其他合约来访问。
如果您现在不理解这些东西也没关系，我们稍后会讨论更多细节。

存储合约示例
===============

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract SimpleStorage {
        uint storedData;

        function set(uint x) public {
            storedData = x;
        }

        function get() public view returns (uint) {
            return storedData;
        }
    }

第一行告诉您，源代码是根据GPL3.0版本授权的。
在发布源代码是默认的情况下，机器可读的许可证说明是很重要的。

下一行指定源代码是为Solidity 0.4.16版本编写的，或该语言的较新版本，直到但不包括0.9.0版本。
这是为了确保合约不能被新的（有重大改变的）编译器版本编译，在那里它可能会有不同的表现。
:ref:`Pragmas <pragma>` 是编译器关于如何处理源代码的常用指令
（例如， `pragma once <https://en.wikipedia.org/wiki/Pragma_once>`_ ）。

Solidity意义上的合约是代码（其 *函数*）和数据（其 *状态*）的集合，
驻留在以太坊区块链的一个特定地址。
这一行 ``uint storedData;`` 声明了一个名为 ``storedData`` 的状态变量，
类型为 ``uint`` （ *u*\nsigned *int*\eger，共 *256* 位）。
您可以把它看作是数据库中的一个槽，您可以通过调用管理数据库的代码函数来查询和改变它。
在这个例子中，合约定义了可以用来修改或检索变量值的函数 ``set`` 和 ``get``。

要访问当前合约的一个成员（如状态变量），通常不需要添加 ``this.`` 前缀，
只需要通过它的名字直接访问它。
与其他一些语言不同的是，省略它不仅仅是一个风格问题，
它导致了一种完全不同的访问成员的方式，但后面会有更多关于这个问题。

该合约能完成的事情并不多（由于以太坊构建的基础架构的原因），
它能允许任何人在合约中存储一个单独的数字，并且这个数字可以被世界上任何人访问，
且没有可行的办法阻止您发布这个数字。
当然，任何人都可以再次调用 ``set`` ，传入不同的值，覆盖您的数字，
但是这个数字仍会被存储在区块链的历史记录中。
随后，我们会看到怎样施加访问限制，以确保只有您才能改变这个数字。

.. warning::
    小心使用Unicode文本，因为有些字符虽然长得相似（甚至一样），
    但其字符码是不同的，其编码后的字符数组也会不一样。

.. note::
    所有的标识符（合约名称，函数名称和变量名称）都只能使用ASCII字符集。
    UTF-8编码的数据可以用字符串变量的形式存储。

.. index:: ! subcurrency

子货币（Subcurrency）例子
===========================

下面的合约实现了一个最简单的加密货币。
这里，币确实可以无中生有地产生，但是只有创建合约的人才能做到（实现一个不同的发行计划也不难）。
而且，任何人都可以给其他人转币，不需要注册用户名和密码，所需要的只是以太坊密钥对。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.26;

    // This will only compile via IR
    contract Coin {
        // 关键字 "public" 使变量可以从其他合约中访问。
        address public minter;
        mapping(address => uint) public balances;

        // 事件允许客户端对您声明的特定合约变化做出反应
        event Sent(address from, address to, uint amount);

        // 构造函数代码只有在合约创建时运行
        constructor() {
            minter = msg.sender;
        }

        // 向一个地址发送一定数量的新创建的代币
        // 但只能由合约创建者调用
        function mint(address receiver, uint amount) public {
            require(msg.sender == minter);
            balances[receiver] += amount;
        }

        // 错误类型变量允许您提供关于操作失败原因的信息。
        // 它们会返回给函数的调用者。
        error InsufficientBalance(uint requested, uint available);

        // 从任何调用者那里发送一定数量的代币到一个地址
        function send(address receiver, uint amount) public {
            require(amount <= balances[msg.sender], InsufficientBalance(amount, balances[msg.sender]));
            balances[msg.sender] -= amount;
            balances[receiver] += amount;
            emit Sent(msg.sender, receiver, amount);
        }
    }

这个合约引入了一些新的概念，让我们逐一解读。

``address public minter;`` 这一行声明了一个可以被公开访问的 :ref:`address<address>` 类型的状态变量。
``address`` 类型是一个160位的值，且不允许任何算数操作。
这种类型适合存储合约地址或 :ref:`外部账户 <accounts>` 的密钥对。

关键字 ``public`` 自动生成一个函数，允许您在这个合约之外访问这个状态变量的当前值。
如果没有这个关键字，其他的合约没有办法访问这个变量。
由编译器生成的函数的代码大致如下所示（暂时忽略 ``external`` 和 ``view``）：

.. code-block:: solidity

    function minter() external view returns (address) { return minter; }

您可以自己添加一个类似上述的函数，但您会有同名的一个函数和一个变量。
您不需要这样做，编译器会帮您解决这个问题。

.. index:: mapping

下一行， ``mapping(address => uint) public balances;`` 也创建了一个公共状态变量，
但它是一个更复杂的数据类型。
:ref:`映射 <mapping-types>` 类型将地址映射到 :ref:`无符号整数 <integers>`。

<<<<<<< HEAD
映射可以被看作是 `哈希表 <https://en.wikipedia.org/wiki/Hash_table>`_，
它实际上是被初始化的，因此每一个可能的键从一开始就存在，并被映射到一个值，其字节表示为全零的值。
然而，它既不可能获得一个映射的所有键的列表，也不可能获得所有值的列表。
因此，要么记住您添加到映射中的内容，要么在不需要的情况下使用它。
甚至更好的是，保留一个列表，或者使用一个更合适的数据类型。
=======
Mappings can be seen as `hash tables <https://en.wikipedia.org/wiki/Hash_table>`_ which are
virtually initialized such that every possible key exists from the start and is mapped to a
value whose byte-representation is all zeros. However, it is neither possible to obtain a list of all keys of
a mapping, nor a list of all values. Record what you
added to the mapping, or use it in a context where this is not needed. Or
even better, keep a list, or use a more suitable data type.
>>>>>>> english/develop

而由 ``public`` 关键字创建的 :ref:`getter 函数 <getter-functions>` 则是更复杂一些的情况，
它大致如下所示：

.. code-block:: solidity

    function balances(address account) external view returns (uint) {
        return balances[account];
    }

您可以用这个函数来查询单个账户的余额。

.. index:: event

这一行 ``event Sent(address from, address to, uint amount);`` 声明了一个 :ref:`"事件" <events>`，
它是在函数 ``send`` 的最后一行发出的。以太坊客户端，如网络应用，可以监听区块链上发出的这些事件，而不需要太多的成本。
一旦发出，监听器就会收到参数 ``from``， ``to`` 和 ``amount``，这使得跟踪交易成为可能。

为了监听这个事件，您可以使用以下方法 JavaScript 代码，
使用 `web3.js <https://github.com/web3/web3.js/>`_ 来创建 ``Coin`` 合约对象，
然后在任何用户界面调用上面自动生成的 ``balances`` 函数：

.. code-block:: javascript

    Coin.Sent().watch({}, '', function(error, result) {
        if (!error) {
            console.log("Coin transfer: " + result.args.amount +
                " coins were sent from " + result.args.from +
                " to " + result.args.to + ".");
            console.log("Balances now:\n" +
                "Sender: " + Coin.balances.call(result.args.from) +
                "Receiver: " + Coin.balances.call(result.args.to));
        }
    })

.. index:: coin

:ref:`constructor <constructor>` 是一个特殊的函数，只在创建合约的过程中执行，事后不能再被调用。
在这种情况下，它永久地存储了创建合约的人的地址。
``msg`` 变量（与 ``tx`` 和 ``block`` 一起）是一个 :ref:`特殊全局变量 <special-variables-functions>`，
其中包含允许访问区块链的属性。 ``msg.sender`` 总是当前（外部）函数调用的地址。

最后，真正被用户或其他合约所调用的，以完成本合约功能的方法是 ``mint`` 和 ``send``。

``mint`` 函数发送一定数量的新创建的代币到另一个地址。
:ref:`require <assert-and-require>` 函数调用定义了一些条件，如果不满足这些条件就会恢复所有的变化。
在这个例子中， ``require(msg.sender == minter);`` 确保只有合约的创建者可以调用 ``mint``。
一般来说，创建者可以随心所欲地铸造代币，但在某些时候，这将导致一种叫做 “溢出” 的现象。
请注意，由于默认的 :ref:`检查过的算术 <unchecked>`，如果表达式 ``balances[receiver] += amount;`` 溢出，
即当任意精度算术中的 ``balances[receiver] + amount`` 大于 ``uint`` 的最大值（ ``2**256 - 1``）时，
交易将被恢复。对于函数 ``send`` 中的语句 ``balances[receiver] += amount;`` 也是如此。

<<<<<<< HEAD
:ref:`错误（Errors） <errors>` 允许您向调用者提供更多关于一个条件或操作失败原因的信息。
错误与 :ref:`恢复状态 <revert-statement>` 一起使用。 ``revert`` 语句无条件地中止和恢复所有的变化，
类似于 ``require`` 函数，但它也允许您提供错误的名称和额外的数据，
这些数据将提供给调用者（并最终提供给前端应用程序或区块资源管理器），以便更容易调试失败或做出反应。
=======
:ref:`Errors <errors>` allow you to provide more information to the caller about
why a condition or operation failed. Errors are used together with the
:ref:`revert statement <revert-statement>`. The ``revert`` statement unconditionally
aborts and reverts all changes, much like the :ref:`require function <assert-and-require-statements>`.
Both approaches allow you to provide the name of an error and additional data which will be supplied to the caller
(and eventually to the front-end application or block explorer) so that
a failure can more easily be debugged or reacted upon.
>>>>>>> english/develop

任何人（已经拥有一些这样的代币）都可以使用 ``send`` 函数来发送代币给其他任何人。
如果发送者没有足够的代币可以发送， 那么 ``if`` 条件就会为真。
因此， ``revert`` 将导致操作失败，同时使用 ``InsufficientBalance`` 错误向发送者提供错误细节。

.. note::
    如果您用这个合约向一个地址发送代币，当您在区块链浏览器上查看该地址时，
    您不会看到任何东西，因为您发送代币的记录和变化的余额只存储在这个特定的代币合约的数据存储中。
    通过使用事件，您可以创建一个 “区块链浏览器”，跟踪您的新币的交易和余额，
    但您必须检查币合约地址，而不是币主的地址。

.. _blockchain-basics:

*****************
区块链基础
*****************

对于程序员来说，区块链这个概念并不难理解，这是因为大多数难懂的东西
（挖矿, `哈希 <https://en.wikipedia.org/wiki/Cryptographic_hash_function>`_ ，
`椭圆曲线密码学 <https://en.wikipedia.org/wiki/Elliptic_curve_cryptography>`_，
`点对点网络（P2P） <https://en.wikipedia.org/wiki/Peer-to-peer>`_ 等）
都只是用于提供特定的功能和承诺。
您只需接受这些既有的特性功能，不必关心底层技术，
比如，难道您必须知道亚马逊的 AWS 内部原理，您才能使用它吗？

.. index:: transaction

交易/事务
============

区块链是全球共享的事务性数据库，这意味着每个人都可加入网络来阅读数据库中的记录。
如果您想改变数据库中的某些东西，您必须创建一个被所有其他人所接受的事务。
事务一词意味着您想做的（假设您想要同时更改两个值），要么一点没做，要么全部完成。
此外，当您的事务被应用到数据库时，其他事务不能修改数据库。

举个例子，设想一张表，列出电子货币中所有账户的余额。
如果请求从一个账户转移到另一个账户，
数据库的事务特性确保了如果从一个账户扣除金额，它总被添加到另一个账户。
如果由于某些原因，无法添加金额到目标账户时，源账户也不会发生任何变化。

此外，交易总是由发送人（创建者）签名。
这样，就可非常简单地为数据库的特定修改增加访问保护机制。
在电子货币的例子中，一个简单的检查确保只有持有账户密钥的人才能从中转移一些补偿，例如以太币。

.. index:: ! block

区块
======

要克服的一个主要障碍是（用比特币的术语）所谓的 “双花攻击 (double-spend attack)”：
如果网络中存在两个交易，都想清空一个账户，会发生什么？
只有其中一个交易是有效的，通常是最先被接受的那个。
问题是，在点对点的网络中，“第一” 不是一个客观的术语。

对此，抽象的答案是，您不必在意。一个全球公认的交易顺序将为您选择，
解决这样的冲突。这些交易将被捆绑成所谓的 “区块”，
然后它们将在所有参与节点中执行和分发。
如果两个交易相互矛盾，最终排在第二位的那个交易将被拒绝，不会成为区块的一部分。

这些区块按时间顺序形成线性序列，这就是“区块链”一词的由来。
区块以固定的时间间隔添加到链上，尽管这些间隔在未来可能会有所改变。
为了获取最新的信息，建议监控网络，例如在 `Etherscan <https://etherscan.io/chart/blocktime>`_ 上进行监测。

<<<<<<< HEAD
作为 “顺序选择机制”（也就是所谓的“挖矿”）的一部分，
可能有时会发生块（blocks）被回滚的情况，但仅在链的“末端”。
末端增加的块越多，其发生回滚的概率越小。
因此您的交易被回滚甚至从区块链中抹除，这是可能的，
但等待的时间越长，这种情况发生的概率就越小。
=======
As part of the "order selection mechanism", which is called `attestation <https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/attestations/>`_, it may happen that
blocks are reverted from time to time, but only at the "tip" of the chain. The more
blocks are added on top of a particular block, the less likely this block will be reverted. So it might be that your transactions
are reverted and even removed from the blockchain, but the longer you wait, the less
likely it will be.
>>>>>>> english/develop

.. note::
    交易不保证被包括在下一个区块或任何特定的未来区块中，
    因为这不是由交易的提交者决定的，而是由矿工来决定交易被包括在哪个区块中。

    如果您想安排您的合约的未来调用，您可以使用智能合约自动化工具或oracle服务。

.. _the-ethereum-virtual-machine:

.. index:: !evm, ! ethereum virtual machine

****************************
以太坊虚拟机
****************************

概述
========

以太坊虚拟机或EVM是以太坊智能合约的运行环境。
它不仅是沙盒封装的，而且实际上是完全隔离的，
这意味着在EVM内运行的代码不能访问网络，文件系统或其他进程。
甚至智能合约之间的访问也是受限的。

.. index:: ! account, address, storage, balance

.. _accounts:

账户
========

在以太坊有两种共享同一地址空间的账户：
**外部账户**，由公钥-私钥对（也就是人）控制；
**合约账户**，由与账户一起存储的代码控制。

外部账户的地址是由公钥确定的，
而合约的地址是在合约创建时确定的
（它是由创建者地址和从该地址发出的交易数量得出的，即所谓的 “nonce”）。

无论账户是否存储代码，这两种类型都被EVM平等对待。

每个账户都有一个持久的键值存储，将256位的字映射到256位的字，称为 **存储**。

此外，每个账户有一个以太 **余额** （ balance ）（单位是“Wei”， ``1 ether`` 是 ``10**18 wei``），
余额会因为发送包含以太币的交易而改变。

.. index:: ! transaction

交易
============

交易可以看作是从一个帐户发送到另一个帐户的消息
（这里的账户，可能是相同的或特殊的零帐户，请参阅下文）。
它能包含一个二进制数据（被称为“合约负载”）和以太。

如果目标账户含有代码，此代码会被执行，并以合约负载（二进制数据） 作为入参。

如果目标账户没有设置（交易没有接收者或接收者被设置为 ``null``），
交易会创建一个 **新合约**。
正如已经提到的，该合约的地址不是零地址，
而是从发送者和其发送的交易数量（“nonce”）中得出的地址。
这种合约创建交易的有效负载被认为是EVM字节码并被执行。
该执行的输出数据被永久地存储为合约的代码。
这意味着，为创建一个合约，您不需要发送实际的合约代码，而是发送能够产生合约代码的代码。

.. note::
  在合约创建的过程中，它的代码还是空的。
  所以直到构造函数执行结束，您都不应该在其中调用合约自己函数。

.. index:: ! gas, ! gas price

Gas
===

一经创建，每笔交易都会被收取一定数量的 **gas**，
这些 gas 必须由交易的发起人 （ ``tx.origin``）支付。
在 EVM 执行交易时，gas 根据特定规则逐渐耗尽。
如果 gas 在某一点被用完（即它会为负），
将触发一个 gas 耗尽异常，
这将结束执行并撤销当前调用栈中对状态所做的所有修改。

此机制激励了对 EVM 执行时间的经济利用，
并为 EVM 执行器（即矿工/持币者）的工作提供补偿。
由于每个区块都有最大 gas 量，因此还限制了验证块所需的工作量。

**gas price** 是交易发起人设定的值，
他必须提前向 EVM 执行器支付 ``gas_price * gas``。
如果执行后还剩下一些 gas，则退还给交易发起人。
如果发生撤销更改的异常，已经使用的 gas 不会退还。

由于 EVM 执行器可以选择包含一笔交易，
因此交易发送者无法通过设置低 gas 价格滥用系统。

.. index:: ! storage, ! memory, ! stack

存储，内存和栈
=============================

以太坊虚拟机有三个存储数据的区域：存储器，内存和堆栈。

每个账户都有一个称为 **存储** 的数据区，在函数调用和交易之间是持久的。
存储是一个键值存储，将256位的字映射到256位的字。
在合约中枚举存储是不可能的，读取的成本相对较高，初始化和修改存储的成本更高。
由于这种成本，您应该把您存储在持久性存储中的内容减少到合约运行所需的程度。
在合约之外存储像派生计算，缓存和聚合的数据。合约既不能读也不能写到除其自身以外的任何存储。

第二个数据区被称为 **内存**，合约在每次消息调用时都会获得一个新清除的实例。
内存是线性的，可以在字节级寻址，但读的宽度限制在256位，
而写的宽度可以是8位或256位。当访问（无论是读还是写）一个先前未触及的内存字（即一个字内的任何偏移）时，
内存被扩展一个字（256位）。在扩展的时候，必须支付gas成本。
内存越大，成本就越高（它以平方级别扩展）。

EVM 不是基于寄存器的，而是基于栈的，因此所有的计算都在一个被称为 **栈（stack）** 的区域执行。
栈最大有1024个元素，每个元素长度是一个字（256位）。对栈的访问只限于其顶端，限制方式为：
允许拷贝最顶端的16个元素中的一个到栈顶，或者是交换栈顶元素和下面16个元素中的一个。
所有其他操作都只能取最顶的两个（或一个，或更多，取决于具体的操作）元素，
运算后，把结果压入栈顶。当然可以把栈上的元素放到存储或内存中。
但是无法只访问栈上指定深度的那个元素，除非先从栈顶移除其他元素。

.. index:: ! instruction

指令集
===============

EVM的指令集应尽量保持最小，以避免不正确或不一致的实现，这可能导致共识问题。
所有的指令都是在基本的数据类型上操作的，256位的字或内存的片断（或其他字节数组）。
具备常用的算术，位，逻辑和比较操作。也可以做到有条件和无条件跳转。
此外，合约可以访问当前区块的相关属性，比如它的编号和时间戳。

关于完整的列表，请参见 :ref:`操作码列表 <opcodes>`，它是内联汇编文档的一部分。

.. index:: ! message call, function;call

消息调用
=============

合约可以通过消息调用的方式来调用其它合约或者发送以太币到非合约账户。
消息调用和交易非常类似，它们都有一个源，目标，数据，以太币，gas和返回数据。
事实上每个交易都由一个顶层消息调用组成，这个消息调用又可创建更多的消息调用。

合约可以决定它剩余的 **gas** 有多少应该随内部消息调用一起发送，有多少它想保留。
如果在内部调用中发生了out-of-gas的异常（或任何其他异常），这将由一个被压入栈顶的错误值来表示。
在这种情况下，只有与调用一起发送的gas被用完。
在Solidity中，在这种情况下，发起调用的合约默认会引起一个手动异常，
所以异常会在调用栈上 “冒泡出来”。

如前文所述，被调用的合约（可以与调用者是同一个合约）将收到一个新清空的内存实例，
并可以访问调用的有效负载-由被称为 **calldata** 的独立区域所提供的数据。
在它执行完毕后，它可以返回数据，这些数据将被存储在调用者内存中由调用者预先分配的位置。
所有这样的调用都是完全同步的。

调用被 **限制** 在1024的深度，这意味着对于更复杂的操作，循环应优先于递归调用。
此外，在一个消息调用中，只有63/64的gas可以被转发，这导致在实践中，深度限制略低于1000。

.. index:: delegatecall, library

委托调用和库
==========================

存在一种特殊的消息调用，被称为 **委托调用（delegatecall）**，
除了目标地址的代码是在调用合约的上下文（即地址）中执行，
``msg.sender`` 和 ``msg.value`` 的值不会更改之外，其他与消息调用相同。

这意味着合约可以在运行时动态地从不同的地址加载代码。
存储，当前地址和余额仍然指的是调用合约，只是代码取自被调用的地址。

这使得在Solidity中实现 “库” 的功能成为可能：
可重复使用的库代码，可以放在一个合约的存储上，例如，用来实现复杂的数据结构的库。

.. index:: log

日志
====

有一种特殊的可索引的数据结构，其存储的数据可以一路映射直到区块层级。
这个特性被称为 **日志（logs）** ，Solidity用它来实现 :ref:`事件 <events>`。
合约创建之后就无法访问日志数据，但是这些数据可以从区块链外高效的访问。
因为部分日志数据被存储在 `布隆过滤器（bloom filter） <https://en.wikipedia.org/wiki/Bloom_filter>`_ 中，
我们可以高效并且加密安全地搜索日志，所以那些没有下载整个区块链的网络节点（轻客户端）也可以找到这些日志。

.. index:: contract creation

创建
======

合约甚至可以通过一个特殊的指令来创建其他合约（不是简单的调用零地址）。
创建合约的调用 **create calls** 和普通消息调用的唯一区别在于，负载会被执行，
执行的结果被存储为合约代码，调用者/创建者在栈上得到新合约的地址。

.. index:: ! selfdestruct, deactivate

停用和自毁
============================

从区块链上删除代码的唯一方法是当该地址的合约执行 ``selfdestruct`` 操作。
存储在该地址的剩余以太币被发送到一个指定的目标，然后存储和代码被从状态中删除。
删除合约在理论上听起来是个好主意，但它有潜在的危险性，
因为如果有人向被删除的合约发送以太币，以太币就会永远丢失。

.. warning::
<<<<<<< HEAD
    从0.8.18及更高版本开始，在 Solidity 和 Yul 中使用 ``selfdestruct`` 将触发弃用警告，
    因为 ``SELFDESTRUCT`` 操作码最终将经历 `EIP-6049 <https://eips.ethereum.org/EIPS/eip-6049>`_ 
    中所述的行为的重大变化。
=======
    From ``EVM >= Cancun`` onwards, ``selfdestruct`` will **only** send all Ether in the account to the given recipient and not destroy the contract.
    However, when ``selfdestruct`` is called in the same transaction that creates the contract calling it,
    the behaviour of ``selfdestruct`` before Cancun hardfork (i.e., ``EVM <= Shanghai``) is preserved and will destroy the current contract,
    deleting any data, including storage keys, code and the account itself.
    See `EIP-6780 <https://eips.ethereum.org/EIPS/eip-6780>`_ for more details.

    The new behaviour is the result of a network-wide change that affects all contracts present on
    the Ethereum mainnet and testnets.
    It is important to note that this change is dependent on the EVM version of the chain on which
    the contract is deployed.
    The ``--evm-version`` setting used when compiling the contract has no bearing on it.

    Also, note that the ``selfdestruct`` opcode has been deprecated in Solidity version 0.8.18,
    as recommended by `EIP-6049 <https://eips.ethereum.org/EIPS/eip-6049>`_.
    The deprecation is still in effect and the compiler will still emit warnings on its use.
    Any use in newly deployed contracts is strongly discouraged even if the new behavior is taken into account.
    Future changes to the EVM might further reduce the functionality of the opcode.
>>>>>>> english/develop

.. warning::
    即使一个合约通过 ``selfdestruct`` 删除，它仍然是区块链历史的一部分，
    可能被大多数以太坊节点保留。
    因此，使用 ``selfdestruct`` 与从硬盘上删除数据不一样。

.. note::
    尽管一个合约的代码中没有显式地调用 ``selfdestruct`` ，
    它仍然有可能通过 ``delegatecall`` 或  ``callcode`` 执行自毁操作。

如果您想停用您的合约，您可以通过改变一些内部状态来 **停用** 它们，
从而使再次调用所有的功能都会被恢复。这样就无法使用合约了，因为它立即返回以太。


.. index:: ! precompiled contracts, ! precompiles, ! contract;precompiled

.. _precompiledContracts:

预编译合约
=====================

<<<<<<< HEAD
有一小群合约地址是特殊的。 ``1`` 和（包括） ``8`` 之间的地址范围包含 “预编译合约“，
可以像其他合约一样被调用，但它们的行为（和它们的gas消耗）
不是由存储在该地址的EVM代码定义的（它们不包含代码），
而是由EVM执行环境本身实现。
=======
There is a small set of contract addresses that are special:
The address range between ``1`` and (including) ``0x0a`` contains
"precompiled contracts" that can be called as any other contract
but their behavior (and their gas consumption) is not defined
by EVM code stored at that address (they do not contain code)
but instead is implemented in the EVM execution environment itself.
>>>>>>> english/develop

不同的EVM兼容链可能使用不同的预编译合约集。
未来也有可能在以太坊主链上添加新的预编译合约，
但您可以合理地预期它们总是在 ``1`` 和 ``0xffff`` （包括）之间。
