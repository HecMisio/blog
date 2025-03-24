---
layout: post
title: "Solidity基础"
description: "介绍Solidity基础语法和注意事项"
date: 2025-03-21T13:15:41+08:00
image: "/posts/blockchain/ethereum/solidity/images/chapter_1-cover.jpg"
tags: ["Blockchain", "Solidity"]
---

## 数据类型

Solidity 提供了多种数据类型，用于满足智能合约开发的不同需求。以下是一些常见的数据类型：

<table>
<tr>
    <th>分类</th>
    <th>数据类型</th>
</tr>
<tr>
    <td>值类型</td>
    <td>bool, int[8-256], uint[8-256], address, bytes[1-32]</td>
</tr>
<tr>
    <td>引用类型</td>
    <td>string, bytes, array, mapping</td>
</tr>
<tr>
    <td>特殊类型</td>
    <td>enum, struct</td>
</tr>
</table>

**bytes[1-32]、bytes 和 string**

bytes[1-32] 是固定长度的字节数组，长度范围为1到32字节，bytes 是动态字节数组，string虽然本质上也是一个动态字节数组，但是它存储的是 UTF-8 编码的字符序列，会有隐式的 UTF-8 编码转换操作。因此如果要存储固定长度的字节数据，如哈希值、标识符可以使用 bytes[1-32]，如果是动态大小的字节数据，如文件内容、二进制数据可以使用 bytes，如果是文本内容，如名称、描述，可以使用 string。另外只有 bytes 有 length 属性，要获取 string 的长度需要将 string 转换为 bytes。

```Solidity
bytes32 public hash = keccak256(abi.encodePacked("data"));

bytes public data = "Hello...";
uint256 len = data.length;

string public text = "Hello, Solidity!";
uint256 len = bytes(text).length;
```

**array**

array 分为固定数组和动态数据，取决于声明时是否指定长度。动态数组可以使用 push 和 pop 来在数组末尾添加或移除元素，但无法直接删除中间的某个元素，比较方便的做法是交换末尾元素和删除元素的位置，再删除末尾元素。

```Solidity
uint[3] fixedArray = [1, 2, 3];
fixedArray[0] = 10;

uint[] dynamicArray;
dynamicArray.push(1);
dynamicArray.push(2);
dynamicArray.pop();
```

**mapping**

mapping 是 Solidity 的哈希表，支持读写和删除操作。不过删除操作不是真的删除键值对，而是使用 delete 函数将键值重置为默认值，事实上 delete 函数还支持对其他数据类型进行重置。mapping 不支持检查键值是否存在，一般通过检查值是否为默认值来判断。

```Solidity
mapping(address => uint) public balances;

function getBalance(address _user) public view returns (uint) {
    return balances[_user];
}

function setBalance(address _user, uint _amount) public {
    balances[_user] = _amount;
}

function removeBalance(address _user) public {
    delete balances[_user]; // reset into default value
}

function exists(address _user) public view returns (bool) {
    return balances[_user] != 0;
}
```

mapping 无法使用 struct，array，mapping 作为键值，但可以嵌套使用。

```Solidity
mapping(address => mapping(uint => bool)) public nestedMapping;

function setNestedValue(address _user, uint _key, bool _value) public {
    nestedMapping[_user][_key] = _value;
}

function getNestedValue(address _user, uint _key) public view returns (bool) {
    return nestedMapping[_user][_key];
}
```

**enum**

enum 用于定义一组命名常量，帮助开发者更清晰地表达常量含义，提升代码的可读性和可维护性。enum 的底层是 uint8 类型，默认值是0，即第一个定义的常量。

```Solidity
enum Status { Pending, Approved, Rejected } // 0, 1, 2
Status public currentStatus; // default value: 0

function getStatusIndex() public view returns (uint) {
    return uint(currentStatus); // convert enum to uint
}
```

**struct**

struct 可以包含多种不同类型的字段，用于让用户自定义复杂数据结构，支持嵌套定义。

```Solidity
struct Profile {
    string avatar;
}

struct User {
    string name;
    Profile profile;
}

User public user;

function setUser(string memory _name, string memory _avatar) public {
    user = User(_name, Profile(_avatar));
}
```

### 类型转换

Solidity 支持显式和隐式的类型转换，但需要注意某些转换可能会导致数据丢失或错误。

**整数类型**

值类型之间的转换通常需要显式声明，特别是当目标类型的范围小于源类型时。如果目标类型的范围小于源类型，可能会发生截断。如果将无符号整数转换为有符号整数，且值超出目标类型的范围，可能会导致溢出。

```Solidity
uint256 a = 100;
int256 b = int256(a); // uint256 转为 int256
uint8 c = uint8(a);   // uint256 转为 uint8（可能会截断数据）
```

**地址类型**

可以通过显式转换将 address 转换为 address payable，以便接收以太币。

```Solidity
address addr = 0x1234567890123456789012345678901234567890;
address payable payableAddr = payable(addr); // address 转为 address payable
```

**字节类型**

可以通过显式转换将较小的字节数组扩展为较大的字节数组。

```Solidity
bytes32 b32 = bytes32("hello"); // 将字符串转换为 bytes32
bytes1 b1 = bytes1(b32);        // 截断为 bytes1
```

bytes 可以显式转换为 string，反之亦然。

```Solidity
bytes memory b = bytes("hello");
string memory s = string(b);
```

**布尔类型**

布尔类型不能直接与其他类型互相转换。需要通过逻辑判断来实现。

```Solidity
uint256 a = 1;
bool isTrue = (a > 0); // 通过逻辑判断转换为布尔值
```

**枚举类型**

枚举类型可以显式转换为 uint，反之亦然。转换时，枚举的值必须在其定义的范围内。

```Solidity
enum Status { Pending, Approved, Rejected }
Status status = Status.Pending;

uint statusIndex = uint(status); // 枚举转为 uint
status = Status(statusIndex);    // uint 转为枚举
```

**数组类型**

固定大小数组和动态数组之间不能直接转换。需要手动复制元素。

```Solidity
uint[3] memory fixedArray = [1, 2, 3];
uint[] memory dynamicArray = new uint[](3);

for (uint i = 0; i < fixedArray.length; i++) {
    dynamicArray[i] = fixedArray[i];
}
```

**函数类型**

函数类型可以显式转换为兼容的函数类型。internal 函数和 external 函数之间不能直接转换。

```Solidity
function add(uint a, uint b) public pure returns (uint) {
    return a + b;
}

function(uint, uint) pure returns (uint) func = add; // 函数类型转换
```

**复杂类型转换**

使用 abi.encode 和 abi.decode 进行复杂类型的转换。

```Solidity
bytes memory encoded = abi.encode(1, "hello");
(uint x, string memory y) = abi.decode(encoded, (uint, string));
```

### 可见性

Solidity 中有4种可见性规则，其中 public，private，internal 适用于状态变量和函数，而 external 只能用于函数，状态变量和函数的默认可见性是 internal。

<table>
<tr>
    <th>可见性</th>
    <th>描述</th>
</tr>
<tr>
    <td>public</td>
    <td>可以从合约内部、外部以及继承的合约中访问。</td>
</tr>
<tr>
    <td>private</td>
    <td>只能从合约内部访问，继承的合约和外部都无法访问。</td>
</tr>
<tr>
    <td>internal</td>
    <td>只能从合约内部以及继承的合约中访问，外部无法访问。</td>
</tr>
<tr>
    <td>external</td>
    <td>只能从合约外部访问，准确来说是无法被直接访问，在合约内部需要用 this 关键字调用。</td>
</tr>
</table>

### EVM 存储模式

Solidity 中支持在给函数传参时，通过 calldata 和 memory，将参数存储到指定的 EVM 存储区中，但这只适用于引用类型（string，bytes，array，mapping）和 struct，其他类型参数默认存储在 EVM 的栈中。calldata 会将数据分配到只读存储区，无法修改，memory 会将数据分配到内存中，可以修改。另外当引用类型和结构体作为函数返回值时，只能使用 memory 修饰，因为引用类型和结构体作为返回值只能被存储在内存中。

```Solidity
function processArray(uint[] calldata _input) external pure returns (uint) {
    return _input[0];
}

function modifyArray(uint[] memory _input) public pure returns (uint[] memory) {
    _input[0] = 42;
    return _input;
}
```

EVM 本身共有 5 种存储区，同时支持将合约事件永久记录到区块链日志中。

<table>
<tr>
    <th>存储模式</th>
    <th>描述</th>
</tr>
<tr>
    <td>Storage</td>
    <td>永久存储在链上的数据，生命周期与合约一致，读写成本较高，是合约状态变量的默认类型</td>
</tr>
<tr>
    <td>Calldata</td>
    <td>只读数据存储区，用于存储外部函数调用的参数，成本较低。</td>
</tr>
<tr>
    <td>Memory</td>
    <td>临时存储在内存中的数据，仅在函数调用期间有效，读写成本较低。</td>
</tr>
<tr>
    <td>Stack</td>
    <td>用于存储函数执行时的临时变量，容量有限（1024个槽，每个槽大小为32字节），访问速度最快。</td>
</tr>
<tr>
    <td>Code</td>
    <td>只读存储区，存储合约的字节码，主要用于执行合约逻辑。</td>
</tr>
<tr>
    <td>Logs</td>
    <td>事件日志存储区，用于记录链上事件，供外部系统查询。</td>
</tr>
</table>

### 常量

Solidity 中的状态变量可以被 constant 修饰符标记为常量，被标记为常量的状态变量必须立即赋值。常量不会被存储到 Storage 存储区中，而是被编译器直接编译到合约的运行时代码中。

```Solidity
uint256 constant MAX_SUPPLY = 10000;
address constant OWNER = 0x1234567890abcdef1234567890abcdef12345678;
```

另外状态变量还可以被 immutable 修饰符标记为不可变量，与常量不同的是，不可变量不需要立即赋值，它可以在合约部署时通过构造函数进行初始化，部署之后不可更改。编译器生成的合约创建代码会在返回之前将运行时代码中的不可变量替换为具体的值，因此不可变量也不会被存储到 Storage 存储区中。

```Solidity
uint256 immutable deploymentTime;

constructor() {
    deploymentTime = block.timestamp;
}
```

## 函数

Solidity 中函数是智能合约的核心，用于定义合约的行为和逻辑。函数可以有多个传参，对于引用类型（string，bytes，array，mapping）和 struct 类型的参数必须指定存储模式（calldata，memory），同时函数也可以有多个返回值，返回值如果时引用类型或 struct 类型，必须指定为 memory，因为引用类型和结构体作为返回值只能被存储在内存中。Solidity 函数支持重载，多个同名函数，只要参数不同，即函数签名不同，就可以重复定义。

```Solidity
contract Example {
    function add(uint a) public pure returns (uint) {
        return a + 1;
    }

    function add(uint a, uint b) public pure returns (uint) {
        return a + b;
    }
}
```

Solidity 官方认为当合约中存在多个函数时，应当以如下的顺序组织。

* constructor
* receive function (if exists)
* fallback function (if exists)
* external
* public
* internal
* private

### 函数修饰符

Solidity 的函数拥有多种修饰符，其声明顺序为可见性、可修改性、虚函数、重写和自定义修饰器。

<table>
<tr>
    <th>特性</th>
    <th>描述</th>
</tr>
<tr>
    <td>可见性</td>
    <td>public，private，internal，external，默认是 internal。</td>
</tr>
<tr>
    <td>可修改性</td>
    <td>view，pure，payable，默认为 nonpayable(nonpayable 并不能用于显式定义，就像状态变量无法被显式定义为 storage 一样)，函数可以直接读写状态变量，当指定为 view 时，只能读状态变量，当指定为 pure 时，无法直接读写状态变量，但可以通过调用 view 函数间接读取，当指定为 payable 时，不仅可以读写状态变量，还可以进行以太币交易。</td>
</tr>
<tr>
    <td>虚函数</td>
    <td>使用 virtual 标示函数为虚函数，允许被子合约重写，virtual 无法与 private 修饰符一起使用。详细说明可见面向对象部分。</td>
</tr>
<tr>
    <td>重写</td>
    <td>使用 override 标示函数重写父合约的同名函数，只有父合约中的同名函数为虚函数时才能被重写。详细说明可见面向对象部分。</td>
</tr>
<tr>
    <td>自定义修饰器</td>
    <td>通过自定义修饰器（modifier）常用于检查函数执行权限、执行条件等，提高代码复用率和可读性。可以同时使用多个自定义修饰器，会按照声明顺序依次调用。</td>
</tr>
</table>

```Solidity
modifier onlyOwner() {
    require(msg.sender == owner, "Not the contract owner");
    _;
}

modifier onlyActive() {
    require(isActive, "Contract is not active");
    _;
}

modifier hasCompleted() {
    _;
    require(isCompleted, "The process was not completed");
}

function funcName(
    args...
)
    [public|private|internal|external]
    [view|pure|payable]
    [virtual]
    [override]
    [onlyOwner|onlyActive|hasCompleted]
    returns (
        args...
    )
{
    // ...
}
```

### 构造函数

构造函数（constructor） 是一种特殊的函数，主要作用是设置合约的初始状态，例如初始化状态变量或设置合约的所有者，它仅在合约部署时执行，且只执行一次，无法被重新调用。构造函数的可见性只能是 public 或 internal，默认是 internal，并且构造函数是可选的，如果不需要初始化操作，可以省略。

```Solidity
contract Example {
    address public owner;
    uint public initialValue;

    constructor(uint _initialValue) {
        owner = msg.sender;
        initialValue = _initialValue;
    }
}
```

### Receive 和 Fallback

receive 和 fallback 是两种特殊的函数，用于处理以太币的接收和未知函数的调用。首先它们必须都被声明为 external，并且 receive 必须声明为 payable，而 fallback 是否声明为 payable 是可选的。

receive 函数专门用于接收以太币，当合约接收到以太币且没有调用任何函数或没有传递任何 calldata 时触发。

```Solidity
contract Example {
    event Received(address sender, uint amount);

    receive() external payable {
        emit Received(msg.sender, msg.value);
    }
}

address(example).transfer(1 ether);
```

fallback 函数用于处理未知函数调用，若合约中未定义 receive 函数且被 payable 修饰时，也会处理接收以太币的情况。

```Solidity
contract Example {
    event FallbackCalled(address sender, uint value, bytes data);

    fallback() external payable {
        emit FallbackCalled(msg.sender, msg.value, msg.data);
    }
}

(bool success, ) = address(example).call{value: 1 ether}("nonexistentFunction()");
```

<table>
<tr>
    <th>特性</th>
    <th>receive</th>
    <th>fallback</th>
</tr>
<tr>
    <td>用途</td>
    <td>专门用于接收以太币</td>
    <td>处理未知函数调用，也可接收以太币</td>
</tr>
<tr>
    <td>声明</td>
    <td>必须声明为 external 和 payable</td>
    <td>必须声明为 external，可选是否 payable</td>
</tr>
<tr>
    <td>触发条件</td>
    <td>合约接收到以太币且没有 calldata</td>
    <td>调用未知函数或接收到以太币但未定义 receive 函数（被声明为 payable）</td>
</tr>
<tr>
    <td>优先级</td>
    <td>高于 fallback</td>
    <td>低于 receive</td>
</tr>
<tr>
    <td>返回值</td>
    <td>无</td>
    <td>无</td>
</tr>
</table>

## 错误处理

错误处理是确保智能合约安全性和可靠性的重要部分。Solidity 提供了以下三种主要的错误处理方式——require，revert，assert。

### require，revert 和 assert

**require**

用于验证输入参数或条件是否满足，如果不满足抛出错误并回滚交易，未消耗的 gas 会被退还。

```Solidity
function setOwner(address _newOwner) public {
    require(msg.sender == owner, "Only the owner can set a new owner");
    owner = _newOwner;
}
```

**revert**

用于手动触发错误并回滚交易，未消耗的 gas 会被退还，通常用于复杂条件下的错误处理。

```Solidity
function withdraw(uint _amount) public {
    if (_amount > balance) {
        revert("Insufficient balance");
    }
    balance -= _amount;
}
```

**assert**

用于检查不应该发生的条件，检测代码逻辑中的严重错误，通常是内部错误或不变量检查，如果错误不会回退未消耗的 gas，并且也不会传递错误消息。

```Solidity
function burn(uint _amount) public {
    totalSupply -= _amount;
    assert(totalSupply >= 0);
}
```

<table>
<tr>
    <th>特性</th>
    <th>require</th>
    <th>revert</th>
    <th>assert</th>
</tr>
<tr>
    <td>用途</td>
    <td>验证输入参数或条件是否满足</td>
    <td>手动触发错误，适用于复杂条件</td>
    <td>检查不应该发生的条件，检测严重错误</td>
</tr>
<tr>
    <td>返回未消耗的 gas</td>
    <td>是</td>
    <td>是</td>
    <td>否</td>
</tr>
<tr>
    <td>错误消息</td>
    <td>支持</td>
    <td>支持</td>
    <td>不支持</td>
</tr>
<tr>
    <td>典型场景</td>
    <td>输入验证、权限检查</td>
    <td>复杂条件下的错误处理</td>
    <td>内部错误、不变量检查</td>
</tr>
<tr>
    <td>推荐使用场景</td>
    <td>外部输入验证</td>
    <td>复杂逻辑的错误处理</td>
    <td>调试和检测代码逻辑</td>
</tr>
</table>

### error

Solidity 0.8.4 之后引入了自定义 error 类型，配合错误处理使用，可以显著减少gas 消耗，并且自定义错误可以携带参数，提供了更灵活的错误信息传递机制。

```Solidity
contract Example {
    error NotOwner(address caller);
    error ContractNotActive();
    error InsufficientBalance(uint requested, uint available);

    address public owner;
    bool public isActive = true;
    uint public balance;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        if (msg.sender != owner) {
            revert NotOwner(msg.sender);
        }
        _;
    }

    modifier onlyActive() {
        if (!isActive) {
            revert ContractNotActive();
        }
        _;
    }

    function deposit() public payable onlyActive {
        balance += msg.value;
    }

    function withdraw(uint amount) public onlyOwner onlyActive {
        if (amount > balance) {
            revert InsufficientBalance(amount, balance);
        }
        balance -= amount;
        payable(msg.sender).transfer(amount);
    }
}
```

## 流程控制

### 条件判断

条件判断用于根据特定条件执行不同的代码块，类似于其他编程语言中的 if、else if 和 else。

```Solidity
if (condition) {
    // ...
} else if (anotherCondition) {
    // ...
} else {
    // ...
}
```

### 循环

循环用于重复执行代码块，Solidity 支持三种循环结构：for，while 和 do while，并且也支持 break 和 continue 语句。

**for**

```Solidity
for (uint i = 1; i <= n; i++) {
    total += i;
}
```

**while**

```Solidity
while (i > 0) {
    result *= i;
    i--;
}
```

**do while**

```Solidity
do {
    result *= i;
    i--;
} while (i > 0);
```

<table>
<tr>
    <th>特性</th>
    <th>for</th>
    <th>while</th>
    <th>do while</th>
</tr>
<tr>
    <td>用途</td>
    <td>适用于已知循环次数的场景</td>
    <td>适用于循环次数未知但需要先检查条件的场景</td>
    <td>适用于循环次数未知但至少需要执行一次的场景</td>
</tr>
<tr>
    <td>条件检查</td>
    <td>在每次迭代前检查</td>
    <td>在每次迭代前检查</td>
    <td>在每次迭代后检查</td>
</tr>
<tr>
    <td>执行次数</td>
    <td>可能为零</td>
    <td>可能为零</td>
    <td>至少执行一次</td>
</tr>
<tr>
    <td>灵活性</td>
    <td>适合计数器驱动的循环</td>
    <td>适合基于条件的循环</td>
    <td>适合需要先执行后判断的循环</td>
</tr>
</table>

## 面向对象

Solidity 是一种面向对象的编程语言，支持多种面向对象的特性，包括继承、函数重写、抽象合约、接口和多态。

### 继承

继承是 Solidity 中的重要特性，用于实现代码复用和逻辑扩展。子合约可以继承父合约的状态变量、函数、修饰器和事件，当然可见性为 private 的状态变量、函数无法被继承。Solidity 支持多继承，当多个父类中存在相同签名的函数时，必须显式重写该函数，对于 constructor，receive 和 fallback 函数，子合约不会继承。

* 如果父类的 constructor 有参数传入，子合约必须在自己的 constructor 中传入父合约需要的参数，并在 constructor 后调用父合约的 constructor。

* 由于 receive 和 fallback 函数不会继承，只要子合约自己重新定义即可，但是如果多个父合约中都显式定义了 receive 或 fallback，则需要将父类中的函数定义为 vitrual，并在子合约中重写。

```Solidity
contract A {
    uint public valueA;

    constructor(uint _valueA) {
        valueA = _valueA;
    }

    receive() external payable virtual { }

    fallback() external virtual { }

    function setValue(uint _value) external virtual mustGT100(_value) {
        valueA = _value;
    }

    modifier mustGT100(uint _value) virtual {
        require(_value > 100, "Not GT 100 in A");
        _;
    }
}

contract B {
    uint public valueB;

    constructor(uint _valueB) {
        valueB = _valueB;
    }

    receive() external payable virtual { }

    fallback() external virtual { }

    function setValue(uint _value) external virtual {
        valueB = _value;
    }

    modifier mustLT200(uint _value) virtual {
        require(_value < 200, "Not LT 200");
        _;
    }
}

contract C is A, B {
    uint public valueC;

    constructor(uint _valueA, uint _valueB, uint _valueC) A(_valueA) B(_valueB) {
        valueC = _valueC;
    }

    receive() external payable override (A, B) { }

    fallback() external override (A, B) { }

    function setValue(uint _value) external override (A, B) mustGT100(_value) mustLT200(_value) {
        valueC = _value;
    }

    modifier mustGT100(uint _value) override {
        require(_value > 100, "Not GT 100 in C");
        _;
    }
}

```

### 抽象合约

在 Solidity 中，抽象合约是指至少有一个未实现的函数的合约。这些未实现的函数只声明了函数签名，没有具体的实现逻辑。抽象合约不能被直接部署，必须由继承它的子合约实现所有未实现的函数后才能部署。

```Solidity
abstract contract AbstractContract {
    function greet() public virtual returns (string memory);

    function sayHello() public pure returns (string memory) {
        return "Hello from AbstractContract";
    }
}

contract ConcreteContract is AbstractContract {
    function greet() public override returns (string memory) {
        return "Hello from ConcreteContract";
    }
}
```

### 函数重写

在 Solidity 中子合约可以重写父合约中的函数，前提是父合约的函数被标记为 virtual，子合约的重写函数需要使用 override。

```Solidity
contract A {
    function greet() public virtual pure returns (string memory) {
        return "Hello from A";
    }
}

contract B is A {
    function greet() public override pure returns (string memory) {
        return "Hello from B";
    }
}
```

需要注意的是，private 无法和 virtual 一起用于修饰同一个函数，这很好理解，private 函数对子合约不可见，也就无法继承，更无法被重写了。另外，子合约在重写父合约的函数时，如果父合约的函数可见性修饰符是 external，那么重写的子合约函数可见性修饰符即可以是 public，也可以是 external，但如果父合约函数是 public 或 internal 时，子合约也必须是 public 或 internal。

### 接口

接口是抽象合约的特殊形式，没有状态变量和构造函数，用于定义标准化的函数集合。接口中的函数只能声明，不能实现，其可见性必须是 external。实现接口的合约必须重写接口中的所有函数。

```Solidity
interface IExample {
    function getValue() external view returns (uint);
    function setValue(uint _value) external;
}

contract Example is IExample {
    uint private value;

    function getValue() external view override returns (uint) {
        return value;
    }

    function setValue(uint _value) external override {
        value = _value;
    }
}
```

### 多态

多态允许通过接口或父合约的引用调用子合约的实现。通过接口或父合约类型的变量，可以实现动态绑定。

```Solidity
contract Parent {
    function greet() public virtual pure returns (string memory) {
        return "Hello from Parent";
    }
}

contract Child is Parent {
    function greet() public override pure returns (string memory) {
        return "Hello from Child";
    }
}

contract Test {
    function testGreet(Parent _contract) public pure returns (string memory) {
        return _contract.greet();
    }
}
```

## 内置对象与函数

在 Solidity 中，内置对象和函数是开发智能合约时的重要工具，它们提供了对区块链状态、交易信息以及其他功能的访问。

```Solidity
contract Example {
    function getBlockInfo() public view returns (uint, uint) {
        return (block.number, block.timestamp);
    }

    function getSenderInfo() public payable returns (address, uint) {
        return (msg.sender, msg.value);
    }
}
```

**区块相关**

<table>
<tr>
    <th>属性</th>
    <th>类型</th>
    <th>描述</th>
</tr>
<tr>
    <td>blockhash(uint blockNumber)</td>
    <td>bytes32</td>
    <td>返回指定区块的哈希值（仅适用于最近的 256 个区块）。</td>
</tr>
<tr>
    <td>block.number</td>
    <td>uint</td>
    <td>当前区块的编号。</td>
</tr>
<tr>
    <td>block.timestamp</td>
    <td>uint</td>
    <td>当前区块的时间戳（以秒为单位）。</td>
</tr>
<tr>
    <td>block.difficulty</td>
    <td>uint</td>
    <td>当前区块的难度。</td>
</tr>
<tr>
    <td>block.gaslimit</td>
    <td>uint</td>
    <td>当前区块的 gas 限制。</td>
</tr>
<tr>
    <td>block.coinbase</td>
    <td>address</td>
    <td>当前区块矿工的地址。</td>
</tr>
<tr>
    <td>block.chainid</td>
    <td>uint</td>
    <td>当前区块链的链 ID（Chain ID）。</td>
</tr>
<tr>
    <td>block.basefee</td>
    <td>uint</td>
    <td>当前区块的基础费用（EIP-1559 引入）。</td>
</tr>
</table>

**交易相关**

<table>
<tr>
    <th>属性</th>
    <th>类型</th>
    <th>描述</th>
</tr>
<tr>
    <td>msg.sender</td>
    <td>address</td>
    <td>调用合约的地址（外部账户或合约）。</td>
</tr>
<tr>
    <td>msg.value</td>
    <td>uint</td>
    <td>随交易发送的以太币数量（以 wei 为单位）。</td>
</tr>
<tr>
    <td>msg.data</td>
    <td>bytes</td>
    <td>完整的 calldata（调用数据）。</td>
</tr>
<tr>
    <td>msg.sig</td>
    <td>bytes4</td>
    <td>calldata（调用数据）的前 4 个字节（函数选择器）。</td>
</tr>
</table>

**Gas相关**

<table>
<tr>
    <th>属性</th>
    <th>类型</th>
    <th>描述</th>
</tr>
<tr>
    <td>gasleft()</td>
    <td>uint</td>
    <td>返回当前剩余的 gas。</td>
</tr>
<tr>
    <td>tx.gasprice</td>
    <td>uint</td>
    <td>当前交易的 gas 价格。</td>
</tr>
<tr>
    <td>tx.origin</td>
    <td>address</td>
    <td>发起交易的外部账户地址。</td>
</tr>
</table>

**合约相关**

<table>
<tr>
    <th>属性</th>
    <th>类型</th>
    <th>描述</th>
</tr>
<tr>
    <td>this</td>
    <td>address</td>
    <td>当前合约的地址。</td>
</tr>
<tr>
    <td>selfdestruct(address payable)</td>
    <td></td>
    <td>销毁合约并将剩余余额发送到指定地址。</td>
</tr>
<tr>
    <td>type(C).name</td>
    <td>string</td>
    <td>返回合约 C 的名称。</td>
</tr>
<tr>
    <td>type(C).creationCode</td>
    <td>bytes</td>
    <td>返回合约 C 的部署字节码。</td>
</tr>
<tr>
    <td>type(C).runtimeCode</td>
    <td>bytes</td>
    <td>返回合约 C 的运行时字节码。</td>
</tr>
<tr>
    <td>type(C).interfaceId</td>
    <td>bytes4</td>
    <td>返回合约 C 的接口标识符（仅适用于接口）。</td>
</tr>
</table>

**地址相关**

<table>
<tr>
    <th>属性</th>
    <th>类型</th>
    <th>描述</th>
</tr>
<tr>
    <td>address(contract)</td>
    <td>uint</td>
    <td>获取指定合约的地址，特殊的 address(this)。</td>
</tr>
<tr>
    <td>payable(address)</td>
    <td>uint</td>
    <td>将普通地址转换为可支付地址（payable），以便接收以太币。</td>
</tr>
<tr>
    <td>address.balance</td>
    <td>uint</td>
    <td>返回地址的以太币余额（以 wei 为单位）。</td>
</tr>
<tr>
    <td>address.transfer(uint amount)</td>
    <td></td>
    <td>向地址发送指定数量的以太币（以 wei 为单位），失败时抛出异常。</td>
</tr>
<tr>
    <td>address.send(uint amount)</td>
    <td>bool</td>
    <td>向地址发送指定数量的以太币（以 wei 为单位），失败时返回 false。</td>
</tr>
<tr>
    <td>address.call(bytes memory data)</td>
    <td>(bool, bytes memory)</td>
    <td>低级调用指定地址的函数，返回调用是否成功及返回数据。</td>
</tr>
<tr>
    <td>address.delegatecall(bytes memory data)</td>
    <td>(bool, bytes memory)</td>
    <td>低级调用指定地址的函数，使用调用者的上下文。</td>
</tr>
<tr>
    <td>address.staticcall(bytes memory data)</td>
    <td>(bool, bytes memory)</td>
    <td>低级静态调用指定地址的函数，不允许状态修改。</td>
</tr>
<tr>
    <td>address.code</td>
    <td>bytes</td>
    <td>返回地址的合约代码。</td>
</tr>
<tr>
    <td>address.codehash</td>
    <td>bytes32</td>
    <td>返回地址的合约代码哈希。</td>
</tr>
<tr>
    <td>address(uint160(uint256(keccak256(abi.encodePacked(...)))))
</td>
    <td>bytes32</td>
    <td>通过hash生成地址。</td>
</tr>
</table>

**数学与哈希**

<table>
<tr>
    <th>函数</th>
    <th>类型</th>
    <th>描述</th>
</tr>
<tr>
    <td>addmod(uint x, uint y, uint k)</td>
    <td>uint</td>
    <td>计算 (x + y) % k，防止溢出。</td>
</tr>
<tr>
    <td>mulmod(uint x, uint y, uint k)</td>
    <td>uint</td>
    <td>计算 (x * y) % k，防止溢出。</td>
</tr>
<tr>
    <td>keccak256(bytes memory data)</td>
    <td>bytes32</td>
    <td>计算输入数据的 Keccak-256 哈希值。</td>
</tr>
<tr>
    <td>sha256(bytes memory data)</td>
    <td>bytes32</td>
    <td>计算输入数据的 SHA-256 哈希值。</td>
</tr>
<tr>
    <td>ripemd160(bytes memory data)</td>
    <td>bytes20</td>
    <td>计算输入数据的 RIPEMD-160 哈希值。</td>
</tr>
<tr>
    <td>ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s)</td>
    <td>address</td>
    <td>从签名中恢复公钥地址。</td>
</tr>
</table>

**ABI编码与解码**

<table>
<tr>
    <th>函数</th>
    <th>类型</th>
    <th>描述</th>
</tr>
<tr>
    <td>abi.encode(...)</td>
    <td>bytes</td>
    <td>将参数编码为字节数组，适用于函数调用。</td>
</tr>
<tr>
    <td>abi.encodePacked(...)</td>
    <td>bytes</td>
    <td>将参数紧凑编码为字节数组，适用于哈希计算。</td>
</tr>
<tr>
    <td>abi.encodeWithSelector(bytes4 selector, ...)</td>
    <td>bytes</td>
    <td>将函数选择器和参数编码为字节数组。</td>
</tr>
<tr>
    <td>abi.encodeWithSignature(string memory signature, ...)</td>
    <td>bytes</td>
    <td>将函数签名和参数编码为字节数组。</td>
</tr>
<tr>
    <td>abi.decode(bytes memory data, ...)</td>
    <td>动态类型（根据解码的类型参数决定）</td>
    <td>将字节数组解码为指定类型的参数。</td>
</tr>
</table>

**其他**

<table>
<tr>
    <th>函数</th>
    <th>类型</th>
    <th>描述</th>
</tr>
<tr>
    <td>delete</td>
    <td></td>
    <td>将变量重置为默认值。如果是array清空数组，如果是mapping将键值对的值重置为默认值，如果是struct将结构体所有字段重置为默认值。</td>
</tr>
</table>

## 日志

在 Solidity 中，日志是通过 事件（events） 实现的。事件允许智能合约将数据记录到区块链的日志中，供外部系统（如 DApps 或区块链浏览器）查询和监听。日志不会存储在合约的存储中，因此不会增加合约的存储成本。

事件使用 event 关键字定义，使用 emit 关键字触发事件。最多可以为事件的三个参数添加 indexed 关键字，搜索时允许通过这些参数进行过滤。

```Solidity
contract Example {
    event Transfer(address indexed from, address indexed to, uint256 value);
    event LogMessage(string message);

    function transfer(address to, uint256 amount) public {
        emit Transfer(msg.sender, to, amount);
    }

    function log(string memory message) public {
        emit LogMessage(message);
    }
}
```

事件在区块链中的存储结构由两部分组成：

* topics：存储 indexed 参数和事件签名的哈希值。
* data：存储非 indexed 参数。

每增加一个 indexed 参数，触发事件时就会增加额外的 gas 消耗，但也比存储数据到合约中便宜。
