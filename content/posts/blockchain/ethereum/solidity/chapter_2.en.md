---
layout: post
Title: "Exploring Chainlink"
Description: "Explore how Chainlink expands the functional boundaries of smart contracts through CCIP, data feeds, data streams, random number generation, and automation services."
date: 2025-03-26T10:24:07+08:00
image: "/posts/blockchain/ethereum/solidity/images/chapter_2-cover.jpg"
tags: ["Blockchain", "Solidity"]
---

In the world of blockchain, smart contracts are a powerful tool that can be automatically executed when certain conditions are met. However, blockchain itself is a closed system and cannot directly access off-chain data, such as prices in financial markets, weather information, or random numbers. This limitation restricts the application scope of smart contracts.

## What is an Oracle?

**Oracles** are the key to solving this problem. Oracles act as a bridge between the blockchain and the outside world, capable of introducing off-chain data onto the blockchain or transmitting on-chain events to off-chain systems. However, building an oracle system is no easy task, as it must ensure that it does not undermine the decentralization and data security of the chain. How can the source of data be guaranteed to be decentralized? How can the authenticity and reliability of the data be ensured? How can the high availability of the service be guaranteed?

Chainlink is currently the most popular decentralized oracle solution. Through its unique architecture and mechanisms, it has successfully addressed these issues.

## Chainlink Architecture

The success of Chainlink is inseparable from its unique architectural design. As a decentralized oracle network, Chainlink ensures the reliability, security and scalability of data through a multi-level architecture. The following are the key components of its core architecture.

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-simple-architecture-diagram.jpg" width="100%">
<p>Simple Architecture Diagram</p>
</div>

* Decentralized Oracle Network (DON): The core of Chainlink is a decentralized network composed of multiple independently operating nodes. These nodes are responsible for obtaining information from off-chain data sources and transmitting it to the blockchain. Multiple nodes work together to avoid single points of failure.

* The economic model of the LINK token: Chainlink's economic model is based on its native token, LINK. Node operators need to stake LINK tokens to participate in the network and receive rewards for providing services. This encourages nodes to offer high-quality services, punishes malicious nodes, and ensures the security and stability of the network.

* Offchain Reporting (OCR): To ensure the accuracy and anti-manipulation of data, Chainlink employs a data aggregation mechanism. Data provided by multiple nodes is aggregated into a final result and passed to the smart contract. Information is obtained from multiple trusted data sources to avoid bias from a single source. At the same time, data is processed through weighted algorithms to eliminate outliers, enhancing the credibility and accuracy of the data and preventing malicious nodes from providing false data.

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-data-aggregation.jpg" width="80%">
<p>Data Aggregation</p>
</div>

* Offchain Components: Chainlink's offchain components are a crucial part of its architecture, responsible for interacting with external systems and handling complex computations, thereby extending the functionality of smart contracts and enabling them to access offchain resources.

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-offchain-to-onchain.jpg" width="100%">
<p>Offchain to Onchain</p>
</div>

* Onchain Components: Onchain components are another key part of Chainlink, responsible for receiving data provided by off-chain nodes and passing it to smart contracts. They collect and process data from multiple nodes and provide a transparent verification process to ensure the authenticity and integrity of the data.

## Core Services of Chainlink

Chainlink offers a series of powerful core services to help developers solve the problem of connecting the blockchain with the outside world and expand the functional boundaries of smart contracts.

### Data Feeds

Data oracles are used to obtain real-time data of off-chain assets, such as the ETH/USD exchange rate. This is crucial for decentralized lending, decentralized exchanges, and derivatives platforms. Data oracles are one of the most widely used services of Chainlink, especially in the decentralized finance (DeFi) sector. They provide high-precision and decentralized data through multiple independent nodes and data sources.

### Verifiable Random Function (VRF)

Random numbers have a wide range of applications in blockchain, such as NFT minting, on-chain games, and lottery activities. However, generating truly random and verifiable random numbers on the blockchain is very difficult. Due to the decentralized and transparent nature of the blockchain, the operation of generating random numbers in smart contracts is also public. If the generation of random numbers relies on on-chain data, such as block hashes or timestamps, then attackers can predict or manipulate these data, thereby affecting the structure of the random numbers.

Chainlink VRF offers a decentralized random number generation service, capable of generating truly random and verifiable random numbers. This method ensures the unpredictability and fairness of the random numbers through cryptography.

### Automation Services (Automation)

Chainlink's automated services enable developers to set up scheduled tasks or condition-triggered on-chain operations, greatly simplifying the automated management of smart contracts and achieving on-chain tasks without human intervention, such as scheduled payments, liquidation operations, or triggering governance proposals. In DeFi applications, automated services can be used to regularly distribute rewards or trigger liquidations.

### Functions

Chainlink Functions is a powerful tool that enables developers to directly call any Web API from smart contracts, facilitating seamless interaction between on-chain and off-chain data. In insurance applications, Functions can be used to fetch real-time weather data from weather APIs and trigger payouts based on the results.

### CCIP (Cross-Chain Interoperability Protocol)

CCIP (Cross-Chain Interoperability Protocol) is a cross-chain solution provided by Chainlink, aiming to achieve interoperability among different blockchains. Through CCIP, developers can transfer data and assets across multiple blockchains. In multi-chain DeFi applications, CCIP can be used to transfer assets between different blockchains, such as moving USDC from Ethereum to Polygon.

## Chainlink Core Service Code Implementation

### Data Feeds

Data Feeds are used to obtain the real-time prices of off-chain assets, such as the ETH/USD exchange rate. Different trading pairs correspond to different Price Feed contract addresses, which can be viewed in the Chainlink developer documentation's <a href="https://docs.chain.link/data-feeds/price-feeds/addresses?network=ethereum&page=1">Feed Address</a>.

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

### Verifiable Random Function

The VRF services can either be accessed by creating a subscription, injecting tokens into the subscription to manage the consumption contract, and obtaining the random number within the consumption contract, or it can be accessed without creating a subscription by directly consuming tokens to obtain the random number. The following demonstration uses code to create a subscription, inject tokens into the subscription, add a consumption contract, and request a random number.

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

### Automation

Automation services are used for on-chain operations triggered by scheduled tasks or conditions. Automation supports time triggers, log event triggers, and custom logic triggers. Time triggers periodically call on-chain contracts according to the cron syntax, log event triggers call on-chain contracts by listening to specific log events on the chain, and custom logic triggers call on-chain contracts by implementing an Upkeep contract that adapts to the automation service and controlling whether to trigger the call through checkUpkeep.

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-automation.jpg" width="100%">
<p>Automation</p>
</div>

The following is an example implementation of the Upkeep contract adapted for custom logic triggers.

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
            counter = counter + 1;
        }
    }
}
```

You can use the <a href="https://automation.chain.link/">Chainlink Automation App</a> to register the Upkeep contract, or you can register the contract using a code snippet similar to the one below.

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

Functions are used to call any Web API from a smart contract (this feature is still in the Beta testing stage).

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

CCIP (Cross-Chain Interoperability Protocol) is used to transfer data and assets between different blockchains. To utilize CCIP, it is necessary to define the sender and the receiver.

<div class="content-image">
<img src="/posts/blockchain/ethereum/solidity/images/chapter_2-ccip-diagram.jpg" width="100%">
<p>CCIP Diagram</p>
</div>

The sender's sample code is as follows:

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

The sample code for the receiver is as follows:

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
