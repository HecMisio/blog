---
layout: post
title: "探索 Foundry Cheatcodes"
description: "列举和介绍常用的 Foundry Cheatcodes，以便更高效地编写测试和调试智能合约"
date: 2025-03-27T16:28:27+08:00
image: "/posts/blockchain/ethereum/solidity/images/chapter_3-cover.jpg"
tags: ["Blockchain", "Solidity"]
---

Foundry 是以太坊智能合约开发者的强大工具，而其中的 Cheatcodes 功能更是为开发和测试提供了极大的便利。无论是模拟区块链环境、操控合约状态，还是快速调试复杂逻辑，Cheatcodes 都能帮助开发者事半功倍。在本文中，我们将探索 Foundry 提供的各种 Cheatcodes，逐一讲解它们的用途和使用方法，助力你更高效地编写和调试 Solidity 智能合约。

## 环境类

### warp，roll，fee，chainId

warp 用于设置区块时间戳，roll 用于设置区块数量，fee 用于设置区块链的 basefee，chainId 用于设置区块链 ID。

```solidity
vm.warp(1641070800);
emit log_uint(block.timestamp); // 1641070800

vm.roll(100);
emit log_uint(block.number); // 100

vm.fee(25 gwei);
emit log_uint(block.basefee); // 25000000000

vm.chainId(31337);
emit log_uint(block.chainid); // 31337
```

### getBlockTimestamp，getBlockNumber

getBlockTimestamp 用于获取当前的区块时间戳，getBlockNumber 用于获取当前的区块数量。

```solidity
assertEq(vm.getBlockTimestamp(), 1, "timestamp should be 1");
vm.warp(10);
assertEq(vm.getBlockTimestamp(), 10, "warp failed");

uint256 height = vm.getBlockNumber();
assertEq(height, uint256(block.number));
vm.roll(10);
assertEq(vm.getBlockNumber(), 10);
```

### store，load

store 用于设置某个账户的 storage，通过 slot 来索引；load 用于从某个账户的 storage 获取数据，通过 slot 来索引。

```solidity
/// contract LeetContract {
///     uint256 private leet = 1337; // slot 0
/// }

bytes32 leet = vm.load(address(leetContract), bytes32(uint256(0)));
emit log_uint(uint256(leet)); // 1337

vm.store(address(leetContract), bytes32(uint256(0)), bytes32(uint256(31337)));
bytes32 leet = vm.load(address(leetContract), bytes32(uint256(0)));
emit log_uint(uint256(leet)); // 31337
```

### deal

deal 用于设置某个地址的账户余额。如果使用 StdCheats.sol 中的替代签名，还可以指定 ERC20 token 地址，直接设置某个地址的 token 余额，这种调整不会更改 totalSupply。

```solidity
address alice = makeAddr("alice");
emit log_address(alice);
vm.deal(alice, 1 ether);
log_uint256(alice.balance); // 1000000000000000000

address alice = makeAddr("alice");
emit log_address(alice);
deal(address(DAI), alice, 1 ether); // import StdUtils.sol first
log_uint256(address(DAI).balanceOf(alice)); // 1000000000000000000
```

### prank，startPrank，stopPrank

prank 用于为下一次 call 设置 msg.sender，支持 staticcall，但默认不支持 delegate call 或对作弊码地址的调用。可以通过设置 delegateCall 为 true 支持 delegate call，也可以设置 tx.origin。

```solidity
prank(address msgSender[, address txOrigin, bool delegateCall])
```

startPrank 可与 stopPrank 一起使用，为它们之间的所有调用子句设置 msg.sender，同样可以设置 delegateCall 为 true 支持 delegate call，也可以设置 tx.origin。

```solidity
startPrank(address msgSender[, address txOrigin, bool delegateCall])
stopPrank()

/// prank delegate call and record state diffs
contract ImplementationTest {
    uint public num;
    address public sender;

    function setNum(uint _num) public {
        num = _num;
    }
}

contract ProxyTest {
    uint public num;
    address public sender;
}

contract FoundryIssue is Script {
    function run() public {
        ProxyTest proxy = new ProxyTest();
        ImplementationTest impl = new ImplementationTest();

        vm.label(address(proxy), 'proxy');
        vm.label(address(impl), 'Impl');

        uint num = 42;
        vm.startPrank(address(proxy), true);
        vm.startStateDiffRecording();
        (bool successTwo,) = address(impl).delegatecall(abi.encodeWithSignature('setNum(uint256)', num));

        VmSafe.AccountAccess[] memory accountAccesses = vm.stopAndReturnStateDiff();
        console.log('accountAccesses.kind', uint8(accountAccesses[0].kind));
        console.log('accountAccesses.accessor', vm.getLabel(accountAccesses[0].accessor));
        console.log('accountAccesses.account', vm.getLabel(accountAccesses[0].account));
        console.logBytes(accountAccesses[0].data);
    }
}
```

### record，accesses

record 用于通知 VM 开始记录合约存储所有的读取与写入，再通过 accesses 获取合约所有已被读取或写入的存储轮。

```solidity
/// contract NumsContract {
///     uint256 public num1 = 100; // slot 0
///     uint256 public num2 = 200; // slot 1
/// }

vm.record();
numsContract.num2();
(bytes32[] memory reads, bytes32[] memory writes) = vm.accesses(
  address(numsContract)
);
emit log_uint(uint256(reads[0])); // 1
```

### recordLogs，getRecordedLogs

recordLogs 用于通知 VM 开始记录所有发出的 event，再通过 getRecordedLogs 获取它们。

```solidity
/// event LogCompleted(
///   uint256 indexed topic1,
///   bytes data
/// );

vm.recordLogs();

emit LogCompleted(10, "operation completed");

Vm.Log[] memory entries = vm.getRecordedLogs();

assertEq(entries.length, 1);
assertEq(entries[0].topics[0], keccak256("LogCompleted(uint256,bytes)"));
assertEq(entries[0].topics[1], bytes32(uint256(10)));
assertEq(abi.decode(entries[0].data, (string)), "operation completed");
```

### setNonce，getNonce

setNonce 用于设置给定账户的 nonce，新的 nonce 必须比当前的 nonce 更高。getNonce 除了可以获取账户的 nonce 之外，也可以获取钱包的 nonce。

```solidity
vm.setNonce(address(100), 1234);

uint256 nonce = vm.getNonce(address(100));

Wallet memory alice = vm.createWallet("alice");
uint256 nonce = vm.getNonce(alice);
```

### mockcall，mockcalls，mockCallRevert，mockFunction，clearMockedCalls

这一组作弊码用于模拟外部合约调用，它们允许你在测试中覆盖某个地址的函数调用，返回自定义数据，而无需实际调用真实合约。mockcall 模拟单个函数调用，而 mockCalls 模拟多个函数调用，mockCallRevert 模拟调用失败，mockFunction 模拟函数调用并且可以设置动态返回值。clearMockedCalls 清除所有模拟调用，恢复对目标合约的真实调用。

### coinbase

coinbase 用于设置打包当前区块的矿工或验证者地址 block.coinbase。

```solidity
emit log_address(block.coinbase); // 0x0000000000000000000000000000000000000000
vm.coinbase(0xEA674fdDe714fd979de3EdF0F56AA9716B898ec8);
emit log_address(block.coinbase); // 0xea674fdde714fd979de3edf0f56aa9716b898ec8
```

### broadcast，startBroadcast，stopBroadcast

broadcast 使用调用测试合约的地址或作为发送方提供的地址/私钥，让下一次调用（仅在此调用深度，不包括作弊码调用）创建一个交易，随后可以在链上签名和发送。

```solidity
function deploy() public {
    vm.broadcast(ACCOUNT_A);
    Test test = new Test();

    // this won't generate tx to sign
    uint256 b = test.t(4);

    // this will
    vm.broadcast(ACCOUNT_B);
    test.t(2);

    // this also will, using a private key from your environment variables
    vm.broadcast(vm.envUint("PRIVATE_KEY"));
    test.t(3);
} 
```

startBroadcast 和 stopBroadcast 使用调用测试合约的地址或作为发送方提供的地址/私人密钥，所有后续调用（仅在此调用深度，不包括作弊码调用）都会创建可在链上签名和发送的交易。

```solidity
function t(uint256 a) public returns (uint256) {
    uint256 b = 0;
    emit log_string("here");
    return b;
}

function deployOther() public {
    vm.startBroadcast(ACCOUNT_A);
    Test test = new Test();
    
    // will trigger a transaction
    test.t(1);
    
    vm.stopBroadcast();

    // broadcast again, this time using a private key from your environment variables
    vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
    test.t(3);
    vm.stopBroadcast();
}
```

### txGasPrice

txGasPrice 为剩下的交易设置 tx.gasprice。

```solidity
function testCalculateGas() public {
    vm.txGasPrice(2);
    uint256 gasStart = gasleft();
    myContract.doStuff();
    uint256 gasEnd = gasleft();
    uint256 gasUsed = (gasStart - gasEnd) * tx.gasprice; // tx.gasprice is now 2
}
```

## 断言类

### expectRevert

expectRevert 用于断言接下来的合约调用会回滚，并可验证回滚的原因（如错误消息或自定义错误）。测试合约在无效输入或未满足条件时是否按预期回滚，验证回滚的错误类型（例如 require 消息或自定义错误）。

```solidity
// Assertion calls are rolled back and checked for error messages
vm.expectRevert("Insufficient balance");
contract.withdraw(100);

// Asserting custom errors
vm.expectRevert(abi.encodeWithSelector(CustomError.selector, arg1));
contract.fail();
```

### expectEmit

expectEmit 用于验证合约是否按预期发出了特定的日志事件（Event）。测试合约是否在特定操作后触发了正确的事件（包括事件参数），可检查事件的发出者。

```solidity
// A Transfer(address, address, uint256) event is expected to be triggered.
vm.expectEmit(true, true, false, address(contractA)); // The first two bools indicate whether or not to check the indexed parameter.
emit Transfer(from, to, amount);// Define expected events

// Trigger operations (e.g. call transfer function)
contractA.transfer(to, amount);
```

### expectCall

expectCall 用于验证目标合约是否按预期调用了另一个合约的特定函数（包括参数和调用次数）。

```solidity
// Assume contract A will call contract B's transfer(address, uint256) function
vm.expectCall(
    address(contractB), // Target contract address
    abi.encodeWithSelector(contractB.transfer.selector, recipient, amount) // Expected functions and parameters
);

// Trigger an action (e.g., call a method) in Contract A.
contractA.doSomething();
```

## 模糊测试类

### assume

assume 用于在 ​fuzz 测试中，动态跳过不满足条件的输入参数。

```solidity
// Good example of using assume
function testSomething(uint256 a) public {
    vm.assume(a != 1);
    require(a != 1);
    // [PASS]
}
```

### assumeNoRevert

assumeNoRevert 用于在 fuzz 测试中，​动态跳过会导致目标合约调用回滚的输入参数。

```solidity
function testDepositFuzz(uint256 amount) public {
    vm.assumeNoRevert(address(erc20Contract)); // Skip amounts that cause ERC20 transfers to fail.
    vault.deposit(amount); // Only test inputs that successfully complete an ERC20 transfer.
}
```

## 文件操纵

```solidity
// Reads the entire content of file to string, (path) => (data)
function readFile(string calldata) external returns (string memory);
/// Reads the entire content of file as binary. `path` is relative to the project root.
function readFileBinary(string calldata path) external view returns (bytes memory data);
/// Reads the directory at the given path recursively, up to `maxDepth`.
/// `maxDepth` defaults to 1, meaning only the direct children of the given directory will be returned.
/// Follows symbolic links if `followLinks` is true.
function readDir(string calldata path) external view returns (DirEntry[] memory entries);
// Reads next line of file to string, (path) => (line)
function readLine(string calldata) external returns (string memory);
/// Reads a symbolic link, returning the path that the link points to.
/// This cheatcode will revert in the following situations, but is not limited to just these cases:
/// - `path` is not a symbolic link.
/// - `path` does not exist.
function readLink(string calldata linkPath) external view returns (string memory targetPath);
// Writes data to file, creating a file if it does not exist, and entirely replacing its contents if it does.
// (path, data) => ()
function writeFile(string calldata, string calldata) external;
// Writes line to file, creating a file if it does not exist.
// (path, data) => ()
function writeLine(string calldata, string calldata) external;
// Closes file for reading, resetting the offset and allowing to read it from beginning with readLine.
// (path) => ()
function closeFile(string calldata) external;
// Removes file. This cheatcode will revert in the following situations, but is not limited to just these cases:
// - Path points to a directory.
// - The file doesn't exist.
// - The user lacks permissions to remove the file.
// (path) => ()
function removeFile(string calldata) external;
// Returns true if the given path points to an existing entity, else returns false
// (path) => (bool)
function exists(string calldata) external returns (bool);
// Returns true if the path exists on disk and is pointing at a regular file, else returns false
// (path) => (bool)
function isFile(string calldata) external returns (bool);
// Returns true if the path exists on disk and is pointing at a directory, else returns false
// (path) => (bool)
function isDir(string calldata) external returns (bool);
```

示例：

```solidity
fs_permissions = [{ access = "read-write", path = "./"}]

string memory path = "file.txt";
string memory data = "hello world";
vm.writeFile(path, data);

assertEq(vm.readFile(path), data);
```
