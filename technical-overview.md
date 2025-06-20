# Crest Protocol Technical Overview

## Introduction

Crest is a self-custodial Bitcoin trading application that brings RFQ (Request for Quote) based trading to Citrea, an EVM-compatible ZK rollup on Bitcoin. It provides a seamless and efficient trading experience by combining off-chain price discovery with on-chain settlement, ensuring that users always maintain full control over their assets.

Our architecture is designed for capital efficiency and low latency, leveraging a ping-based RFQ system to source liquidity directly from professional market makers.

## How it Works

The Crest protocol is composed of two core components: an off-chain RFQ system for price discovery and an on-chain smart contract for settlement.

### Architecture Overview

1.  **Off-chain RFQ Engine**: This is the heart of our price discovery mechanism. Instead of relying on a continuous stream of bids and asks, which can be resource-intensive, our system pings connected market makers for quotes on demand.
2.  **On-chain Settlement Contract**: A secure and non-custodial smart contract on the Citrea network that executes trades. It ensures that the exchange of assets happens atomically and only with the explicit approval of both the user and the market maker.

### The RFQ Process

The off-chain RFQ process is designed to be fast and efficient, providing users with competitive, real-time quotes.

1.  **Quote Request**: A user initiates a trade by requesting a quote through the Crest interface for a specific asset pair and amount.
2.  **MM Ping**: Our RFQ engine instantly pings all connected market makers in parallel over a WebSocket connection.
3.  **Quote Aggregation**: The engine waits for a short period (e.g., 500ms) to gather quotes from the market makers. Each quote is a signed message containing the proposed price, expiry, and other trade parameters.
4.  **Best Quote Selection**: The system automatically selects the best quote (e.g., the one with the best price) from the responses.
5.  **Trade Execution**: The user is presented with the best quote and can choose to execute it.

### On-chain Settlement

Once a user accepts a quote, the settlement occurs on-chain via our `Settlement` smart contract. The contract offers two modes of settlement: RFQ-T and RFQ-M.

#### RFQ-T (Request for Quote and Trade)

In this mode, the user executes the trade directly by calling the `settleRFQT` function on the `Settlement` contract. This is a standard transaction where the user pays the gas fees. This flow is required when the user is trading the native asset (e.g., cBTC).

#### RFQ-M (Request for Quote - Meta-transaction)

For a more seamless experience (i.e., gas-less transaction for the user), Crest offers a relayer-based execution model.

1.  The user signs the quote, providing their approval for the trade.
2.  A relayer submits the user's signed quote and the market maker's signed quote to the `settleRFQM` function.
3.  The contract verifies both signatures and executes the trade. The relayer pays the gas fee.

This model is only available for trades where the input token is an ERC20 asset, as it relies on the user pre-approving the `Settlement` contract to spend their tokens.

## Smart Contracts

Our on-chain logic is encapsulated in the `Settlement.sol` contract, which is designed to be secure, non-custodial, and efficient.

### Non-Custodial by Design

The `Settlement` contract never holds custody of user or market maker funds. Instead, it uses the ERC20 `approve` and `transferFrom` pattern to move assets directly between the trading parties in a single, atomic transaction. This ensures that users and market makers are always in control of their funds.

### Core Settlement Functions

Here are the primary functions for trade settlement:

**`settleRFQT(QuoteParams calldata params, bytes calldata marketMakerSignature)`**

This function is called by the user to execute a trade. It verifies the market maker's signature and then proceeds with the asset swap.

```solidity
// crest-contracts/contracts/Settlement.sol

function settleRFQT(
    QuoteParams calldata params,
    bytes calldata marketMakerSignature
) external payable nonReentrant {
    // User must be the sender
    require(params.user == msg.sender, "Sender must be the user");

    // Validate the trade
    _validateRFQT(params, marketMakerSignature);

    // Execute the trade
    _executeRFQT(params);
}
```

**`settleRFQM(QuoteParams calldata params, bytes calldata marketMakerSignature, bytes calldata userSignature)`**

This function is called by a relayer to execute a trade on behalf of a user. It requires valid signatures from both the user and the market maker.

```solidity
// crest-contracts/contracts/Settlement.sol

function settleRFQM(
    QuoteParams calldata params,
    bytes calldata marketMakerSignature,
    bytes calldata userSignature
) external payable nonReentrant {
    // RFQM only supports ERC20 tokens for tokenIn
    require(params.tokenIn != NATIVE_TOKEN, "RFQM does not support native tokenIn");

    // Validate the trade
    _validateRFQM(params, marketMakerSignature, userSignature);

    // Execute the trade
    _executeRFQM(params);
}
```

### EIP-712 Signatures

To provide a better and safer user experience, Crest utilizes EIP-712 for signing quotes. This standard allows for structured, human-readable data to be signed, so users can see exactly what they are approving in their wallets.

The `hashQuote` function creates the EIP-712 compliant hash that market makers and users sign.

```solidity
// crest-contracts/contracts/Settlement.sol

function hashQuote(QuoteParams memory params) public view returns (bytes32) {
    return _hashTypedDataV4(
        keccak256(
            abi.encode(
                QUOTE_TYPEHASH,
                params.user,
                params.tokenIn,
                params.tokenOut,
                params.amountIn,
                params.amountOut,
                params.expiry,
                params.quoteId
            )
        )
    );
}
```

This ensures a high level of security and transparency for all parties involved in a trade.