---
title: Application Architecture
subtitle:
tags: [transport, device, communicate, companion wallet]
category: Connect your app
author:
toc: true
layout: doc
---


## Architecture Overview  


In order to understand deeply the interactions between different components of the application, we need to understand the architecture of the application.
The overall architecture of the integration is based on the following layers:

{: .center}
[![Ledger Live Prerequisites](../images/application-architecture.png){: width="100%" }](../images/application-architecture.png)
*Fig. 1: Overall Architecture*

On the client side we have your core application containing your business logic and workflows.
You need to integrate at least the LedgerJS Transport Library in order to discuss with the Ledger device.

Some major networks have dedicated JS Library, like Bitcoin / Ethereum. You should use them if you are planning to integrate with these apps. This will provide a good abstraction level and allow you to use a proper SDK.
You can see how to use them in [Integration Walkthrough - Web USB/HID](../web-hid-usb).


## Nano API calls

This section describes what are the role of the Nano API.
Indeed we call Nano API the "LedgerJS Dedicated App Lib" on the above image at the top of the page.
You must have encontered few of the APIs if you have gone through the [Integration Walkthrough](../web-hid-usb).
In the Walkthroughs, we have mostly used the Bitcoin and Ethereum API provided by Ledger.

The Nano API role is to help you to carry out operations in the "Nano Apps" (rf. image at the top).

The APIs are not written in the same way for all Nano apps. In addition, while some of them are provided by ledger, others are written by the various crypto communities.

You can have a look at how they are coded and documented at the <a href='https://github.com/LedgerHQ/ledgerjs'>LedgerHQ repository</a>.
You can also see the different Nano API provided by Ledger below.

### Nano APIs provided by Ledger

Here is the list of all the APIs provided by Ledger.

| Blockchain | Nano API |
|-------------|--------------|
|Algorand | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-algorand.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-algorand) [@ledgerhq/hw-app-algorand](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-algorand)   |
|Bitcoin | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-btc.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-btc) [@ledgerhq/hw-app-btc](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-btc)   |
|Cosmos | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-cosmos.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-cosmos) [@ledgerhq/hw-app-cosmos](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-cosmos)   |
|Elrond | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-elrond.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-elrond) [@ledgerhq/hw-app-elrond](https://github.com/LedgerHQ/ledger-live-common/tree/master/src/families/elrond/hw-app-elrond)   |
|Ethereum | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-eth.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-eth) [@ledgerhq/hw-app-eth](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-eth)   |
|Polkadot | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-polkadot.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-polkadot) [@ledgerhq/hw-app-polkadot](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-polkadot)   |
|Solana | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-solana.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-solana) [@ledgerhq/hw-app-solana](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-solana)   |
|Stellar | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-str.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-str) [@ledgerhq/hw-app-str](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-str)   |
|Tezos | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-tezos.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-tezos) [@ledgerhq/hw-app-tezos](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-tezos)   |
|Tron | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-trx.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-trx) [@ledgerhq/hw-app-trx](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-trx)   |
|Ripple | [![npm](https://img.shields.io/npm/v/@ledgerhq/hw-app-xrp.svg)](https://www.npmjs.com/package/@ledgerhq/hw-app-xrp) [@ledgerhq/hw-app-xrp](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-xrp)   |


### Nano APIs provided by the crypto-asset community

You may find these libraries on the GitHub repositories written by the community.

We currently do not have an updated list of all community-developed Nano APIs. If you wish to contribute yours, please use the comment box at the bottom of this page.

Here is the list of all the APIs from the community that you can find on npm.

| Blockchain | Nano API |
|-------------|--------------|
|Vite | [![npm](https://img.shields.io/npm/v/@vite/ledgerjs-hw-app-vite.svg)](https://www.npmjs.com/package/@vite/ledgerjs-hw-app-vite) [ledgerjs-hw-app-vite](https://github.com/vitelabs/ledgerjs-hw-app-vite)   |
|Ergo | [![npm](https://img.shields.io/npm/v/ledgerjs-hw-app-ergo.svg)](https://www.npmjs.com/package/ledgerjs-hw-app-ergo) [ledgerjs-hw-app-ergo](https://github.com/tesseract-one/ledger-app-ergo)   |
|Cardano | [![npm](https://img.shields.io/npm/v/@cardano-foundation/ledgerjs-hw-app-cardano.svg)](https://www.npmjs.com/package/@cardano-foundation/ledgerjs-hw-app-cardano) [@cardano-foundation/ledgerjs-hw-app-cardano](https://github.com/cardano-foundation/ledger-app-cardano)   |
|Sia | [![npm](https://img.shields.io/npm/v/@siacentral/ledgerjs-sia.svg)](https://www.npmjs.com/package/@siacentral/ledgerjs-sia) [@siacentral/ledgerjs-sia](https://github.com/siacentral/ledgerjs-sia)   |

## Nano API-less calls

When there is no available Nano API, the way to make your Nano calls is by constructing the ADPU call yourself, by using the syntax information found in each application documentation.

Here are some examples:

| Blockchain App | Application documentation |
|-------------|--------------| 
| Polymath | [Polymath](https://github.com/LedgerHQ/app-polymesh/blob/master/docs/APDUSPEC.md) | 
| Filecoin | [Filecoin](https://github.com/LedgerHQ/app-filecoin/blob/master/docs/APDUSPEC.md) | 

