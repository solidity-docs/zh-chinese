.. index:: style, coding style

#############
风格指南
#############

************
概述
************

本指南旨在为编写 Solidity 代码提供编码规范。
这个指南应该被认为是一个不断发展的文件，随着有用的约定被发现和旧的约定被淘汰，它将随着时间而改变。

许多项目会实施他们自己的编码风格指南。如遇冲突，应优先使用具体项目的风格指南。

本风格指南中的结构和许多建议是取自Python的
`pep8 风格指南 <https://peps.python.org/pep-0008/>`_ 。

本指南并 *不是* 以指导正确或最佳的Solidity编码方式为目的。
本指南的目的是保持代码的 *一致性* 。
来自Python的参考文档 `pep8 <https://peps.python.org/pep-0008/#a-foolish-consistency-is-the-hobgoblin-of-little-minds>`_，
很好地阐述了这个概念。

.. note::

    风格指南讲究一致性。与本风格指南保持一致是重要的。项目内的一致性很重要。一个模块或功能内的一致性最为重要。

    但最重要的是 **知道什么时候应该前后不一致** -- 有时风格指南并不适用。有疑问时，请自行判断。看看其他例子，然后决定什么看起来最好。并应毫不犹豫地询问他人！

***********
代码结构
***********


缩进
===========

每个缩进级别使用4个空格。

制表符或空格
==============

空格是首选的缩进方法。

应该避免混合使用制表符和空格。

空行
===========

在 solidity 源码中合约声明之间留出两个空行。

正确写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract A {
        // ...
    }


    contract B {
        // ...
    }


    contract C {
        // ...
    }

错误写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract A {
        // ...
    }
    contract B {
        // ...
    }

    contract C {
        // ...
    }

在一个合约中的函数声明之间留有一个空行。

在相关联的各组单行语句之间可以省略空行。（例如抽象合约的 stub 函数）。

正确写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    abstract contract A {
        function spam() public virtual pure;
        function ham() public virtual pure;
    }


    contract B is A {
        function spam() public pure override {
            // ...
        }

        function ham() public pure override {
            // ...
        }
    }

错误写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    abstract contract A {
        function spam() virtual pure public;
        function ham() public virtual pure;
    }


    contract B is A {
        function spam() public pure override {
            // ...
        }
        function ham() public pure override {
            // ...
        }
    }

.. _maximum_line_length:

代码行的最大长度
===================

最大建议行长度为120个字符。

折行时应该遵从以下指引。

1. 第一个参数不应该紧跟在左括号后边
2. 用一个，且只用一个缩进
3. 每个函数应该单起一行
4. 结束符号 :code:`);` 应该单独放在最后一行

函数调用

正确写法：

.. code-block:: solidity

    thisFunctionCallIsReallyLong(
        longArgument1,
        longArgument2,
        longArgument3
    );

错误写法：

.. code-block:: solidity

    thisFunctionCallIsReallyLong(longArgument1,
                                  longArgument2,
                                  longArgument3
    );

    thisFunctionCallIsReallyLong(longArgument1,
        longArgument2,
        longArgument3
    );

    thisFunctionCallIsReallyLong(
        longArgument1, longArgument2,
        longArgument3
    );

    thisFunctionCallIsReallyLong(
    longArgument1,
    longArgument2,
    longArgument3
    );

    thisFunctionCallIsReallyLong(
        longArgument1,
        longArgument2,
        longArgument3);

赋值语句

正确写法：

.. code-block:: solidity

    thisIsALongNestedMapping[being][set][toSomeValue] = someFunction(
        argument1,
        argument2,
        argument3,
        argument4
    );

错误写法：

.. code-block:: solidity

    thisIsALongNestedMapping[being][set][toSomeValue] = someFunction(argument1,
                                                                       argument2,
                                                                       argument3,
                                                                       argument4);

事件定义和事件发生

正确写法：

.. code-block:: solidity

    event LongAndLotsOfArgs(
        address sender,
        address recipient,
        uint256 publicKey,
        uint256 amount,
        bytes32[] options
    );

    emit LongAndLotsOfArgs(
        sender,
        recipient,
        publicKey,
        amount,
        options
    );

错误写法：

.. code-block:: solidity

    event LongAndLotsOfArgs(address sender,
                            address recipient,
                            uint256 publicKey,
                            uint256 amount,
                            bytes32[] options);

    emit LongAndLotsOfArgs(sender,
                      recipient,
                      publicKey,
                      amount,
                      options);

源文件编码格式
====================

首选 UTF-8 或 ASCII 编码。

Imports 规范
==============

Import 语句应始终放在文件的顶部。

正确写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    import "./Owned.sol";

    contract A {
        // ...
    }


    contract B is Owned {
        // ...
    }

错误写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract A {
        // ...
    }


    import "./Owned.sol";


    contract B is Owned {
        // ...
    }

函数顺序
==================

排序有助于读者识别他们可以调用哪些函数，并更容易地找到构造函数和 fallback 函数的定义。

函数应根据其可见性和顺序进行分组：

- 构造函数
- receive 函数（如果存在）
- fallback 函数（如果存在）
- 外部函数
- 公共函数
- 内部函数
- 私有函数

在一个分组中，把 ``view`` 和 ``pure`` 函数放在最后。

正确写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    contract A {
        constructor() {
            // ...
        }

        receive() external payable {
            // ...
        }

        fallback() external {
            // ...
        }

        // 外部函数
        // ...

        // 是 view 修饰的外部函数
        // ...

        // 是 pure 修饰的外部函数
        // ...

        // 公共函数
        // ...

        // 内部函数
        // ...

        // 私有函数
        // ...
    }

错误写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    contract A {

        // 外部函数
        // ...

        fallback() external {
            // ...
        }
        receive() external payable {
            // ...
        }

        // 私有函数
        // ...

        // 公共函数
        // ...

        constructor() {
            // ...
        }

        // 内部函数
        // ...
    }

表达式中的空格
=========================

在以下情况下避免无关的空格：

除单行函数声明外，紧接着小括号，中括号或者大括号的内容应该避免使用空格。

正确写法：

.. code-block:: solidity

    spam(ham[1], Coin({name: "ham"}));

错误写法:

.. code-block:: solidity

    spam( ham[ 1 ], Coin( { name: "ham" } ) );

除外：

.. code-block:: solidity

    function singleLine() public { spam(); }

紧接在逗号，分号之前：

正确写法：

.. code-block:: solidity

    function spam(uint i, Coin coin) public;

错误写法:

.. code-block:: solidity

    function spam(uint i , Coin coin) public ;

赋值或其他操作符两边多于一个的空格：

正确写法：

.. code-block:: solidity

    x = 1;
    y = 2;
    longVariable = 3;

错误写法:

.. code-block:: solidity

    x            = 1;
    y            = 2;
    longVariable = 3;

在receive和fallback函数中不要包含空格：

正确写法：

.. code-block:: solidity

    receive() external payable {
        ...
    }

    fallback() external {
        ...
    }

错误写法:

.. code-block:: solidity

    receive () external payable {
        ...
    }

    fallback () external {
        ...
    }


控制结构
==================

用大括号表示一个合约，库，函数和结构。 应该为：

* 开括号与声明应在同一行。
* 闭括号在与之前函数声明对应的开括号保持同一缩进级别上另起一行。
* 开括号前应该有一个空格。

正确写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract Coin {
        struct Bank {
            address owner;
            uint balance;
        }
    }

错误写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract Coin
    {
        struct Bank {
            address owner;
            uint balance;
        }
    }

对于控制结构  ``if``， ``else``， ``while``， 和 ``for`` 的实施建议与以上相同。

另外，诸如 ``if``， ``else``， ``while``， 和 ``for`` 这类的控制结构
和条件表达式的块之间应该有一个单独的空格，
同样的，条件表达式的块和开括号之间也应该有一个空格。

正确写法：

.. code-block:: solidity

    if (...) {
        ...
    }

    for (...) {
        ...
    }

错误写法:

.. code-block:: solidity

    if (...)
    {
        ...
    }

    while(...){
    }

    for (...) {
        ...;}

对于控制结构， *如果* 其主体内容只包含一行，则可以省略括号。

正确写法：

.. code-block:: solidity

    if (x < 10)
        x += 1;

错误写法:

.. code-block:: solidity

    if (x < 10)
        someArray.push(Coin({
            name: 'spam',
            value: 42
        }));

对于具有 ``else`` 或 ``else if`` 子句的 ``if`` 块，
``else`` 应该是与 ``if`` 的闭大括号放在同一行上。
这一规则区别于其他块状结构。

正确写法：

.. code-block:: solidity

    if (x < 3) {
        x += 1;
    } else if (x > 7) {
        x -= 1;
    } else {
        x = 5;
    }


    if (x < 3)
        x += 1;
    else
        x -= 1;

错误写法:

.. code-block:: solidity

    if (x < 3) {
        x += 1;
    }
    else {
        x -= 1;
    }

函数声明
====================

对于简短的函数声明，建议函数体的开括号与函数声明保持在同一行。

闭大括号应该与函数声明的缩进级别相同。

开大括号之前应该有一个空格。

正确写法：

.. code-block:: solidity

    function increment(uint x) public pure returns (uint) {
        return x + 1;
    }

    function increment(uint x) public pure onlyOwner returns (uint) {
        return x + 1;
    }

错误写法:

.. code-block:: solidity

    function increment(uint x) public pure returns (uint)
    {
        return x + 1;
    }

    function increment(uint x) public pure returns (uint){
        return x + 1;
    }

    function increment(uint x) public pure returns (uint) {
        return x + 1;
        }

    function increment(uint x) public pure returns (uint) {
        return x + 1;}

一个函数的修饰符顺序应该是：

1. 可见性
2. 可变性
3. 虚拟性
4. 覆盖性
5. 自定义修饰符

正确写法：

.. code-block:: solidity

    function balance(uint from) public view override returns (uint)  {
        return balanceOf[from];
    }

    function increment(uint x) public pure onlyOwner returns (uint) {
        return x + 1;
    }

<<<<<<< HEAD
错误写法:
=======

No:
>>>>>>> english/develop

.. code-block:: solidity

    function balance(uint from) public override view returns (uint)  {
        return balanceOf[from];
    }

    function increment(uint x) onlyOwner public pure returns (uint) {
        return x + 1;
    }

对于长的函数声明，建议将每个参数放在自己的行中，与函数主体的缩进程度相同。
闭小括号和开括号也应该放在自己的行中，与函数声明的缩进程度相同。

正确写法：

.. code-block:: solidity

    function thisFunctionHasLotsOfArguments(
        address a,
        address b,
        address c,
        address d,
        address e,
        address f
    )
        public
    {
        doSomething();
    }

错误写法:

.. code-block:: solidity

    function thisFunctionHasLotsOfArguments(address a, address b, address c,
        address d, address e, address f) public {
        doSomething();
    }

    function thisFunctionHasLotsOfArguments(address a,
                                            address b,
                                            address c,
                                            address d,
                                            address e,
                                            address f) public {
        doSomething();
    }

    function thisFunctionHasLotsOfArguments(
        address a,
        address b,
        address c,
        address d,
        address e,
        address f) public {
        doSomething();
    }

如果一个长函数声明有修饰符，那么每个修饰符都应该被丢到独立的一行。

正确写法：

.. code-block:: solidity

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public
        onlyOwner
        priced
        returns (address)
    {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(
        address x,
        address y,
        address z
    )
        public
        onlyOwner
        priced
        returns (address)
    {
        doSomething();
    }

错误写法:

.. code-block:: solidity

    function thisFunctionNameIsReallyLong(address x, address y, address z)
                                          public
                                          onlyOwner
                                          priced
                                          returns (address) {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public onlyOwner priced returns (address)
    {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public
        onlyOwner
        priced
        returns (address) {
        doSomething();
    }

多行输出参数和返回值语句应该遵从 :ref:`代码行的最大长度 <maximum_line_length>` 一节的说明。

正确写法：

.. code-block:: solidity

    function thisFunctionNameIsReallyLong(
        address a,
        address b,
        address c
    )
        public
        returns (
            address someAddressName,
            uint256 LongArgument,
            uint256 Argument
        )
    {
        doSomething()

        return (
            veryLongReturnArg1,
            veryLongReturnArg2,
            veryLongReturnArg3
        );
    }

错误写法:

.. code-block:: solidity

    function thisFunctionNameIsReallyLong(
        address a,
        address b,
        address c
    )
        public
        returns (address someAddressName,
                 uint256 LongArgument,
                 uint256 Argument)
    {
        doSomething()

        return (veryLongReturnArg1,
                veryLongReturnArg1,
                veryLongReturnArg1);
    }

对于继承合约中需要参数的构造函数，如果函数声明很长或难以阅读，
建议将基础构造函数像多个修饰符的风格那样，每个下沉到一个新行上书写。

正确写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    // 基础合约，为了使这段代码能被编译
    contract B {
        constructor(uint) {
        }
    }


    contract C {
        constructor(uint, uint) {
        }
    }


    contract D {
        constructor(uint) {
        }
    }


    contract A is B, C, D {
        uint x;

        constructor(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4)
        {
            // 用参数 param5 做一些事情
            x = param5;
        }
    }

错误写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    // 基础合约，为了使这段代码能被编译
    contract B {
        constructor(uint) {
        }
    }


    contract C {
        constructor(uint, uint) {
        }
    }


    contract D {
        constructor(uint) {
        }
    }


    contract A is B, C, D {
        uint x;

        constructor(uint param1, uint param2, uint param3, uint param4, uint param5)
        B(param1)
        C(param2, param3)
        D(param4) {
            x = param5;
        }
    }


    contract X is B, C, D {
        uint x;

        constructor(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4) {
                x = param5;
            }
    }


当用单个语句声明简短函数时，允许在一行中完成。

允许：

.. code-block:: solidity

    function shortFunction() public { doSomething(); }

这些函数声明的准则旨在提高可读性。
因为本指南不会涵盖所有内容，作者应该自行作出最佳判断。

映射
========

在变量声明中，不要用空格将关键字 ``mapping`` 和其类型分开。
不要用空格分隔任何嵌套的 ``mapping`` 关键词和其类型。

正确写法：

.. code-block:: solidity

    mapping(uint => uint) map;
    mapping(address => bool) registeredAddresses;
    mapping(uint => mapping(bool => Data[])) public data;
    mapping(uint => mapping(uint => s)) data;

错误写法:

.. code-block:: solidity

    mapping (uint => uint) map;
    mapping( address => bool ) registeredAddresses;
    mapping (uint => mapping (bool => Data[])) public data;
    mapping(uint => mapping (uint => s)) data;

变量声明
=====================

数组变量的声明在变量类型和括号之间不应该有空格。

正确写法：

.. code-block:: solidity

    uint[] x;

错误写法:

.. code-block:: solidity

    uint [] x;


其他建议
=====================

* 字符串应该用双引号而不是单引号。

正确写法：

.. code-block:: solidity

    str = "foo";
    str = "Hamlet says, 'To be or not to be...'";

错误写法:

.. code-block:: solidity

    str = 'bar';
    str = '"Be yourself; everyone else is already taken." -Oscar Wilde';

* 操作符两边应该各有一个空格。

正确写法：

.. code-block:: solidity
    :force:

    x = 3;
    x = 100 / 10;
    x += 3 + 4;
    x |= y && z;

错误写法:

.. code-block:: solidity
    :force:

    x=3;
    x = 100/10;
    x += 3+4;
    x |= y&&z;

* 为了表示优先级，高优先级操作符两边可以省略空格。
  这样可以提高复杂语句的可读性。
  您应该在操作符两边总是使用相同的空格数：

正确写法：

.. code-block:: solidity

    x = 2**3 + 5;
    x = 2*y + 3*z;
    x = (a+b) * (a-b);

错误写法:

.. code-block:: solidity

    x = 2** 3 + 5;
    x = y+z;
    x +=1;

***************
布局顺序
***************

<<<<<<< HEAD
按以下顺序布置合约的元素：

1. Pragma 语句
2. 导入语句
3. 接口
4. 库
5. 合约
=======
Contract elements should be laid out in the following order:

1. Pragma statements
2. Import statements
3. Events
4. Errors
5. Interfaces
6. Libraries
7. Contracts
>>>>>>> english/develop

在每个合约，库或接口内，使用以下顺序：

1. 类型声明
2. 状态变量
3. 事件
4. 错误
5. 修饰符
6. 函数

.. note::

    在接近事件或状态变量的使用时，声明类型可能会更清楚。

正确写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.4 <0.9.0;

    abstract contract Math {
        error DivideByZero();
        function divide(int256 numerator, int256 denominator) public virtual returns (uint256);
    }

错误写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.4 <0.9.0;

    abstract contract Math {
        function divide(int256 numerator, int256 denominator) public virtual returns (uint256);
        error DivideByZero();
    }


******************
命名规范
******************

当完全采纳和使用命名规范时会产生强大的作用。
当使用不同的规范时，则不会立即获取代码中传达的重要 *元* 信息。

这里给出的命名建议旨在提高可读性，
因此它们不是规则，而是透过名称来尝试和帮助传达最多的信息。

最后，基于代码库中的一致性，本文档中的任何规范总是可以被（代码库中的规范）取代。


命名方式
=============

为了避免混淆，下面的名字用来指明不同的命名方式。

* ``b`` （单个小写字母）
* ``B`` （单个大写字母）
* ``lowercase`` （小写）
* ``UPPERCASE`` （大写）
* ``UPPER_CASE_WITH_UNDERSCORES`` （大写和下划线）
* ``CapitalizedWords`` （驼峰式，首字母大写）
* ``mixedCase``  （混合式，与驼峰式的区别在于首字母小写！）

.. note::

    当在驼峰式命名中使用缩写时，应该将缩写中的所有字母都大写。 因此 HTTPServerError 比 HttpServerError 好。
    当在混合式命名中使用缩写时，除了第一个缩写中的字母小写（如果它是整个名称的开头的话）以外，
    其他缩写中的字母均大写。 因此 xmlHTTPRequest 比 XMLHTTPRequest 更好。


应避免的名称
==============

* ``l`` - el的小写方式
* ``O`` - oh的大写方式
* ``I`` - eye的大写方式

切勿将任何这些用于单个字母的变量名称。 他们经常难以与数字 1 和 0 区分开。


合约和库名称
==========================

* 合约和库名称应该使用驼峰式风格。比如： ``SimpleToken``， ``SmartBank``， ``CertificateHashRepository``， ``Player``， ``Congress``， ``Owned``.。
* 合约和库的名称也应与它们的文件名相符。
* 如果一个合约文件包括多个合约和/或库，那么文件名应该与 *核心合约* 相匹配。但是，如果可以避免的话，不建议这样做。

如下面的例子所示，如果合约名称是 ``Congress``，库名称是 ``Owned``，
那么它们的相关文件名应该是 ``Congress.sol`` 和 ``Owned.sol``。

正确写法：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    // Owned.sol
    contract Owned {
        address public owner;

        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }

        constructor() {
            owner = msg.sender;
        }

        function transferOwnership(address newOwner) public onlyOwner {
            owner = newOwner;
        }
    }

在 ``Congress.sol`` 合约里：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    import "./Owned.sol";


    contract Congress is Owned, TokenRecipient {
        //...
    }

错误写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    // owned.sol
    contract owned {
        address public owner;

        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }

        constructor() {
            owner = msg.sender;
        }

        function transferOwnership(address newOwner) public onlyOwner {
            owner = newOwner;
        }
    }

在 ``Congress.sol`` 合约里：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.7.0;


    import "./owned.sol";


    contract Congress is owned, tokenRecipient {
        //...
    }

结构体名称
==========================

结构体名称应该使用驼峰式风格。比如： ``MyCoin``， ``Position``， ``PositionXY``。

事件名称
===========

事件名称应该使用驼峰式风格。
比如： ``Deposit``， ``Transfer``， ``Approval``， ``BeforeTransfer``， ``AfterTransfer``。

函数名称
==============

函数名称应该使用混合式命名风格。
比如： ``getBalance``， ``transfer``， ``verifyOwner``， ``addMember``， ``changeOwner``。


函数参数命名
=======================

函数参数命名应该使用混合式命名风格。
比如： ``initialSupply``， ``account``， ``recipientAddress``， ``senderAddress``， ``newOwner``。

在编写操作自定义结构的库函数时，
这个结构体应该作为函数的第一个参数，并且应该始终命名为 ``self``。


局部变量和状态变量名称
==============================

使用混合式命名风格。
比如： ``totalSupply``， ``remainingSupply``， ``balancesOf``， ``creatorAddress``， ``isPreSale``， ``tokenExchangeRate``。


常量命名
=========

常量应该全都使用大写字母书写，并用下划线分割单词。
比如： ``MAX_BLOCKS``， ``TOKEN_NAME``， ``TOKEN_TICKER``， ``CONTRACT_VERSION``。


修饰符命名
==============

使用混合式命名风格。比如： ``onlyBy``， ``onlyAfter``， ``onlyDuringThePreSale``。


枚举变量命名
============

在声明简单类型时，枚举应该使用驼峰式风格。
比如： ``TokenGroup``， ``Frame``， ``HashStyle``， ``CharacterLocation``。

避免命名冲突
==========================

* ``singleTrailingUnderscore_``

当所需的名称与现有状态变量，函数，内置或其他保留关键字名称冲突时，建议使用此约定。

非外部函数和变量的下划线前缀
==============================

* ``_singleLeadingUnderscore``

建议对非外部函数和状态变量（ ``private`` 或 ``internal`` ）使用此约定。没有指定可见性的状态变量默认为 ``internal`` 。

在设计智能合约时，面向公众的API（可以被任何帐户调用的函数）是一个重要的考虑因素。
前导下划线允许您立即识别这些函数的意图，
但更重要的是，如果您将一个函数从非外部改为外部（包括 ``public``）并相应地重命名，
这将迫使您在重命名时查看每一个调用点。
这应该是一个重要的手动检查，以防止出现非预期的外部函数
和常见的安全漏洞（避免对这种变化使用全文查找替换工具）。

非外部函数和变量的下划线前缀
==========================================================

* ``_singleLeadingUnderscore``

非外部函数和状态变量（ ``private`` 或 ``internal``）建议使用此约定。没有指定可见性的状态变量默认为 ``internal``。

在设计智能合约时，面向公众的应用程序接口（可被任何账户调用的功能）
是一个重要的考虑因素。
前导下划线可让您立即识别此类函数的意图，
但更重要的是，如果您将函数从非外部函数更改为外部函数（包括 ``public``）
并对其进行相应重命名，这将迫使您在重命名时审查每个调用站点。
这可能是针对非预期外部函数的重要手动检查，
也是安全漏洞的常见来源（避免使用查找-替换-全部工具来进行此更改）。

.. _style_guide_natspec:

*******
NatSpec
*******

Solidity合约也可以包含NatSpec注释。
它们用三重斜线（ ``///``）或双星号块（ ``/** ... */``）来写，
它们应该直接用在函数声明或语句之上。

例如，来自 :ref:`一个简单的智能合约 <simple-smart-contract>` 的合约在添加了注释后看起来就像下面这个：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    /// @author Solidity团队
    /// @title 一个简单的存储例子
    contract SimpleStorage {
        uint storedData;

        /// 存储 `x`。
        /// @param x 要存储的新值
        /// @dev 将数字存储在状态变量 `storedData` 中
        function set(uint x) public {
            storedData = x;
        }

        /// 返回存储的值。
        /// @dev 检索状态变量 `storedData` 的值
        /// @return 存储的值
        function get() public view returns (uint) {
            return storedData;
        }
    }

建议 Solidity 合约使用 :ref:`NatSpec <natspec>` 对所有公共接口（ABI 中的一切）进行完全注释。

请参阅关于 :ref:`NatSpec <natspec>` 的部分，以获得详细解释。
