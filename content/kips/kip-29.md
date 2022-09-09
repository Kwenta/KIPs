---
kip: 29
title: Copy Trading Account MVP
status: Draft
author: Troy (@Tburm)
created: 2022-09-08
---

## Summary

A proposal to implement a feature to allow traders to open copy trading accounts on Kwenta.

## Abstract

This proposal outlines a new feature on Kwenta which allows traders to open an account which copies the trades of another account. Traders will specify a cross-margin account and a list of assets to copy to create a new copy trading account. Once the account is created, keepers will listen to trades on the target account and rebalance the copy trading account to mirror market exposure.

## Motivation

1. Traders have expressed interest in copying trades of other accounts on Kwenta. Since futures market trades are all executed on-chain, traders have access to historical trading data for every other trading account.
2. Traders want to promote others to copy trade them in exchange for a share of their profits.

## Specification

Terminology:
- Copy trading account: A cross-margin (CM) enabled smart contract account which mirrors trades from another CM account 
- Target account: The account being copied by a copy trading account

The `MarginBase.sol` contract provides users with an interface to manage margin and futures positions across many markets in a single transaction. Keepers will use these functions to copy positions from the target account and execute trades on the copy trading account. To enable these functions the `MarginBase.sol` contract needs some new functionality:

- Stores the address of the target account (`address targetAccount`.)
- Stores an array of markets to be copied (`address[] copiedMarkets`.)
- A function `rebalance` which matches the market exposure between the copy trading account and the target account when they are imbalanced.
- Implements new access controls which prevent traders from submitting specific trades. Instead, these functions are delegated to a keeper account.
- Implements a new `withdraw` function which closes positions proportionally and withdraws margin to the account owner.

**Account Creation**

Account creation follows a similar flow to cross margin, however users are required to specify two inputs at account creation time: `targetAccount` and `copiedMarkets`. Once an account is created, the account owner can deposit sUSD to the account. The rebalance function is called on deposit to copy positions between the copy trading account and the target account.

**Rebalancing**

When the `copiedTrader` account emits any `PositionModified` event, the keeper calls `rebalance` on the copy trading account. Rebalancing will open or close positions accordingly to give the copy trader the same market exposure as their target using the following logic:

* `marginTotal = marginFree + marginMarket1 + ... + marginMarketN`
* `freeMarginPct = marginFree / marginTotal`
* `marginPctMarket1 = marginMarket1 / marginTotal` (calculate for all `copiedMarkets`)
* `leverageMarket1 = notionalSizeMarket1 / marginMarket1` (calculate for all `copiedMarkets`)
* Modify positions such that `freeMarginPct`, `marginPctMarketN`, and `leverageMarketN` are equal for all `copiedMarkets` between the copy trading account and the target account

**Considerations and Risks**

* Open interest caps could cause transactions to fail if the copy trading account trade sizes exceed caps during rebalancing
* Dynamic fees could cause the copy trader to pay significant fees compared to their target since the transactions are executed sequentially

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
