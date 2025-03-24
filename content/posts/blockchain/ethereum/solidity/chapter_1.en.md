---
layout: post
title: "Solidity Basics"
description: "Introduce the basic syntax and precautions of Solidity."
date: 2025-03-21T13:15:41+08:00
image: "/posts/blockchain/ethereum/solidity/images/chapter_1-cover.jpg"
tags: ["Blockchain", "Solidity"]
---

## Data Types

Solidity offers a variety of data types to meet the different needs of smart contract development. The following are some common data types:

<table>
<tr>
    <th>Category</th>
    <th>Data Type</th>
</tr>
<tr>
    <td>Value Type</td>
    <td><code>bool</code>, <code>int[8-256]</code>, <code>uint[8-256]</code>, <code>address</code>, <code>bytes[1-32]</code></td>
</tr>
<tr>
    <td>Reference Type</td>
    <td><code>string</code>, <code>bytes</code>, <code>array</code>, <code>mapping</code></td>
</tr>
<tr>
    <td>Special Type</td>
    <td><code>enum</code>, <code>struct</code></td>
</tr>
</table>

**bytes[1-32], bytes and string**

`bytes[1-32]` is a fixed-length byte array with a length range of 1 to 32 bytes. `bytes` is a dynamic byte array, while `string`, although essentially also a dynamic byte array, stores UTF-8 encoded character sequences and involves implicit UTF-8 encoding conversion operations. Therefore, if you need to store fixed-length byte data, such as hash values or identifiers, you can use `bytes[1-32]`. For dynamic-sized byte data, such as file content or binary data, use `bytes`. If it is text content, such as names or descriptions, use `string`. Additionally, only `bytes` has a `length` attribute; to obtain the length of a `string`, it needs to be converted to `bytes`.

```Solidity
bytes32 public hash = keccak256(abi.encodePacked("data"));


bytes public data = "Hello..." ;
uint256 len = data.length;


string public text = "Hello, Solidity!" ;
uint256 len = bytes(text).length;
```

**array**

An array can be classified as a fixed array or a dynamic data structure, depending on whether its length is specified at the time of declaration. Dynamic arrays can use push and pop to add or remove elements at the end of the array, but it is not possible to directly delete a specific element in the middle. A more convenient approach is to swap the element to be deleted with the last element and then remove the last element.

```Solidity
uint[3] fixedArray = [1, 2, 3];
fixedArray[0] = 10;


uint[] dynamicArray;
dynamicArray.push(1);
dynamicArray.push(2);
dynamicArray.pop();
```

**mapping**

Mapping is a hash table in Solidity that supports read, write, and delete operations. However, the delete operation does not truly remove the key-value pair but resets the key-value to the default value using the delete function. In fact, the delete function also supports resetting other data types. Mapping does not support checking whether a key-value exists. Generally, it is determined by checking whether the value is the default value.

```solidity
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

Mapping cannot use struct, array, or mapping as keys, but they can be nested.

```solidity
mapping(address => mapping(uint => bool)) public nestedMapping;

function setNestedValue(address _user, uint _key, bool _value) public {
    nestedMapping[_user][_key] = _value;
}

function getNestedValue(address _user, uint _key) public view returns (bool) {
    return nestedMapping[_user][_key];
}
```

**enum**

The `enum` is used to define a set of named constants, helping developers express the meanings of constants more clearly and improving the readability and maintainability of the code. The underlying type of `enum` is `uint8`, and the default value is 0, which is the first defined constant.

```solidity
enum Status { Pending, Approved, Rejected } // 0, 1, 2
Status public currentStatus; // default value: 0

function getStatusIndex() public view returns (uint) {
    return uint(currentStatus); // convert enum to uint
}
```

**struct**

A struct can contain fields of various types, allowing users to define complex data structures and supporting nested definitions.

```solidity
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

### Type Conversion

Solidity supports both explicit and implicit type conversions, but it is important to note that some conversions may result in data loss or errors.

**Integer Types**

Conversions between value types usually require explicit declaration, especially when the range of the target type is smaller than that of the source type. If the range of the target type is smaller than that of the source type, truncation may occur. If an unsigned integer is converted to a signed integer and the value exceeds the range of the target type, an overflow may occur.

```Solidity
uint256 a = 100;
int256 b = int256(a); // Convert uint256 to int256
uint8 c = uint8(a);   // Convert uint256 to uint8 (data may be truncated)
```

**Address Type**

The address can be explicitly converted to address payable to receive Ether.

```Solidity
address addr = 0x1234567890123456789012345678901234567890;
address payable payableAddr = payable(addr); // Convert address to address payable
```

**Byte Type**

A smaller byte array can be expanded to a larger byte array through explicit conversion.

```Solidity
bytes32 b32 = bytes32("hello"); // Convert a string to bytes32 
bytes1 b1 = bytes1(b32);        // Truncate to bytes1
```

Bytes can be explicitly converted to strings and vice versa.

```Solidity
bytes memory b = bytes("hello");
string memory s = string(b);
```

**Boolean Type**

Boolean types cannot be directly converted to or from other types. Conversion can only be achieved through logical judgments.

```Solidity
uint256 a = 1;
bool isTrue = (a > 0); // Convert to a boolean value through logical judgment
```

**Enumeration Type**

An enumeration type can be explicitly converted to a uint, and vice versa. During the conversion, the value of the enumeration must be within the range defined by its definition.

```Solidity
enum Status { Pending, Approved, Rejected }
Status status = Status.Pending;


uint statusIndex = uint(status); // Convert enumeration to 
uint status = Status(statusIndex);    // Convert uint to enumeration
```

**Array Type**

Fixed-size arrays and dynamic arrays cannot be directly converted. Elements need to be manually copied.

```Solidity
uint[3] memory fixedArray = [1, 2, 3];
uint[] memory dynamicArray = new uint[](3);


for (uint i = 0; i < fixedArray.length; i++) {
    dynamicArray[i] = fixedArray[i];
}
```

**Function Types**

Function types can be explicitly converted to compatible function types. Internal functions and external functions cannot be directly converted between each other.

```Solidity
function add(uint a, uint b) public pure returns (uint) {
    return a + b;
}


function(uint, uint) pure returns (uint) func = add; // Function type conversion
```

**Complex Type Conversion**

Use abi.encode and abi.decode for the conversion of complex types.

```Solidity
bytes memory encoded = abi.encode(1, "hello");
(uint x, string memory y) = abi.decode(encoded, (uint, string));
```

### Visibility

There are four visibility rules in Solidity. Among them, public, private, and internal are applicable to both state variables and functions, while external can only be used for functions. The default visibility for state variables and functions is internal.

<table>
<tr>
<th>Visibility</th>
<th>Description</th> </tr>
<tr>
    <td>public</td>
    <td>It can be accessed from within the contract, from outside the contract, and from inherited contracts. </td>
</tr>
<tr>
    <td>private</td>
    <td>Can only be accessed from within the contract; neither inherited contracts nor external entities can access it. </td>
</tr>
<tr>
    <td>internal</td>
    <td>Can only be accessed from within the contract and inherited contracts; external access is not allowed. </td>
</tr>
<tr>
    <td>external</td>
    <td>It can only be accessed from outside the contract. To be precise, it cannot be directly accessed. Within the contract, it needs to be called using the "this" keyword. </td>
</tr>
</table>

### EVM Storage Mode

In Solidity, when passing parameters to a function, `calldata` and `memory` can be used to store the parameters in the specified EVM storage area. However, this is only applicable to reference types (`string`, `bytes`, `array`, `mapping`) and `struct`. Other types of parameters are stored in the EVM stack by default. `calldata` allocates data to a read-only storage area and cannot be modified, while `memory` allocates data to the memory and can be modified. Additionally, when reference types and structs are used as function return values, they can only be marked with `memory`, because reference types and structs as return values can only be stored in memory.

```solidity
function processArray(uint[] calldata _input) external pure returns (uint) {
    return _input[0];
}

function modifyArray(uint[] memory _input) public pure returns (uint[] memory) {
    _input[0] = 42;
    return _input;
}
```

The EVM itself has five types of storage areas and also supports permanently recording contract events in the blockchain log.

<table>
<tr>
    <th>Storage Mode</th>
    <th>Description</th>
</tr>
<tr>
    <td>Storage</td>
    <td>Data permanently stored on the chain, with a lifecycle consistent with the contract, has relatively high read and write costs and is the default type for contract state variables.</td>
</tr>
<tr>
    <td>Calldata</td>
    <td>Read-only data storage area, used for storing parameters of external function calls, with relatively low cost.</td>
</tr>
<tr>
    <td>Memory</td>
    <td>Data temporarily stored in memory is only valid during the function call and has a relatively low cost for reading and writing.</td>
</tr>
<tr>
    <td>Stack</td>
    <td>It is used to store temporary variables during function execution, with limited capacity (1024 slots, each slot being 32 bytes in size), and offers the fastest access speed.</td>
</tr>
<tr>
    <td>Code</td>
    <td>The read-only storage area stores the bytecode of the contract and is mainly used to execute the contract logic.</td>
</tr>
<tr>
    <td>Logs</td>
    <td>The event log storage area is used to record on-chain events for external systems to query.</td>
</tr>
</table>

### Constant

In Solidity, state variables can be marked as constants with the `constant` modifier. State variables marked as constants must be assigned a value immediately. Constants are not stored in the Storage area but are directly compiled by the compiler into the contract's bytecode, or it can be said that they are stored in the Code area.

```Solidity
uint256 constant MAX_SUPPLY = 10000;
address constant OWNER = 0x1234567890abcdef1234567890abcdef12345678;
```

In addition, state variables can also be marked as immutable by the `immutable` modifier. Unlike constants, immutable variables do not need to be assigned a value immediately. They can be initialized through the constructor when the contract is deployed and cannot be changed after deployment. The contract creation code generated by the compiler will replace the immutable variables in the runtime code with specific values before returning, so immutable variables will not be stored in the Storage area either.

```Solidity
uint256 immutable deploymentTime;

constructor() {
    deploymentTime = block.timestamp;
}
```

## Function

Functions are the core of smart contracts in Solidity, used to define the behavior and logic of the contract. Functions can have multiple parameters. For reference types (`string`, `bytes`, `array`, `mapping`) and `struct` types of parameters, the storage mode (`calldata`, `memory`) must be specified. At the same time, functions can also have multiple return values. If the return value is a reference type or `struct` type, it must be specified as `memory`, because reference types and structs as return values can only be stored in memory. Solidity functions support overloading. Multiple functions with the same name but different parameters can coexist.

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

The Solidity official documentation suggests that when a contract contains multiple functions, they should be organized in the following order:

* constructor
* receive function (if exists)
* fallback function (if exists)
* external
* public
* internal
* private

### Function Modifiers

Solidity functions have multiple modifiers, and their declaration order is visibility, mutability, virtual function, override, and custom modifiers.

<table>
<tr>
    <th>Features</th>
    <th>Description</th>
</tr>
<tr>
    <td>Visibility</td>
    <td>public, private, internal, external. The default is internal.</td>
</tr>
<tr>
    <td>Modifiability</td>
    <td>view, pure, payable. The default is nonpayable (nonpayable cannot be explicitly defined, just like state variables cannot be explicitly defined as storage). Functions can directly read and write state variables. When specified as view, only state variables can be read. When specified as pure, state variables cannot be directly read or written, but can be indirectly read by calling view functions. When specified as payable, not only can state variables be read and written, but also Ether transactions can be conducted.</td>
</tr>
<tr>
    <td>Virtual Function</td>
    <td>Mark a function as virtual using the "virtual" keyword, allowing it to be overridden by derived contracts. "Virtual" cannot be used with the "private" modifier. For detailed explanations, refer to the object-oriented section.</td>
</tr>
<tr>
    <td>Overriding</td>
    <td>Use the `override` keyword to indicate that a function is overriding a function of the same name in the parent contract. Overriding is only possible if the function in the parent contract is virtual. For detailed explanations, see the object-oriented programming section.</td>
</tr>
<tr>
    <td>Custom modifiers</td>
    <td>Custom modifiers are often used to check the execution permissions and conditions of functions, improving code reusability and readability. Multiple custom modifiers can be used simultaneously and will be called in the order they are declared.</td>
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

### Constructor

A constructor is a special function whose main role is to set the initial state of a contract, such as initializing state variables or setting the contract's owner. It is executed only once when the contract is deployed and cannot be called again. The visibility of a constructor can only be public or internal, with internal being the default. Moreover, a constructor is optional; if no initialization operations are needed, it can be omitted.

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

### Receive and Fallback

`receive` and `fallback` are two special functions used to handle the reception of Ether and the invocation of unknown functions. Firstly, both must be declared as `external`, and `receive` must be declared as `payable`, while whether `fallback` is declared as `payable` is optional.

The `receive` function is specifically designed to receive Ether. It is triggered when the contract receives Ether without any function being called or any calldata being passed.

```Solidity
contract Example {
    event Received(address sender, uint amount);

    receive() external payable {
        emit Received(msg.sender, msg.value);
    }
}

// Example usage
address(example).transfer(1 ether);
```

The fallback function is used to handle unknown function calls. If the contract does not define a receive function and is marked as payable, it will also handle the situation of receiving Ether.

```Solidity
contract Example {
    event FallbackCalled(address sender, uint value, bytes data);

    fallback() external payable {
        emit FallbackCalled(msg.sender, msg.value, msg.data);
    }
}

// Example usage
(bool success, ) = address(example).call{value: 1 ether}("nonexistentFunction()");
```

<table>
<tr>
    <th>Features</th>
    <th>receive</th>
    <th>fallback</th>
</tr>
<tr>
    <td>Purpose</td>
    <td>Specifically used for receiving Ether</td>
    <td>Handles unknown function calls and can also receive Ether</td>
</tr>
<tr>
    <td>Declaration</td>
    <td>Must be declared as external and payable</td>
    <td>Must be declared as external, payable is optional</td>
</tr>
<tr>
    <td>Triggering condition</td>
    <td>The contract receives Ether and has no calldata</td>
    <td>Calling an unknown function or receiving Ether without defining a receive function (declared as payable)</td>
</tr>
<tr>
    <td>Priority</td>
    <td>Higher than fallback</td>
    <td>Lower than receive</td>
</tr>
<tr>
    <td>Return Value</td>
    <td>None</td>
    <td>None</td>
</tr>
</table>

## Error Handling

Error handling is an important part of ensuring the security and reliability of smart contracts. Solidity provides three main error handling methods: `require`, `revert`, and `assert`.

### require, revert and assert

**require**

It is used to verify whether the input parameters or conditions are met. If not, an error will be thrown and the transaction will be rolled back. The unused gas will be refunded.

```solidity
function setOwner(address _newOwner) public {
    require(msg.sender == owner, "Only the owner can set a new owner");
    owner = _newOwner;
}
```

**revert**

It is used to manually trigger an error and roll back the transaction. The unused gas will be refunded. It is typically used for error handling under complex conditions.

```solidity
function withdraw(uint _amount) public {
    if (_amount > balance) {
        revert("Insufficient balance");
    }
    balance -= _amount;
}
```

**assert**

Used to check conditions that should not occur, detect serious errors in code logic, usually internal errors or invariant checks. If an error occurs, it will not refund the unused gas and will not pass an error message.

```solidity
function burn(uint _amount) public {
    totalSupply -= _amount;
    assert(totalSupply >= 0);
}
```

<table>
<tr>
    <th>Features</th>
    <th>require</th>
    <th>revert</th>
    <th>assert</th>
</tr>
<tr>
    <td>Purpose</td>
    <td>Verify whether input parameters or conditions are met</td>
    <td>Manually trigger errors, suitable for complex conditions</td>
    <td>Check for conditions that should not occur, detect serious errors</td>
</tr>
<tr>
    <td>Return unused gas</td>
    <td>Yes</td>
    <td>Yes</td>
    <td>No</td>
</tr>
<tr>
    <td>Error message</td>
    <td>Supported</td>
    <td>Supported</td>
    <td>Not supported</td>
</tr>
<tr>
    <td>Typical scenarios</td>
    <td>Input validation, permission checks</td>
    <td>Error handling under complex conditions</td>
    <td>Internal errors, invariant checks</td>
</tr>
<tr>
    <td>Recommended usage scenarios</td>
    <td>External input validation</td>
    <td>Error handling for complex logic</td>
    <td>Debugging and testing code logic</td>
</tr>
</table>

### error

Solidity 0.8.4 introduced custom error types, which, when used in conjunction with error handling, can significantly reduce gas consumption. Moreover, custom errors can carry parameters, providing a more flexible mechanism for error information transmission.

```solidity
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

## Process Control

### Conditional Judgment

Conditional judgment is used to execute different code blocks based on specific conditions, similar to `if`, `else if`, and `else` in other programming languages.

```solidity
if (condition) {
    // ...
} else if (anotherCondition) {
    // ...
} else {
    // ...
}
```

### Loop

Loops are used to repeatedly execute code blocks. Solidity supports three types of loop structures: `for`, `while`, and `do while`, and also supports the `break` and `continue` statements.

**for**

```solidity
for (uint i = 1; i <= n; i++) {
    total += i;
}
```

**while**

```solidity
while (i > 0) {
    result *= i;
    i--;
}
```

**do while**

```solidity
do {
    result *= i;
    i--;
} while (i > 0);
```

<table>
<tr>
    <th>Features</th>
    <th>for</th>
    <th>while</th>
    <th>do while</th>
</tr>
<tr>
    <td>Purpose</td>
    <td>Applicable to scenarios where the number of loops is known</td>
    <td>Applicable to scenarios where the number of loops is unknown but a condition needs to be checked first</td>
    <td>Applicable to scenarios where the number of loops is unknown but at least one execution is required</td>
</tr>
<tr>
    <td>Condition Check</td>
    <td>Check before each iteration</td>
    <td>Check before each iteration</td>
    <td>Check after each iteration</td>
</tr>
<tr>
    <td>Execution times</td>
    <td>May be zero</td>
    <td>May be zero</td>
    <td>At least once</td>
</tr>
<tr>
    <td>Flexibility</td>
    <td>Suitable for counter-driven loops</td>
    <td>Suitable for condition-based loops</td>
    <td>Suitable for loops that require execution before evaluation</td>
</tr>
</table>

## Object-Oriented

Solidity is an object-oriented programming language that supports various object-oriented features, including inheritance, function overriding, abstract contracts, interfaces, and polymorphism.

### Inheritance

Inheritance is an important feature in Solidity, used to achieve code reuse and logic expansion. Child contracts can inherit the state variables, functions, modifiers, and events of parent contracts. However, state variables and functions with private visibility cannot be inherited. Solidity supports multiple inheritance. When multiple parent classes have functions with the same signature, the function must be explicitly overridden. For constructors, receive, and fallback functions, child contracts do not inherit them.

If the constructor of the parent class has parameters, the child contract must pass the required parameters of the parent contract in its own constructor and call the constructor of the parent contract after its own constructor.

Since the `receive` and `fallback` functions are not inherited, the child contract can redefine them. However, if multiple parent contracts explicitly define `receive` or `fallback`, the function definitions in the parent classes need to be marked as `virtual`, and then overridden in the child contract.

```solidity
contract A {
    uint public valueA;

    constructor(uint _valueA) {
        valueA = _valueA;
    }

    receive() external payable virtual {}

    fallback() external virtual {}

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

    receive() external payable virtual {}

    fallback() external virtual {}

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

    receive() external payable override(A, B) {}

    fallback() external override(A, B) {}

    function setValue(uint _value) external override(A, B) mustGT100(_value) mustLT200(_value) {
        valueC = _value;
    }

    modifier mustGT100(uint _value) override {
        require(_value > 100, "Not GT 100 in C");
        _;
    }
}
```

### Abstract Contract

In Solidity, an abstract contract refers to a contract that has at least one unimplemented function. These unimplemented functions only declare the function signature without specific implementation logic. Abstract contracts cannot be directly deployed; they must be inherited by a child contract that implements all the unimplemented functions before they can be deployed.

```solidity
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

### Function Overriding

In Solidity, a child contract can override a function in its parent contract, provided that the function in the parent contract is marked as `virtual`, and the overriding function in the child contract needs to use the `override` keyword.

```solidity
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

It should be noted that `private` cannot be used together with `virtual` to modify the same function. This is because `private` functions are not visible to child contracts and thus cannot be inherited or overridden. Additionally, when a child contract overrides a function of the parent contract, if the visibility modifier of the parent contract's function is `external`, the visibility modifier of the overridden function in the child contract can be either `public` or `external`. However, if the parent contract's function is `public` or `internal`, the child contract's function must also be `public` or `internal`.

### Interface

An interface is a special form of an abstract contract, without state variables or constructors, used to define a standardized set of functions. Functions in an interface can only be declared, not implemented, and their visibility must be `external`. Contracts that implement an interface must override all functions in the interface.

```solidity
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

### Polymorphism

Polymorphism allows the implementation of a child contract to be called through a reference to an interface or a parent contract. Dynamic binding can be achieved through variables of interface or parent contract types.

```solidity
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

## Built-in Objects and Functions

In Solidity, built-in objects and functions are important tools for developing smart contracts, providing access to blockchain state, transaction information, and other functionalities.

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

**Block-related**

<table>
<tr>
<th>Property</th>
<th>Type</th>
<th>Description</th>
</tr>
<tr>
<td>blockhash(uint blockNumber)</td>
<td>bytes32</td>
<td>Return the hash value of the specified block (only applicable to the most recent 256 blocks).</td>
</tr>
<tr>
<td>block.number</td>
<td>uint</td>
<td>The number of the current block.</td>
</tr>
<tr>
<td>block.timestamp</td>
<td>uint</td>
<td>The timestamp of the current block (in seconds).</td>
</tr>
<tr>
<td>block.difficulty</td>
<td>uint</td>
<td>The difficulty of the current block.</td>
</tr>
<tr>
<td>block.gaslimit</td>
<td>uint</td>
<td>The gas limit of the current block.</td>
</tr>
<tr>
<td>block.coinbase</td>
<td>address</td>
<td>The address of the current block miner.</td>
</tr>
<tr>
<td>block.chainid</td>
<td>uint</td>
<td>The current blockchain's chain ID (Chain ID).</td>
</tr>
<tr>
<td>block.basefee</td>
<td>uint</td>
<td>The base fee of the current block (introduced by EIP-1559).</td>
</tr>
</table>

**Transaction-related**

<table>
<tr>
<th>Property</th>
<th>Type</th>
<th>Description</th>
</tr>
<tr>
<td>msg.sender</td>
<td>address</td>
<td>The address of the contract being called (an external account or a contract).</td>
</tr>
<tr>
<td>msg.value</td>
<td>uint</td>
<td>The amount of Ether sent with the transaction (in wei).</td>
</tr>
<tr>
<td>msg.data</td>
<td>bytes</td>
<td>The complete calldata (call data).</td>
</tr>
<tr>
<td>msg.sig</td>
<td>bytes4</td>
<td>The first 4 bytes (function selector) of calldata (call data).</td>
</tr>
</table>

**Gas-related**

<table>
<tr>
<th>Property</th>
<th>Type</th>
<th>Description</th>
</tr>
<tr>
<td>gasleft()</td>
<td>uint</td>
<td>Return the remaining gas of the current transaction.</td>
</tr>
<tr>
<td>tx.gasprice</td>
<td>uint</td>
<td>The gas price of the current transaction.</td>
</tr>
<tr>
<td>tx.origin</td>
<td>address</td>
<td>The external account address that initiated the transaction.</td>
</tr>
</table>

**Contract-related**

<table>
<tr>
<th>Property</th>
<th>Type</th>
<th>Description</th>
</tr>
<tr>
<td>this</td>
<td>address</td>
<td>The address of the current contract.</td>
</tr>
<tr>
<td>selfdestruct(address payable)</td>
<td></td>
<td>Destroy the contract and send the remaining balance to the specified address.</td>
</tr>
<tr>
<td>type(C).name</td>
<td>string</td>
<td>Return the name of contract C.</td>
</tr>
<tr>
<td>type(C).creationCode</td>
<td>bytes</td>
<td>Return the deployment bytecode of contract C.</td>
</tr>
<tr>
<td>type(C).runtimeCode</td>
<td>bytes</td>
<td>Return the runtime bytecode of contract C.</td>
</tr>
<tr>
<td>type(C).interfaceId</td>
<td>bytes4</td>
<td>Return the interface identifier of contract C (only applicable to interfaces).</td>
</tr>
</table>

**Address-related**

<table>
<tr>
<th>Property</th>
<th>Type</th>
<th>Description</th>
</tr>
<tr>
<td>address(contract)</td>
<td>uint</td>
<td>Get the address of the specified contract, the special address(this).</td>
</tr>
<tr>
<td>payable(address)</td>
<td>uint</td>
<td>Convert a regular address to a payable address to receive Ether.</td>
</tr>
<tr>
<td>address.balance</td>
<td>uint</td>
<td>Return the Ether balance of the address (in wei).</td>
</tr>
<tr>
<td>address.transfer(uint amount)</td>
<td></td>
<td>Send the specified amount of Ether (in wei) to the address, and throw an exception if it fails.</td>
</tr>
<tr>
<td>address.send(uint amount)</td>
<td>bool</td>
<td>Send a specified amount of Ether (in wei) to the address, and return false if the operation fails.</td>
</tr>
<tr>
<td>address.call(bytes memory data)</td>
<td>(bool, bytes memory)</td>
<td>Call a function at the specified address at the low level and return whether the call was successful and the returned data.</td>
</tr>
<tr>
<td>address.delegatecall(bytes memory data)</td>
<td>(bool, bytes memory)</td>
<td>Call the function at the specified address at the low level, using the context of the caller.</td>
</tr>
<tr>
<td>address.staticcall(bytes memory data)</td>
<td>(bool, bytes memory)</td>
<td>A low-level static call to a function at a specified address is made, and state modification is not allowed.</td>
</tr>
<tr>
<td>address.code</td>
<td>bytes</td>
<td>The contract code for the return address.</td>
</tr>
<tr>
<td>address.codehash</td>
<td>bytes32</td>
<td>The hash of the contract code for the return address.</td>
</tr>
<tr>
<td>address(uint160(uint256(keccak256(abi.encodePacked(...)))))</td>
<td>bytes32</td>
<td>Generate the address through hash.</td>
</tr>
</table>

**Mathematics and Hashing**

<table>
<tr>
<th>Function</th>
<th>Type</th>
<th>Description</th>
</tr>
<tr>
<td>addmod(uint x, uint y, uint k)</td>
<td>uint</td>
<td>Calculate (x + y) % k to prevent overflow.</td>
</tr>
<tr>
<td>mulmod(uint x, uint y, uint k)</td>
<td>uint</td>
<td>Calculate (x * y) % k to prevent overflow.</td>
</tr>
<tr>
<td>keccak256(bytes memory data)</td>
<td>bytes32</td>
<td>Calculate the Keccak-256 hash value of the input data.</td>
</tr>
<tr>
<td>sha256(bytes memory data)</td>
<td>bytes32</td>
<td>Calculate the SHA-256 hash value of the input data.</td>
</tr>
<tr>
<td>ripemd160(bytes memory data)</td>
<td>bytes20</td>
<td>Calculate the RIPEMD-160 hash value of the input data.</td>
</tr>
<tr>
<td>ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s)</td>
<td>address</td>
<td>Recover the public key address from the signature.</td>
</tr>
</table>

**ABI Encoding and Decoding**

<table>
<tr>
<th>Function</th>
<th>Type</th>
<th>Description</th>
</tr>
<tr>
<td>abi.encode(...)</td>
<td>bytes</td>
<td>Encode the parameters as a byte array, suitable for function calls.</td>
</tr>
<tr>
<td>abi.encodePacked(...)</td>
<td>bytes</td>
<td>Compactly encode the parameters as a byte array for hash calculation.</td>
</tr>
<tr>
<td>abi.encodeWithSelector(bytes4 selector, ...)</td>
<td>bytes</td>
<td>Encode the function selector and parameters as a byte array.</td>
</tr>
<tr>
<td>abi.encodeWithSignature(string memory signature, ...)</td>
<td>bytes</td>
<td>Encode the function signature and parameters as a byte array.</td>
</tr>
<tr>
<td>abi.decode(bytes memory data, ...)</td>
<td>Dynamic type (determined by the type parameter of the decoded data)</td>
<td>Decode the byte array into a parameter of the specified type.</td>
</tr>
</table>

**Others**

<table>
<tr>
<th>Function</th>
<th>Type</th>
<th>Description</th>
</tr>
<tr>
<td>delete</td>
<td></td>
<td>Reset the variable to its default value. If it is an array, clear the array; if it is a mapping, reset the value of the key-value pair to the default value; if it is a struct, reset all fields of the struct to the default value.</td>
</tr>
</table>

## Log

In Solidity, logs are implemented through events. Events allow smart contracts to record data to the blockchain's logs, which can be queried and listened to by external systems such as DApps or blockchain browsers. Logs are not stored in the contract's storage, thus not increasing the contract's storage cost.

Events are defined using the `event` keyword and triggered using the `emit` keyword. Up to three parameters of an event can be marked with the `indexed` keyword, allowing filtering by these parameters during search.

```solidity
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

The storage structure of events in the blockchain consists of two parts:

<table>
<tr>
    <th>Component</th>
    <th>Description</th>
</tr>
<tr>
    <td>topics</td>
    <td>Stores the hash values of indexed parameters and event signatures.</td>
</tr>
<tr>
    <td>data</td>
    <td>Stores non-indexed parameters.</td>
</tr>
</table>

Each additional indexed parameter will increase the gas consumption when the event is triggered, but it is still cheaper than storing data in the contract.
