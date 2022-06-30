---
kip: 19
title: Advanced Orders
status: Draft
author: Jeremy Chiaramonte
created: 2022-06-30
---

## Summary

Advanced orders introduce to new trading strategies for futures.

## Abstract

Leveraging Gelato's keeper system, Kwenta will enable the automation of orders for futures. The first two products to come of this will be Limit Orders & Stop Market Orders. 

## Motivation

Market orders are the only available order type for Kwenta futures at the moment. There are trading benefits by allowing orders to execute in the future based on defined conditions. 

## Specification

Advanced orders are now feasible as a result of "margin accounts" introduced in [KIP-18: Cross Margin](./kip-18.md).

A trader can specify a limit order or stop market order and register the task with Gelato. If the specified condition evaluates true, Gelato keepers will execute the order on behalf of the trader. 

Margin will be committed for an advanced order meaning it cannot be used for any other order or cross margin while the order is active. 

Currently only one order (limit or stop) can be active for a market.

### Slippage

If the next oracle price falls below a given price during a limit buy order, (ie. $400 to buy ETH & new oracle price = $398) the order will experience positive slippage and will execute at $398.

If the next oracle price falls below a given price for a stop loss order, (ie. $400 stop loss & new oracle price = $398) the order will experience negative slippage and will execute at $398.

### Rationale & User Flows

- As a **trader** I want:
    - To gain exposure at a particular price:
        - By placing a limit buy to buy an asset at a given price or better.
    - To reduce risk on a bounce:
        - By placing a limit sell to sell an asset at a given price or better. 
    - To catch a rising trend:
        - By placing a buy stop order to purchase at a given price or higher.
    - To cap downside on a trade:
        - By placing a stop loss order to sell a declining asset at a specified price.

### Technical Specification

All advanced orders are indexed by market key. This means there can only be one order per market. There are three key functions for interacting with advanced orders:

- Placing an order. This stores the order onchain until execution or cancellation. 
```
function placeOrder(
    address market, 
    int256 marginDelta, 
    int256 sizeDelta, 
    uint256 givenPrice
) external onlyOwner;
```
- Cancelling an order. This removes an order for a given market.
```
function cancelOrder(address market) external onlyOwner;
```
- Executing an order. This will be executed by a keeper when a condition is true.
```
function executeOrder(address market) external onlyOps;
```

When an order is placed margin is committed to that order. This prevents weird UX loopholes where margin expected for a limit order can accidentally be used by other orders or cross margin. Committed margin is also not eligible for withdrawal from the margin account. The remaining amount of `freeMargin` is visible in the contract with:

```
function freeMargin() public view returns (uint256);
```

### Fee Structure

Fees (TBD) will be charged on positive slippage of a limit order. See section on *Slippage*.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
