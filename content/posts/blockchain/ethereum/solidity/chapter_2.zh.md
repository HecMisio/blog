---
layout: post
title: "Chainlink：解锁区块链的无限可能"
description: "探讨 Chainlink 如何通过CCIP、数据喂价、数据流、随机数和自动化服务扩展智能合约的功能边界"
date: 2025-03-26T10:24:07+08:00
image: "/posts/blockchain/ethereum/solidity/images/chapter_2-cover.jpg"
tags: ["Blockchain", "Solidity"]
---

在区块链的世界中，智能合约是一种强大的工具，它能够在满足特定条件时自动执行。然而，区块链本身是一个封闭的系统，无法直接访问链外的数据，例如金融市场的价格、天气信息或随机数。这种局限性使得智能合约的应用范围受到了限制。

## 什么是预言机？

**预言机（Oracle）** 是解决这一问题的关键。预言机是连接区块链与外部世界的桥梁，它能够将链外的数据引入区块链，或者将链上的事件传递到链外系统。但要构建一个预言机系统并不是一件容易的事情，因为它必须保证不破坏链上去中心化和数据安全。如何保证数据来源是去中心化的？如何保证数据是真实可靠的？如何保证服务的高可用性？

Chainlink 是目前最流行的去中心化预言机解决方案，它通过独特的架构和机制，成功应对了这些问题。

## Chainlink 架构

Chainlink 的成功离不开其独特的架构设计。作为一个去中心化预言机网络，Chainlink 通过多层次的架构确保了数据的可靠性、安全性和可扩展性，以下是其核心架构的关键组成部分。

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-simple-architecture-diagram.jpg" width="100%">
<p>Simple Architecture Diagram</p>
</div>

* 去中心化预言机网络（Decentralized Oracle Network，DON）：Chainlink 的核心是由多个独立运行的节点组成的去中心化网络。这些节点负责从链外数据源获取信息，并将其传递到区块链上，多个节点共同工作，避免单点故障。

* LINK 代币的经济模型：Chainlink 的经济模型基于其原生代币 LINK。节点运营者需要质押 LINK 代币以参与网络，并通过提供服务获得奖励，鼓励节点提供高质量服务，惩戒恶意节点，保证网络的安全性和稳定性。

* 链下报告（Offchain Reporting，OCR）：为了确保数据的准确性和抗操纵性，Chainlink 使用了数据聚合机制。多个节点提供的数据会被聚合成一个最终结果，并传递给智能合约。从多个可信数据源获取信息，避免单一来源的偏差，同时利用通过加权算法对数据进行处理，剔除异常值，提高数据的可信度和准确性，防止恶意节点提供虚假数据。

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-data-aggregation.jpg" width="80%">
<p>Data Aggregation</p>
</div>

* 链下组件（Offchain Components）：Chainlink 的链下组件是其架构的重要部分，负责与外部系统交互并处理复杂计算，扩展了智能合约的功能，使其能够访问链外资源。

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-offchain-to-onchain.jpg" width="100%">
<p>Offchain to Onchain</p>
</div>

* 链上组件（Onchain Components）：链上组件是 Chainlink 的另一个关键部分，负责接收链下节点提供的数据并将其传递给智能合约。收集和处理来自多个节点的数据，并提供透明的验证过程，确保数据的真实性和完整性。

## Chainlink 的核心服务

Chainlink 提供了一系列强大的核心服务，帮助开发者解决区块链与外部世界连接的难题，并扩展智能合约的功能边界。

### 数据喂价（Data Feeds）

数据喂价用于获取链外资产的实时数据，例如 ETH/USD 汇率。这对于去中心化借贷、去中心化交易所和衍生品平台至关重要。数据喂价是 Chainlink 最广泛使用的服务之一，特别是在去中心化金融（DeFi）领域。它通过多个独立节点和数据源提供高精度、去中心化的数据。

### 随机数生成（VRF, Verifiable Random Function）

随机数在区块链中有着广泛的应用，例如 NFT 铸造、链上游戏和抽奖活动。然而，在区块链上生成真正随机且可验证的随机数非常困难，因为区块链的去中心化和透明特性，智能合约中生成随机数的操作也是公开的，如果随机数的生成依赖于链上数据，如区块哈希或时间戳等，那么攻击者就可以预测或操纵这些数据，从而影响随机数的结构。

Chainlink VRF 提供了一种去中心化的随机数生成服务，能够生成真正随机且可验证的随机数，这种方法通过密码学保证随机数的不可预测性和公平性。

### 自动化服务（Automation）

Chainlink 的自动化服务允许开发者设置定时任务或条件触发的链上操作，极大地简化了智能合约的自动化管理，实现无需人工干预的链上任务，例如定时支付、清算操作或触发治理提案。在 DeFi 应用中，自动化服务可用于定期分发奖励或触发清算。

### Functions

Chainlink Functions 是一个强大的工具，允许开发者直接从智能合约中调用任意 Web API，从而实现链上与链下的无缝交互。在保险应用中，Functions 可以用于从天气 API 获取实时天气数据，并根据结果触发赔付。

### CCIP（跨链互操作协议）

CCIP（Cross-Chain Interoperability Protocol）是 Chainlink 提供的跨链解决方案，旨在实现不同区块链之间的互操作性。通过 CCIP，开发者可以在多个区块链之间传输数据和资产。在多链 DeFi 应用中，CCIP 可以用于在不同区块链之间转移资产，例如从以太坊转移 USDC 到 Polygon。

## Chainlink 核心服务代码实现

### 数据喂价

数据喂价用于获取链外资产的实时价格，例如 ETH/USD 汇率。不同的交易对对应的 Price Feed 合约地址不同，可以在 Chainlink 开发文档的 <a href="https://docs.chain.link/data-feeds/price-feeds/addresses?network=ethereum&page=1">Feed Address</a> 中查看。

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

contract DataFeedExample {
    AggregatorV3Interface internal priceFeed;

    constructor() {
        // Chainlink ETH/USD price feed address (Ethereum Sepolia)
        priceFeed = AggregatorV3Interface(0x694AA1769357215DE4FAC081bf1f309aDC325306);
    }

    /**
     * @dev Returns the latest ETH/USD price
     * @return price The latest price with 8 decimals
     */
    function getLatestPrice() public view returns (int) {
        (
            , // roundId
            int price, // latest price
            , // startedAt
            , // updatedAt
            // answeredInRound
        ) = priceFeed.latestRoundData();
        return price; // Return ETH/USD price with 8 decimals
    }
}
```

### 随机数生成

随机数生成服务既可以通过创建订阅，向订阅中注入代币管理消费合约，在消费合约中获取随机数，也可以不创建订阅，直接消耗代币获取随机数。以下演示使用代码创建订阅，并向订阅注入代币、添加消费合约，并请求随机数。

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-subscription-architecture-diagram.jpg" width="100%">
<p>Subscription Architecture Diagram</p>
</div>

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {LinkTokenInterface} from "@chainlink/contracts/src/v0.8/shared/interfaces/LinkTokenInterface.sol";
import {IVRFCoordinatorV2Plus} from "@chainlink/contracts/src/v0.8/vrf/dev/interfaces/IVRFCoordinatorV2Plus.sol";
import {VRFConsumerBaseV2Plus} from "@chainlink/contracts/src/v0.8/vrf/dev/VRFConsumerBaseV2Plus.sol";
import {VRFV2PlusClient} from "@chainlink/contracts/src/v0.8/vrf/dev/libraries/VRFV2PlusClient.sol";

contract VRFv2PlusSubscriptionManager is VRFConsumerBaseV2Plus {
    LinkTokenInterface LINKTOKEN;

    // Sepolia coordinator. For other networks,
    // see https://docs.chain.link/docs/vrf/v2-5/subscription-supported-networks#configurations
    address public vrfCoordinatorV2Plus =
        0x9DdfaCa8183c41ad55329BdeeD9F6A8d53168B1B;

    // Sepolia LINK token contract. For other networks, see
    // https://docs.chain.link/docs/vrf-contracts/#configurations
    address public link_token_contract =
        0x779877A7B0D9E8603169DdbD7836e478b4624789;

    // The gas lane to use, which specifies the maximum gas price to bump to.
    // For a list of available gas lanes on each network,
    // see https://docs.chain.link/docs/vrf/v2-5/subscription-supported-networks#configurations
    bytes32 public keyHash =
        0x787d74caea10b2b357790d5b5247c2f63d1d91572a9846f780606e4d953677ae;

    // A reasonable default is 100000, but this value could be different
    // on other networks.
    uint32 public callbackGasLimit = 100000;

    // The default is 3, but you can set this higher.
    uint16 public requestConfirmations = 3;

    // For this example, retrieve 1 random values in one request.
    // Cannot exceed VRFCoordinatorV2.MAX_NUM_WORDS.
    uint32 public numWords = 1;

    // Storage parameters
    uint256[] public s_randomWords;
    uint256 public s_requestId;
    uint256 public s_subscriptionId;

    constructor() VRFConsumerBaseV2Plus(vrfCoordinatorV2Plus) {
        s_vrfCoordinator = IVRFCoordinatorV2Plus(vrfCoordinatorV2Plus);
        LINKTOKEN = LinkTokenInterface(link_token_contract);
        //Create a new subscription when you deploy the contract.
        _createNewSubscription();
    }

    // Assumes the subscription is funded sufficiently.
    function requestRandomWords() external onlyOwner {
        // Will revert if subscription is not set and funded.
        s_requestId = s_vrfCoordinator.requestRandomWords(
            VRFV2PlusClient.RandomWordsRequest({
                keyHash: keyHash,
                subId: s_subscriptionId,
                requestConfirmations: requestConfirmations,
                callbackGasLimit: callbackGasLimit,
                numWords: numWords,
                extraArgs: VRFV2PlusClient._argsToBytes(
                    VRFV2PlusClient.ExtraArgsV1({nativePayment: false})
                )
            })
        );
    }

    function fulfillRandomWords(
        uint256 /* requestId */,
        uint256[] calldata randomWords
    ) internal override {
        s_randomWords = randomWords;
    }

    // Create a new subscription when the contract is initially deployed.
    function _createNewSubscription() private onlyOwner {
        s_subscriptionId = s_vrfCoordinator.createSubscription();
        // Add this contract as a consumer of its own subscription.
        s_vrfCoordinator.addConsumer(s_subscriptionId, address(this));
    }

    // Assumes this contract owns link.
    // 1000000000000000000 = 1 LINK
    function topUpSubscription(uint256 amount) external onlyOwner {
        LINKTOKEN.transferAndCall(
            address(s_vrfCoordinator),
            amount,
            abi.encode(s_subscriptionId)
        );
    }

    function addConsumer(address consumerAddress) external onlyOwner {
        // Add a consumer contract to the subscription.
        s_vrfCoordinator.addConsumer(s_subscriptionId, consumerAddress);
    }

    function removeConsumer(address consumerAddress) external onlyOwner {
        // Remove a consumer contract from the subscription.
        s_vrfCoordinator.removeConsumer(s_subscriptionId, consumerAddress);
    }

    function cancelSubscription(address receivingWallet) external onlyOwner {
        // Cancel the subscription and send the remaining LINK to a wallet address.
        s_vrfCoordinator.cancelSubscription(s_subscriptionId, receivingWallet);
        s_subscriptionId = 0;
    }

    // Transfer this contract's funds to an address.
    // 1000000000000000000 = 1 LINK
    function withdraw(uint256 amount, address to) external onlyOwner {
        LINKTOKEN.transfer(to, amount);
    }
}
```

### 自动化服务

自动化服务用于定时任务或条件触发的链上操作。自动化支持时间触发器、日志事件触发器和自定义逻辑触发器。时间触发器按照 cron 语法来周期调用链上合约，日志事件触发器则通过监听链上特定日志事件来调用链上合约，而自定义逻辑触发器通过实现适配自动化服务的 Upkeep 合约，通过 checkUpkeep 来控制是否触发调用。

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-automation.jpg" width="100%">
<p>Automation</p>
</div>

以下是适配自定义逻辑触发器的 Upkeep 合约实现示例。

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {AutomationCompatibleInterface} from "@chainlink/contracts/src/v0.8/automation/interfaces/AutomationCompatibleInterface.sol";

contract AutomationExample is AutomationCompatibleInterface {
    uint256 public counter;
    uint256 public interval;
    uint256 public lastTimeStamp;

    constructor(uint256 _interval) {
        interval = _interval;
        lastTimeStamp = block.timestamp;
        counter = 0;
    }

    function checkUpkeep(bytes calldata) external view override returns (bool upkeepNeeded, bytes memory) {
        upkeepNeeded = (block.timestamp - lastTimeStamp) > interval;
        return (upkeepNeeded, "0x0");
    }

    function performUpkeep(bytes calldata) external override {
        if ((block.timestamp - lastTimeStamp) > interval) {
            lastTimeStamp = block.timestamp;
            counter = counter + 1; // 定时增加计数器
        }
    }
}
```

你可以使用 <a href="https://automation.chain.link/">Chainlink Automation App</a> 来注册 Upkeep 合约，也可以使用类似下方的代码方式来注册合约。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {LinkTokenInterface} from "@chainlink/contracts/src/v0.8/shared/interfaces/LinkTokenInterface.sol";

struct RegistrationParams {
    string name;
    bytes encryptedEmail;
    address upkeepContract;
    uint32 gasLimit;
    address adminAddress;
    uint8 triggerType;
    bytes checkData;
    bytes triggerConfig;
    bytes offchainConfig;
    uint96 amount;
}

/**
 * string name = "test upkeep";
 * bytes encryptedEmail = 0x;
 * address upkeepContract = 0x...;
 * uint32 gasLimit = 500000;
 * address adminAddress = 0x....;
 * uint8 triggerType = 0;
 * bytes checkData = 0x;
 * bytes triggerConfig = 0x;
 * bytes offchainConfig = 0x;
 * uint96 amount = 1000000000000000000;
 */

interface AutomationRegistrarInterface {
    function registerUpkeep(
        RegistrationParams calldata requestParams
    ) external returns (uint256);
}

contract UpkeepIDConditionalExample {
    LinkTokenInterface public immutable i_link;
    AutomationRegistrarInterface public immutable i_registrar;

    constructor(
        LinkTokenInterface link,
        AutomationRegistrarInterface registrar
    ) {
        i_link = link;
        i_registrar = registrar;
    }

    function registerAndPredictID(RegistrationParams memory params) public {
        // LINK must be approved for transfer - this can be done every time or once
        // with an infinite approval
        i_link.approve(address(i_registrar), params.amount);
        uint256 upkeepID = i_registrar.registerUpkeep(params);
        if (upkeepID != 0) {
            // DEV - Use the upkeepID however you see fit
        } else {
            revert("auto-approve disabled");
        }
    }
}
```

### Functions

Functions 用于从智能合约中调用任意 Web API（该功能仍处于 Bate 测试阶段）。

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-request-and-receive.jpg" width="100%">
<p>Request And Receive</p>
</div>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {FunctionsClient} from "@chainlink/contracts/src/v0.8/functions/v1_0_0/FunctionsClient.sol";
import {ConfirmedOwner} from "@chainlink/contracts/src/v0.8/shared/access/ConfirmedOwner.sol";
import {FunctionsRequest} from "@chainlink/contracts/src/v0.8/functions/v1_0_0/libraries/FunctionsRequest.sol";

contract FunctionsConsumerExample is FunctionsClient, ConfirmedOwner {
    using FunctionsRequest for FunctionsRequest.Request;

    bytes32 public s_lastRequestId;
    bytes public s_lastResponse;
    bytes public s_lastError;

    error UnexpectedRequestID(bytes32 requestId);

    event Response(bytes32 indexed requestId, bytes response, bytes err);

    constructor(
        address router
    ) FunctionsClient(router) ConfirmedOwner(msg.sender) {}

    /**
     * @notice Send a simple request
     * @param source JavaScript source code
     * @param encryptedSecretsUrls Encrypted URLs where to fetch user secrets
     * @param donHostedSecretsSlotID Don hosted secrets slotId
     * @param donHostedSecretsVersion Don hosted secrets version
     * @param args List of arguments accessible from within the source code
     * @param bytesArgs Array of bytes arguments, represented as hex strings
     * @param subscriptionId Billing ID
     */
    function sendRequest(
        string memory source,
        bytes memory encryptedSecretsUrls,
        uint8 donHostedSecretsSlotID,
        uint64 donHostedSecretsVersion,
        string[] memory args,
        bytes[] memory bytesArgs,
        uint64 subscriptionId,
        uint32 gasLimit,
        bytes32 donID
    ) external onlyOwner returns (bytes32 requestId) {
        FunctionsRequest.Request memory req;
        req.initializeRequestForInlineJavaScript(source);
        if (encryptedSecretsUrls.length > 0)
            req.addSecretsReference(encryptedSecretsUrls);
        else if (donHostedSecretsVersion > 0) {
            req.addDONHostedSecrets(
                donHostedSecretsSlotID,
                donHostedSecretsVersion
            );
        }
        if (args.length > 0) req.setArgs(args);
        if (bytesArgs.length > 0) req.setBytesArgs(bytesArgs);
        s_lastRequestId = _sendRequest(
            req.encodeCBOR(),
            subscriptionId,
            gasLimit,
            donID
        );
        return s_lastRequestId;
    }

    /**
     * @notice Send a pre-encoded CBOR request
     * @param request CBOR-encoded request data
     * @param subscriptionId Billing ID
     * @param gasLimit The maximum amount of gas the request can consume
     * @param donID ID of the job to be invoked
     * @return requestId The ID of the sent request
     */
    function sendRequestCBOR(
        bytes memory request,
        uint64 subscriptionId,
        uint32 gasLimit,
        bytes32 donID
    ) external onlyOwner returns (bytes32 requestId) {
        s_lastRequestId = _sendRequest(
            request,
            subscriptionId,
            gasLimit,
            donID
        );
        return s_lastRequestId;
    }

    /**
     * @notice Store latest result/error
     * @param requestId The request ID, returned by sendRequest()
     * @param response Aggregated response from the user code
     * @param err Aggregated error from the user code or from the execution pipeline
     * Either response or error parameter will be set, but never both
     */
    function fulfillRequest(
        bytes32 requestId,
        bytes memory response,
        bytes memory err
    ) internal override {
        if (s_lastRequestId != requestId) {
            revert UnexpectedRequestID(requestId);
        }
        s_lastResponse = response;
        s_lastError = err;
        emit Response(requestId, s_lastResponse, s_lastError);
    }
}
```

### CCIP

CCIP（跨链互操作协议）用于在不同区块链之间传输数据和资产。要利用 CCIP 需要定义发送者和接收者。

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-ccip-diagram.jpg" width="100%">
<p>CCIP Diagram</p>
</div>

发送者示例代码如下:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.24;

import {IRouterClient} from "@chainlink/contracts-ccip/src/v0.8/ccip/interfaces/IRouterClient.sol";
import {OwnerIsCreator} from "@chainlink/contracts-ccip/src/v0.8/shared/access/OwnerIsCreator.sol";
import {Client} from "@chainlink/contracts-ccip/src/v0.8/ccip/libraries/Client.sol";
import {LinkTokenInterface} from "@chainlink/contracts/src/v0.8/shared/interfaces/LinkTokenInterface.sol";

/**
 * THIS IS AN EXAMPLE CONTRACT THAT USES HARDCODED VALUES FOR CLARITY.
 * THIS IS AN EXAMPLE CONTRACT THAT USES UN-AUDITED CODE.
 * DO NOT USE THIS CODE IN PRODUCTION.
 */

/// @title - A simple contract for sending string data across chains.
contract Sender is OwnerIsCreator {
    // Custom errors to provide more descriptive revert messages.
    error NotEnoughBalance(uint256 currentBalance, uint256 calculatedFees); // Used to make sure contract has enough balance.

    // Event emitted when a message is sent to another chain.
    event MessageSent(
        bytes32 indexed messageId, // The unique ID of the CCIP message.
        uint64 indexed destinationChainSelector, // The chain selector of the destination chain.
        address receiver, // The address of the receiver on the destination chain.
        string text, // The text being sent.
        address feeToken, // the token address used to pay CCIP fees.
        uint256 fees // The fees paid for sending the CCIP message.
    );

    IRouterClient private s_router;

    LinkTokenInterface private s_linkToken;

    /// @notice Constructor initializes the contract with the router address.
    /// @param _router The address of the router contract.
    /// @param _link The address of the link contract.
    constructor(address _router, address _link) {
        s_router = IRouterClient(_router);
        s_linkToken = LinkTokenInterface(_link);
    }

    /// @notice Sends data to receiver on the destination chain.
    /// @dev Assumes your contract has sufficient LINK.
    /// @param destinationChainSelector The identifier (aka selector) for the destination blockchain.
    /// @param receiver The address of the recipient on the destination blockchain.
    /// @param text The string text to be sent.
    /// @return messageId The ID of the message that was sent.
    function sendMessage(
        uint64 destinationChainSelector,
        address receiver,
        string calldata text
    ) external onlyOwner returns (bytes32 messageId) {
        // Create an EVM2AnyMessage struct in memory with necessary information for sending a cross-chain message
        Client.EVM2AnyMessage memory evm2AnyMessage = Client.EVM2AnyMessage({
            receiver: abi.encode(receiver), // ABI-encoded receiver address
            data: abi.encode(text), // ABI-encoded string
            tokenAmounts: new Client.EVMTokenAmount[](0), // Empty array indicating no tokens are being sent
            extraArgs: Client._argsToBytes(
                // Additional arguments, setting gas limit and allowing out-of-order execution.
                // Best Practice: For simplicity, the values are hardcoded. It is advisable to use a more dynamic approach
                // where you set the extra arguments off-chain. This allows adaptation depending on the lanes, messages,
                // and ensures compatibility with future CCIP upgrades. Read more about it here: https://docs.chain.link/ccip/best-practices#using-extraargs
                Client.EVMExtraArgsV2({
                    gasLimit: 200_000, // Gas limit for the callback on the destination chain
                    allowOutOfOrderExecution: true // Allows the message to be executed out of order relative to other messages from the same sender
                })
            ),
            // Set the feeToken  address, indicating LINK will be used for fees
            feeToken: address(s_linkToken)
        });

        // Get the fee required to send the message
        uint256 fees = s_router.getFee(
            destinationChainSelector,
            evm2AnyMessage
        );

        if (fees > s_linkToken.balanceOf(address(this)))
            revert NotEnoughBalance(s_linkToken.balanceOf(address(this)), fees);

        // approve the Router to transfer LINK tokens on contract's behalf. It will spend the fees in LINK
        s_linkToken.approve(address(s_router), fees);

        // Send the message through the router and store the returned message ID
        messageId = s_router.ccipSend(destinationChainSelector, evm2AnyMessage);

        // Emit an event with message details
        emit MessageSent(
            messageId,
            destinationChainSelector,
            receiver,
            text,
            address(s_linkToken),
            fees
        );

        // Return the message ID
        return messageId;
    }
}
```

接收者示例代码如下：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.24;

import {Client} from "@chainlink/contracts-ccip/src/v0.8/ccip/libraries/Client.sol";
import {CCIPReceiver} from "@chainlink/contracts-ccip/src/v0.8/ccip/applications/CCIPReceiver.sol";

/**
 * THIS IS AN EXAMPLE CONTRACT THAT USES HARDCODED VALUES FOR CLARITY.
 * THIS IS AN EXAMPLE CONTRACT THAT USES UN-AUDITED CODE.
 * DO NOT USE THIS CODE IN PRODUCTION.
 */

/// @title - A simple contract for receiving string data across chains.
contract Receiver is CCIPReceiver {
    // Event emitted when a message is received from another chain.
    event MessageReceived(
        bytes32 indexed messageId, // The unique ID of the message.
        uint64 indexed sourceChainSelector, // The chain selector of the source chain.
        address sender, // The address of the sender from the source chain.
        string text // The text that was received.
    );

    bytes32 private s_lastReceivedMessageId; // Store the last received messageId.
    string private s_lastReceivedText; // Store the last received text.

    /// @notice Constructor initializes the contract with the router address.
    /// @param router The address of the router contract.
    constructor(address router) CCIPReceiver(router) {}

    /// handle a received message
    function _ccipReceive(
        Client.Any2EVMMessage memory any2EvmMessage
    ) internal override {
        s_lastReceivedMessageId = any2EvmMessage.messageId; // fetch the messageId
        s_lastReceivedText = abi.decode(any2EvmMessage.data, (string)); // abi-decoding of the sent text

        emit MessageReceived(
            any2EvmMessage.messageId,
            any2EvmMessage.sourceChainSelector, // fetch the source chain identifier (aka selector)
            abi.decode(any2EvmMessage.sender, (address)), // abi-decoding of the sender address,
            abi.decode(any2EvmMessage.data, (string))
        );
    }

    /// @notice Fetches the details of the last received message.
    /// @return messageId The ID of the last received message.
    /// @return text The last received text.
    function getLastReceivedMessageDetails()
        external
        view
        returns (bytes32 messageId, string memory text)
    {
        return (s_lastReceivedMessageId, s_lastReceivedText);
    }
}
```
