---
kip: 18
title: Cross Margin
status: Draft
author: Jeremy Chiaramonte
created: 2022-06-30
---

## Summary

Cross margin enables futures positions to be managed from a singular margin account.

## Abstract

An abstraction layer to be built on top of the Synthetix Futures (V1) protocol. It enables the creation of "margin accounts" for each trader. Margin accounts enable Kwenta to provide cross margin functionality.

## Motivation

The current Futures system requires traders to deposit margin into specific markets before trading. This two step process causes friction when trying to act on market conditions and when managing margin between multiple markets. Cross margin improves the UX of trading on Kwenta.

## Specification

A major change introduced with this KIP is the introduction of a "margin account" system. Each trader will be required to set up a smart contract wallet (this step is managed by the UI) that holds margin on behalf of the trader. Smart contract accounts also open the door for future functionality as seen by [KIP-19: Advanced Orders](./kip-19.md).

Cross margin is achieved by bundling together the margin step and the trade step when modifying a futures position. With the current implementation a trader can also open/modify/close multiple positions at once (atomically). It will be up to the frontend implementation on how to manage and rebalance margin. 

### Rationale & User Flows

- As a **trader** I want:
    - To gain exposure to a **singular asset**:
        - By opening BTC position without having to perform two transactions.
        - By opening BTC position using the margin from closing a LINK position.
    - To gain exposure to **multiple assets**:
        - By opening BTC and ETH position without having to perform multiple transactions.
    - To gain exposure on the **negative correlation of two assets**:
        - By opening pair trade ETH/SOL in one step.
        - By rebalancing a pair trade with profits from a drifting asset.

### Technical Specification

There will be a factory contract (MarginAccountFactory) that creates new margin accounts. Each margin account's functionality will be restricted to the owner of the account, which is the original creator.

The current architecture relies on MinimalProxies where each margin account is a minimal proxy. Events are emitted on account creation.

`distributeMargin` is the core function of Cross Margin and accepts an array of `UpdateMarketPositionSpec` structs. 

```
function distributeMargin(UpdateMarketPositionSpec[] memory _newPositions);
``` 

Each spec will specify the market, margin Δ, size Δ, and optional* boolean to denote positions being closed. 

A positive margin Δ will deposit margin into a specified market and a negative margin delta will withdraw margin from a specified market. 

A positive size Δ will increase the notional size of a position and negative size Δ will decrease the notional size or invert the direction of a position (long vs. short).

*Note that positions can still be closed if the passed size Δ is the exact inverse of the current size. 

### Fee Structure

Fees (TBD) will be charged on the rebalancing of margin. 

### Caveats

An expectation of typical cross margin is rebalancing. This iteration does not include rebalancing, due to keeper system constraints, and it will be up to the end user to balance their margin. For example if a trader opens two positions with equal margin for both, over time their positions will become unbalanced respective to market movements and it will be possible for one position to be liquidated before the other.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
