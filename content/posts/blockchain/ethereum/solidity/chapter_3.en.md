---
layout: post
Title: "Exploring Foundry Cheatcodes"
Description: "List and introduce commonly used Foundry Cheatcodes to write tests and debug smart contracts more efficiently."
date: 2025-03-27T16:28:27+08:00
image: "/posts/blockchain/ethereum/solidity/images/chapter_3-cover.jpg"
tags: ["Blockchain", "Solidity"]
---

Foundry is a powerful tool for Ethereum smart contract developers, and its Cheatcodes feature provides great convenience for development and testing. Whether it's simulating the blockchain environment, manipulating contract states, or quickly debugging complex logic, Cheatcodes can help developers achieve twice the result with half the effort. In this article, we will explore the various Cheatcodes provided by Foundry, explaining their uses and usage methods one by one, to help you write and debug Solidity smart contracts more efficiently.

## Environment

### warp，roll，fee，chainId

The `warp` is used to set the block timestamp, `roll` is used to set the block number, `fee` is used to set the base fee of the blockchain, and `chainId` is used to set the blockchain ID.

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

The function `getBlockTimestamp` is used to obtain the current block's timestamp, and `getBlockNumber` is used to get the current block number.

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

The `store` is used to set the storage of a certain account, indexed by `slot`; the `load` is used to retrieve data from the storage of a certain account, indexed by `slot`.

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

The `deal` function is used to set the account balance of a certain address. If the alternative signature in `StdCheats.sol` is used, the address of an ERC20 token can also be specified to directly set the token balance of a certain address. Such adjustments do not change the total supply.

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

The `prank` function is used to set `msg.sender` for the next call, supporting `staticcall`, but by default, it does not support `delegatecall` or calls to cheat code addresses. `delegatecall` can be enabled by setting `delegateCall` to `true`, and `tx.origin` can also be set.

```solidity
prank(address msgSender[, address txOrigin, bool delegateCall])
```

`startPrank` can be used in conjunction with `stopPrank` to set msg.sender for all the calling clauses between them. It can also set delegateCall to true to support delegate call and set tx.origin.

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

The `record` is used to notify the VM to start recording all the reads and writes of the contract's storage, and then the `accesses` can be used to obtain all the storage rounds that have been read from or written to by the contract.

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

The `recordLogs` function is used to notify the VM to start recording all emitted events, and then the recorded logs can be obtained through `getRecordedLogs`.

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

The `setNonce` function is used to set the nonce of a given account. The new nonce must be higher than the current one. The `getNonce` function can not only obtain the nonce of an account but also that of a wallet.

```solidity
vm.setNonce(address(100), 1234);

uint256 nonce = vm.getNonce(address(100));

Wallet memory alice = vm.createWallet("alice");
uint256 nonce = vm.getNonce(alice);
```

### mockcall，mockcalls，mockCallRevert，mockFunction，clearMockedCalls

This set of cheat codes is used to simulate external contract calls. They allow you to override function calls of a certain address in tests, return custom data, and avoid actually calling the real contract. `mockcall` simulates a single function call, while `mockCalls` simulates multiple function calls. `mockCallRevert` simulates a failed call, and `mockFunction` simulates a function call and can set dynamic return values. `clearMockedCalls` clears all simulated calls and restores real calls to the target contract.

### coinbase

Coinbase is used to set the address of the miner or validator who packs the current block, block.coinbase.

```solidity
emit log_address(block.coinbase); // 0x0000000000000000000000000000000000000000
vm.coinbase(0xEA674fdDe714fd979de3EdF0F56AA9716B898ec8);
emit log_address(block.coinbase); // 0xea674fdde714fd979de3edf0f56aa9716b898ec8
```

### broadcast，startBroadcast，stopBroadcast

The `broadcast` uses the address of the test contract invocation or the address/private key provided by the sender to create a transaction for the next invocation (only at this invocation depth, excluding cheatcode invocations), which can then be signed and sent on the chain.

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

`startBroadcast` and `stopBroadcast` use the address of the test contract being called or the address/private key provided by the sender. All subsequent calls (only at this call depth, excluding cheat code calls) will create transactions that can be signed and sent on the chain.

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

Set `tx.gasprice` for the remaining transactions with `txGasPrice`.

```solidity
function testCalculateGas() public {
    vm.txGasPrice(2);
    uint256 gasStart = gasleft();
    myContract.doStuff();
    uint256 gasEnd = gasleft();
    uint256 gasUsed = (gasStart - gasEnd) * tx.gasprice; // tx.gasprice is now 2
}
```

## Assertions

### expectRevert

`expectRevert` is used to assert that the subsequent contract call will revert and can verify the reason for the revert (such as an error message or a custom error). Test whether the contract reverts as expected when given invalid input or when conditions are not met, and verify the type of revert error (for example, a require message or a custom error).

```solidity
// Assertion calls are rolled back and checked for error messages
vm.expectRevert("Insufficient balance");
contract.withdraw(100);

// Asserting custom errors
vm.expectRevert(abi.encodeWithSelector(CustomError.selector, arg1));
contract.fail();
```

### expectEmit

`expectEmit` is used to verify whether a contract emits a specific log event as expected. It can be used to test whether a contract triggers the correct event (including event parameters) after a specific operation, and can also check the emitter of the event.

```solidity
// A Transfer(address, address, uint256) event is expected to be triggered.
vm.expectEmit(true, true, false, address(contractA)); // The first two bools indicate whether or not to check the indexed parameter.
emit Transfer(from, to, amount);// Define expected events

// Trigger operations (e.g. call transfer function)
contractA.transfer(to, amount);
```

### expectCall

`expectCall` is used to verify whether the target contract has called a specific function of another contract as expected (including parameters and the number of calls).

```solidity
// Assume contract A will call contract B's transfer(address, uint256) function
vm.expectCall(
    address(contractB), // Target contract address
    abi.encodeWithSelector(contractB.transfer.selector, recipient, amount) // Expected functions and parameters
);

// Trigger an action (e.g., call a method) in Contract A.
contractA.doSomething();
```

## Fuzzer

### assume

`assume` is used in fuzz testing to dynamically skip input parameters that do not meet the conditions.

```solidity
// Good example of using assume
function testSomething(uint256 a) public {
    vm.assume(a != 1);
    require(a != 1);
    // [PASS]
}
```

### assumeNoRevert

The `assumeNoRevert` is used in fuzz testing to dynamically skip input parameters that would cause the target contract call to revert.

```solidity
function testDepositFuzz(uint256 amount) public {
    vm.assumeNoRevert(address(erc20Contract)); // Skip amounts that cause ERC20 transfers to fail.
    vault.deposit(amount); // Only test inputs that successfully complete an ERC20 transfer.
}
```

## Files

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

Example：

```solidity
fs_permissions = [{ access = "read-write", path = "./"}]

string memory path = "file.txt";
string memory data = "hello world";
vm.writeFile(path, data);

assertEq(vm.readFile(path), data);
```
