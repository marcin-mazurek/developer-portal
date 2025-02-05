---
title: "4 - Add accounts: light sync"
tags: [Ledger Live Common, typescript]
category: Blockchain Support
author:
toc: true
layout: doc
---

## Types

Generic types can be found in [src/types/](https://github.com/Ledger-Coin-Integration-team/ledger-live-common/blob/master/src/types), with some documentation in source.

### Account

An account represents a wallet where a user detains crypto assets for a given crypto currency.

Ledger Live model is essentially an array of `Account` because many accounts can be created, even within a same crypto currency.

More technically, an account is a view of the blockchain in the context of a specific user. While blockchain tracks the series of transactions, assets motions that happen between addresses, an account essentializes that from the point of view of a user that owns a set of addresses and wants to view the associated funds they detain and be able to perform transactions with it.

Essentially what the user wants to see at the end is his balance, a historic graph, and a series of past operations that were performed. Moving from the blockchain to the concept of account is not necessarily trivial (in blockchains like Bitcoin, the concept of account does not exist – you don't create or destroy, this concept is a view, a lense, that we abstract for the user).

In Live Common, there are currently 3 types of accounts:

- `Account` which is a top level account associated to a `CryptoCurrency` - the "main" account.
- `TokenAccount` which is a nested level account, **inside** an Account and that is associated to a `TokenCurrency`.
- `ChildAccount` which is a variant of TokenAccount but more in context of a `CryptoCurrency` (typically used for Tezos)

They are aggregated as a single `AccountLike` type, used across Ledger Live implementations.

We will focus only on the `Account` type as we won't cover the Token integration in this document.

All main accounts share a common ground, that you will find defined and commented [here](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/types/account.ts#L118).

If needed by the blockchain, an account can also contain coin-specific resources related to a single account, like its "nonce" or additional balances (e.g. for staking), or anything that may be displayed or used in your implementation. It's generally an additional field like `myCoinResources`. See [Family-specific types](#family-specific-types) below.

### Operation

{% include alert.html style="note" text="This type is not required at light sync step and will soon be removed from this section" %}

Note the wording "Operation" is used instead of "Transaction". An account does not have transactions, it has **operations**.

The same abstraction applies as for Account on top of blockchain: an Operation is a view of a transaction that happened in the blockchain that **concerns** the contextual account. It's always in the context of an account, in other words, an operation does not exist on its own.

Most of the time, a transaction yield of one operation. But in some blockchains (like Ethereum) one transaction that concerns the account associated addresses can result in 0 to N operations. A typical example is contract or token transfers (one transaction to move a token generate a token operation and a fee operation in the ethereum account). Another example that is possible in most blockchains is a "self" transaction yields two operations (sending out, sending in). But in fact, that's semantic, we, Ledger, have put. We could also have allowed the concept of "SELF" operation.

In short, transactions history in Ledger Live is a list of `Operation`, that are confirmed, unconfirmed or pending (not yet fetched from explorer).

They all share the same model, with an `extra` field that can store any additional data you may need to display. You will find the detailed and commented fields [here](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/types/operation.ts#L34).

If <i>MyCoin</i> has specific operation fields (like `additionalField` we added for example), you will be able to display them later. They are not meant to be useful in any flow, only for UI.

If <i>MyCoin</i> uses a "nonce", then `transactionSequenceNumber` must be filled correctly, as it will be necessary for signing new transactions (and will interpreted to clear pending operations). Only outgoing transaction must have this value though.

### Operation Type

An `Operation` has a `type` which is generic string typed as `OperationType`, giving more or less the direction of the operation:

- `OUT`: A send / transfer amount transaction
- `IN`: A received / incoming amount transaction
- `FEES`: A transaction that only charges fees
- `NONE`: A transaction that does not affect balance
- `REWARD`: A claimed reward (as an outgoing transaction with fees)
- `REWARD_PAYOUT`: A received reward (as an incoming transaction)
- `SLASH`: A staking slash (with slashed amount generally)

There are more types available, existing one will have predefined icons, translations and behaviours (i.e. `getOperationAmountNumber()` in [src/operation.ts](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/operation.ts)).

<i>MyCoin</i> could have also specific operation types, if you need to add a type that is not yet implemented, add them in `src/types/operation.ts`. You will later need to implement some specific code for the Ledger Live Desktop and Mobile to display them correctly.

You will be asked to add an svg for the icon of your type and a label to be translated, you can check for "OPT_IN" operation in Algorand.

In LLD :

```
src/renderer/components/OperationsList/ConfirmationCheck.js
```

In LLM :
(You will need to check SVG is wrapped into our component)

```
src/icons/OperationStatusIcon/index.js
```

### Transaction

{% include alert.html style="note" text="This type may not be required at light sync step." %}

In Ledger Live, the "Transaction" is the data model that is created and updated in order to build the final blob to be signed by the device, and then broadcasted to the blockchain.

It is highly specific to the blockchain protocol, and contains any data that will need to be serialized into the transmission format of the blockchain. It only lives for a short period of time in the application - during which the user is choosing the parameters of its transaction before it is signed and sent to the blockchain, after which it will be transformed into an Operation.

Note that this Operation will be considered "pending", as it is an optimistic version of the real Operation that will appear in the account history, after being synchronized.

`Transaction` will contains any data needed to create a transaction on the blockchain. It is created as soon as the user initiates a transaction flow on Ledger Live, and will be updated according to its inputs, like the amount or choosing a recipient.

It shares a common shape between all coins, to which we will add specific-fields:

- `amount: BigNumber`: amount parsed from user input - will be 0 if user use all amount
- `recipient: string`: the recipient of this amount
- `useAllAmount?: boolean`: indicated if user wants to transfer whole available balance
- `subAccountId?: ?string`: the child/token account id if relevant
- `family: string`: the coin-family of the transaction (as a coin discriminator)

Some coins also have some fields generally respecting the same semantics, here are some provided as example:

- `mode: string`: type of transaction to be broadcasted (`"send"`, `"freeze"`, `"delegate"`, `"claimReward"`...)
- `fees: BigNumber`: fees (provided by blockchain, or filled by user)
- `memo: ?string`: memo if required by exchange

Although some fields are required, they can be emptied (recipent = "" and amount = 0) and ignored for transactions not using them.

You should add any fields that would be required by <i>MyCoin</i> to be correctly broadcasted - respecting as much as possible its protocol's lexicon.

See existing implementations for inspiration: [Polkadot types](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/families/polkadot/types.ts), [Cosmos types](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/families/cosmos/types.ts)

### Family-specific types

You will be implementing typescript types that will be used in your integration, like the Transaction type or the additional data needed to be added to the Account shared type, but also any other types that you will need (remember to always type your functions with typescript).

`src/families/mycoin/types.ts`:

```ts
import type { BigNumber } from "bignumber.js";
import type {
  TransactionCommon,
  TransactionCommonRaw,
} from "../../types/transaction";

/**
 * MyCoin account resources
 */
export type MyCoinResources = {
  nonce: number;
  additionalBalance: BigNumber;
};

/**
 * MyCoin account resources from raw JSON
 */
export type MyCoinResourcesRaw = {
  nonce: number;
  additionalBalance: string;
};

/**
 * MyCoin transaction
 */
export type Transaction = TransactionCommon & {
  mode: string;
  family: "mycoin";
  fees?: BigNumber;
  // add here all transaction-specific fields if you implement other modes than "send"
};

/**
 * MyCoin transaction from a raw JSON
 */
export type TransactionRaw = TransactionCommonRaw & {
  family: "mycoin";
  mode: string;
  fees?: string;
  // also the transaction fields as raw JSON data
};

/**
 * MyCoin currency data that will be preloaded.
 * You can for instance add a list of validators for Proof-of-Stake blockchains,
 * or any volatile data that could not be set as constants in the code (staking progress, fee estimation variables, etc.)
 */
export type MyCoinPreloadData = {
  somePreloadedData: Record<any, any>;
};
```

Since some of thoses types will be serialized when stored or cached, you may need to define serialize/deserialize functions for those:

`src/families/mycoin/serialization.ts`:

```ts
import { BigNumber } from "bignumber.js";
import type { MyCoinResourcesRaw, MyCoinResources } from "./types";

export function toMyCoinResourcesRaw(r: MyCoinResources): MyCoinResourcesRaw {
  const { nonce, additionalBalance } = r;
  return {
    nonce,
    additionalBalance: additionalBalance.toString(),
  };
}

export function fromMyCoinResourcesRaw(r: MyCoinResourcesRaw): MyCoinResources {
  const { nonce, additionalBalance } = r;
  return {
    nonce,
    additionalBalance: BigNumber(additionalBalance),
  };
}
```

Because of Account being generic, you may need to add your specific resources to `src/types/account.ts`, if you need to store specific information related to the blockchain (like staking, validators, or frozen balance...) that are not handle in Account.

```ts
// ...
import type {
  MyCoinResources,
  MyCoinResourcesRaw,
} from "../families/mycoin/types";

// export type Account = {
// ...
// // On some blockchain, an account can have resources (gained, delegated, ...)
  myCoinResources?: MyCoinResources;
// };

// export type AccountRaw = {
  myCoinResources?: MyCoinResourcesRaw;
// };
```

...and handle the associated serialization in `src/account/serialization.ts` (if you use BigInt you will need to make it raw by changing it to string for example):

```ts
// ...
import {
  toMyCoinResourcesRaw,
  fromMyCoinResourcesRaw,
} from "../families/mycoin/serialization";
// ...
export { toMyCoinResourcesRaw, fromMyCoinResourcesRaw };
// ...

// export function fromAccountRaw(rawAccount: AccountRaw): Account {
//   const {
//     ...
    myCoinResources,
// }
// ...
  if (myCoinResources) {
    res.myCoinResources = fromMyCoinResourcesRaw(myCoinResources);
  }
//   return res;
// }

// export function toAccountRaw({
// ...
  myCoinResources,
// }: Account): AccountRaw {
// ...
  if (myCoinResources) {
    res.myCoinResources = toMyCoinResourcesRaw(myCoinResources);
  }
//   return res;
// }
```

<!--  -->

{% include alert.html style="success" text="If your integration of <i>MyCoin</i> does not require coin-specific data in an account, you will not need to define <code>MyCoinResources</code>." %}

<!--  -->

### Display Format

Since `Operation` will be stored as JSON, you will need to implement specific serializers for the `extra` field.

We also would like the `Operation` and `Account` to be displayed in CLI with their specifics, so you must provide formatters to display them.

{% include alert.html style="note" text="In this code sample, all references to operations are not needed at light sync step. They are currently necessary but will soon be removed from this section" %}

`src/families/mycoin/account.ts`:

```ts
import { BigNumber } from "bignumber.js";
import type { Account, Operation, Unit } from "../../types";
import { getAccountUnit } from "../../account";
import { formatCurrencyUnit } from "../../currencies";

function formatAccountSpecifics(account: Account): string {
  const { myCoinResources } = account;
  if (!myCoinResources) {
    throw new Error("mycoin account expected");
  }

  const unit = getAccountUnit(account);
  const formatConfig = {
    disableRounding: true,
    alwaysShowSign: false,
    showCode: true,
  };

  let str = " ";

  str +=
    formatCurrencyUnit(unit, account.spendableBalance, formatConfig) +
    " spendable. ";

  if (myCoinResources.additionalBalance.gt(0)) {
    str +=
      formatCurrencyUnit(
        unit,
        myCoinResources.additionalBalance,
        formatConfig
      ) + " additional. ";
  }

  if (myCoinResources.nonce) {
    str += "\nonce : " + myCoinResources.nonce;
  }

  return str;
}

function formatOperationSpecifics(op: Operation, unit: ?Unit): string {
  const { additionalField } = op.extra;

  let str = " ";

  const formatConfig = {
    disableRounding: true,
    alwaysShowSign: false,
    showCode: true,
  };

  str +=
    additionalField && !additionalField.isNaN()
      ? `\n    additionalField: ${
          unit
            ? formatCurrencyUnit(unit, additionalField, formatConfig)
            : additionalField
        }`
      : "";

  return str;
}

export function fromOperationExtraRaw(extra: ?Object) {
  if (extra && extra.additionalField) {
    extra = {
      ...extra,
      additionalField: BigNumber(extra.additionalField),
    };
  }
  return extra;
}

export function toOperationExtraRaw(extra: ?Object) {
  if (extra && extra.additionalField) {
    extra = {
      ...extra,
      additionalField: extra.additionalField.toString(),
    };
  }
  return extra;
}

export default {
  formatAccountSpecifics,
  formatOperationSpecifics,
  fromOperationExtraRaw,
  toOperationExtraRaw,
};
```

<!--  -->

{% include alert.html style="success" text="<code>formatOperationSpecifics()</code> and <code>formatAccountSpecifics()</code> are used in the CLI to display account-specific fields and extras of the transaction history, useful for debugging." %}

<!--  -->

The same idea applies also to the `Transaction` type which needs to be serialized and formatted for CLI:

{% include alert.html style="note" text="This part may not be required if you don't need transactions for light sync." %}

`src/families/mycoin/transaction.ts`:

```ts
import type { Transaction, TransactionRaw } from "./types";
import { BigNumber } from "bignumber.js";
import {
  fromTransactionCommonRaw,
  toTransactionCommonRaw,
} from "../../transaction/common";
import type { Account } from "../../types";
import { getAccountUnit } from "../../account";
import { formatCurrencyUnit } from "../../currencies";

export const formatTransaction = (
  { mode, amount, recipient, useAllAmount }: Transaction,
  account: Account
): string =>
  `
${mode.toUpperCase()} ${
    useAllAmount
      ? "MAX"
      : amount.isZero()
      ? ""
      : " " +
        formatCurrencyUnit(getAccountUnit(account), amount, {
          showCode: true,
          disableRounding: true,
        })
  }${recipient ? `\nTO ${recipient}` : ""}`;

export const fromTransactionRaw = (tr: TransactionRaw): Transaction => {
  const common = fromTransactionCommonRaw(tr);
  return {
    ...common,
    family: tr.family,
    mode: tr.mode,
    fees: tr.fees ? BigNumber(tr.fees) : null,
  };
};

export const toTransactionRaw = (t: Transaction): TransactionRaw => {
  const common = toTransactionCommonRaw(t);
  return {
    ...common,
    family: t.family,
    mode: t.mode,
    fees: t.fees?.toString() || null,
  };
};

export default { formatTransaction, fromTransactionRaw, toTransactionRaw };
```

## Wrap your API

{% include alert.html style="note" text="In the code samples below, all references to operations are not needed at light sync step. They are currently necessary but will soon be removed from this section" %}

Before this part, you will need a node/explorer to get the state of an account on the blockchain, such as balances, nonce (if your blockchain uses something similar), and any data relevant to show or fetch in Ledger Live.

For the example, we will assume that <i>MyCoin</i> provides a SDK that is able to do both getting the state and the account history.

The best way to implement your API in Live Common is to create a dedicated `api` subfolder, that exports all functions that require calls to 3rd-party and map their responses to Ledger Live types.

```plaintext
./src/families/mycoin
└── api
  ├── index.ts
  ├── sdk.ts
  └── sdk.types.ts
```

<!--  -->

{% include alert.html style="tip" text="Try to separate as much as possible your different APIs (if you use multiple providers) and use typings to ensure you map correctly API responses to Ledger Live types" %}

<!--  -->

You will likely need to export thoses functions, but implemention is up-to-developer:

`src/families/mycoin/api/index.ts`:

```ts
export {
  getAccount,
  getOperations,
  getPreloadedData, // adjust with needs of preloaded data
  getFees,
  submit,
  disconnect, // if using persisting connection
} from "./sdk";
```

Basically, in the next sections, `getAccount` will be called to create an `Account` with balances and any additional resources, and `getOperations` will be called to fill the `operations[]` of this account, with the whole history of operations that can be requested incrementally. Then `getFees` before sending a transaction to let the user know of the network cost (estimated or effective), and `submit` to broadcast its transaction after signing.

See [Polkadot Coin Integration's api](https://github.com/LedgerHQ/ledger-live-common/tree/master/src/families/polkadot/api) for good inspiration.

### API Example

Here is an example of a <i>MyCoin</i> API implementation using a fictive SDK that uses something like WebSocket to fetch data.

<!--  -->

{% include alert.html style="tip" text="We don't recommend using WebSocket as it could have drawbacks and is more difficult to scale up and put behind a reverse proxy. If possible, use HTTPS requests as much as possible." %}

<!--  -->

```ts
import MyCoinApi from "my-coin-sdk";
import type { MyCoinTransaction } from "my-coin-sdk";
import { BigNumber } from "bignumber.js";

import { getEnv } from "../../../env";
import { encodeOperationId } from "../../../operation";

type AsyncApiFunction = (api: typeof MyCoinApi) => Promise<any>;

const MYCOIN_API_ENDPOINT = () => getEnv("MYCOIN_API_ENDPOINT");

let api = null;

/**
 * Connects to MyCoin Api
 */
async function withApi(execute: AsyncApiFunction): Promise<any> {
  // If client is instanciated already, ensure it is connected & ready
  if (api) {
    try {
      await api.isReady;
    } catch (err) {
      api = null;
    }
  }

  if (!api) {
    api = new MyCoinApi(MYCOIN_API_ENDPOINT);
    api.connect();
    api.onClose(() => {
      api = null;
    });
    await api.isReady;
  }

  try {
    const res = await execute(api);

    return res;
  } catch {
    // Handle Error or Retry
    await disconnect();
  }
}

/**
 * Disconnects MyCoin Api
 */
export const disconnect = async () => {
  if (api) {
    const disconnecting = api;
    api = null;
    await disconnecting.close();
  }
};

/**
 * Get account balances and nonce
 */
export const getAccount = async (addr: string) =>
  withApi(async (api) => {
    const { balance, additionalBalance } = await api.getBalances(addr);
    const nonce = await api.getNonce(addr);
    const blockHeight = await api.getBlockHeight();

    return {
      blockHeight,
      balance: BigNumber(balance),
      additionalBalance: BigNumber(additionalBalance),
      nonce,
    };
  });

/**
 * Returns true if account is the signer
 */
function isSender(transaction: MyCoinTransaction, addr: string): boolean {
  return transaction.sender === addr;
}

/**
 * Map transaction to an Operation Type
 */
function getOperationType(
  transaction: MyCoinTransaction,
  addr: string
): OperationType {
  return isSender(transaction, addr) ? "OUT" : "IN";
}

/**
 * Map transaction to a correct Operation Value (affecting account balance)
 */
function getOperationValue(
  transaction: MyCoinTransaction,
  addr: string
): BigNumber {
  return isSender(transaction, addr)
    ? BigNumber(transaction.value).plus(transaction.fees)
    : BigNumber(transaction.value);
}

/**
 * Extract extra from transaction if any
 */
function getOperationExtra(transaction: MyCoinTransaction): Object {
  return {
    additionalField: transaction.additionalField,
  };
}

/**
 * Map the MyCoin history transaction to a Ledger Live Operation
 */
function transactionToOperation(
  accountId: string,
  addr: string,
  transaction: MyCoinTransaction
): Operation {
  const type = getOperationType(transaction, addr);

  return {
    id: encodeOperationId(accountId, transaction.hash, type),
    accountId,
    fee: BigNumber(transaction.fees || 0),
    value: getOperationValue(transaction, addr),
    type,
    // This is where you retrieve the hash of the transaction
    hash: transaction.hash,
    blockHash: null,
    blockHeight: transaction.blockNumber,
    date: new Date(transaction.timestamp),
    extra: getOperationExtra(transaction),
    senders: [transaction.sender],
    recipients: transaction.recipient ? [transaction.recipient] : [],
    transactionSequenceNumber: isSender(transaction, addr)
      ? transaction.nonce
      : undefined,
    hasFailed: !transaction.success,
  };
}

/**
 * Fetch operation list
 */
export const getOperations = async (
  accountId: string,
  addr: string,
  startAt: number
): Promise<Operation[]> =>
  withApi(async (api) => {
    const rawTransactions = await api.getHistory(addr, startAt);

    return rawTransactions.map((transaction) =>
      transactionToOperation(accountId, addr, transaction)
    );
  });
```

If you need to disconnect from your API after using it, update `src/api/index.ts` to add your api disconnect in the `disconnectAll` function, it will avoid tests and CLI to hang.

## Account Bridge

AccountBridge offers a generic abstraction to synchronize accounts and perform transactions.

It is designed for the end user frontend interface and is agnostic of the way it runs, has multiple implementations and does not know how the data is even stored: in fact **it's just a set of stateless functions**.

<!-- ------------- Image ------------- -->
<!-- --------------------------------- -->

![account bridge flow](../images/account-bridge-flow.png)

### Receive

The `receive` method allows to derivatives address of an account with a Nano device but also display it on the device if verify is passed in.
As you may see in `src/families/mycoin/bridge.ts`, Live Common provides a helper to implement it easily with `makeAccountBridgeReceive()`, and there is a very few reason to implement your own.

### Synchronization

We usually group the `scanAccounts` and `sync` into the same file `js-synchronisation.ts` as they both use similar logic as a `getAccountShape` function passed to helpers.

`src/families/mycoin/js-synchronisation.ts`:

```ts
import type { Account } from "../../types";
import type { GetAccountShape } from "../../bridge/jsHelpers";
import { makeSync, makeScanAccounts, mergeOps } from "../../bridge/jsHelpers";

import { getAccount, getOperations } from "./api";

const getAccountShape: GetAccountShape = async (info) => {
  const { id, address, initialAccount } = info;
  const oldOperations = initialAccount?.operations || [];

  // Needed for incremental synchronisation
  const startAt = oldOperations.length
    ? (oldOperations[0].blockHeight || 0) + 1
    : 0;

  // get the current account balance state depending your api implementation
  const { blockHeight, balance, additionalBalance, nonce } = await getAccount(
    address
  );

  // Merge new operations with the previously synced ones
  const newOperations = await getOperations(id, address, startAt);
  const operations = mergeOps(oldOperations, newOperations);

  const shape = {
    id,
    balance,
    spendableBalance: balance,
    operationsCount: operations.length,
    blockHeight,
    myCoinResources: {
      nonce,
      additionalBalance,
    },
  };

  return { ...shape, operations };
};

const postSync = (initial: Account, parent: Account) => parent;

export const scanAccounts = makeScanAccounts(getAccountShape);

export const sync = makeSync(getAccountShape, postSync);
```

The `scanAccounts` function performs the derivation of addresses for a given `currency` and `deviceId`, and returns an Observable that will notify every `Account` that it discovered.

With the `makeScanAccounts` helper, you only have to pass a `getAccountShape` function to execute the generic scan that Ledger Live use with the correct derivation modes for <i>MyCoin</i>, and it will determine when to stop (generally as soon as an empty account was found).

The `sync` function performs one "Account synchronisation" which consists of refreshing all fields of a (previously created) Account from your api.

It is executed every 2 minutes if everything works as expected, but if it fails, a retry strategy will be executed with an increasing delay (to avoid burdening a failing API).

Under the hood of the `makeSync` helper, the returned value is an Observable of an updater function (Account=>Account), which is a pattern having some advantages:

- it avoids race conditions
- the updater is called in a reducer, and allows to produce an immutable state by applying the update to the latest account instance (with reconciliation on Ledger Live Desktop)
- it's an observable, so we can interrupt it when/if multiple updates occurs

In some cases, you might need to do a `postSync` patch to add some update logic after sync (<i>before the reconciliation that occurs on Ledger Live Desktop</i>). If this `postSync` function is complex, you should split this function in a `src/families/mycoin/js-postSyncPatch.js` file.

### Reconciliation

Currently, Ledger Live Desktop executes this bridge in a separate thread. Thus, the "avoid race condition" aspect of sync might not be respected since the UI renderer thread does not share the same objects.
This may be improved in the future, but for updates to be reflected during sync, we implemented reconciliation in [src/reconciliation.js](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/reconciliation.ts), between the account that is in the renderer and the new account produced after sync.

Since we might have added some coin-specific data in `Account`, we must also reconciliate it:

`src/reconciliation`:

```ts
// import {
// ...
  fromMyCoinResourcesRaw,
// } from "./account";
// ...
// export function patchAccount(
//   account: Account,
//   updatedRaw: AccountRaw
// ): Account {
// ...
  if (
    updatedRaw.myCoinResources &&
    account.myCoinResources !== updatedRaw.myCoinResources
  ) {
    next.myCoinResources = fromMyCoinResourcesRaw(
      updatedRaw.myCoinResources
    );
    changed = true;
  }
//   if (!changed) return account; // nothing changed at all
//
//   return next;
// }
```

## Currency Bridge

### Scanning accounts

As we have seen [Synchronization](#synchronization), the `scanAccounts`, which is part of the CurrencyBridge, share common logic with the sync function, that's why we preferably put them in a `js-synchronisation.ts` file.

The `makeScanAccounts` helper will automatically execute the default address derivation logic, but for some reason if you need to have a completely new way to scan account, you could then implement your own strategy.

## Icon

Icons are usually maintained by Ledger's design team, so you must first check that <i>MyCoin</i> icon is not already added in ledger-live-common, in [src/data/icons/svg](https://github.com/LedgerHQ/ledger-live-common/tree/master/src/data/icons/svg). It contains cleaned-up versions of Cryptocurrency Icons from [cryptoicons.co](http://cryptoicons.co/), organized by ticker.

If you need to add your own, they must respect those requirements:

- Clean SVG with **only** `<path>` elements representing the crypto
- Size and viewport must be `24x24`
- Icon should be `18x18` and centered / padded
- Flat-styled, and must respect crypto color scheme (filled)
- No background or decorative shape added
- No `<g>` or `transform`, `style` attributes...

Name should be the coin's ticker (e.g. `MYC.svg`) and must not conflict with an existing coin or token.

When building ledger-live-common, a [script](https://github.com/LedgerHQ/ledger-live-common/blob/master/scripts/buildReactIcons.js) automatically converts them to React and React Native components.


## Starting with a mock

A mock will help you test different UI flows on Desktop and Mobile.
It's connected to any indexer / explorer and gives you a rough idea on how it will look like when connected to the UI.

For example you can use it by doing `MOCK=1 yarn start` on `ledger-live-desktop`

```ts
import { BigNumber } from "bignumber.js";
import {
  NotEnoughBalance,
  RecipientRequired,
  InvalidAddress,
  FeeTooHigh,
} from "@ledgerhq/errors";
import type { Transaction } from "../types";
import type { AccountBridge, CurrencyBridge } from "../../../types";
import {
  scanAccounts,
  signOperation,
  broadcast,
  sync,
  isInvalidRecipient,
} from "../../../bridge/mockHelpers";
import { getMainAccount } from "../../../account";
import { makeAccountBridgeReceive } from "../../../bridge/mockHelpers";

const receive = makeAccountBridgeReceive();

const createTransaction = (): Transaction => ({
  family: "mycoin",
  mode: "send",
  amount: BigNumber(0),
  recipient: "",
  useAllAmount: false,
  fees: null,
});

const updateTransaction = (t, patch) => ({ ...t, ...patch });

const prepareTransaction = async (a, t) => t;

const estimateMaxSpendable = ({ account, parentAccount, transaction }) => {
  const mainAccount = getMainAccount(account, parentAccount);
  const estimatedFees = transaction?.fees || BigNumber(5000);
  return Promise.resolve(
    BigNumber.max(0, mainAccount.balance.minus(estimatedFees))
  );
};

const getTransactionStatus = (account, t) => {
  const errors = {};
  const warnings = {};
  const useAllAmount = !!t.useAllAmount;

  const estimatedFees = BigNumber(5000);

  const totalSpent = useAllAmount
    ? account.balance
    : BigNumber(t.amount).plus(estimatedFees);

  const amount = useAllAmount
    ? account.balance.minus(estimatedFees)
    : BigNumber(t.amount);

  if (amount.gt(0) && estimatedFees.times(10).gt(amount)) {
    warnings.amount = new FeeTooHigh();
  }

  if (totalSpent.gt(account.balance)) {
    errors.amount = new NotEnoughBalance();
  }

  if (!t.recipient) {
    errors.recipient = new RecipientRequired();
  } else if (isInvalidRecipient(t.recipient)) {
    errors.recipient = new InvalidAddress();
  }

  return Promise.resolve({
    errors,
    warnings,
    estimatedFees,
    amount,
    totalSpent,
  });
};

const accountBridge: AccountBridge<Transaction> = {
  estimateMaxSpendable,
  createTransaction,
  updateTransaction,
  getTransactionStatus,
  prepareTransaction,
  sync,
  receive,
  signOperation,
  broadcast,
};

const currencyBridge: CurrencyBridge = {
  scanAccounts,
  preload: async () => {},
  hydrate: () => {},
};

export default { currencyBridge, accountBridge };
```

## Split your code

You can now start to implement the JS bridge for <i>MyCoin</i>. It may need some changes back and forth between the types, your api wrapper, and the different files.

The skeleton of `src/families/mycoin/bridge/js.ts` should look something like this:

```ts
import type { AccountBridge, CurrencyBridge } from "../../../types";
import type { Transaction } from "../types";
import { makeAccountBridgeReceive } from "../../../bridge/jsHelpers";

import { getPreloadStrategy, preload, hydrate } from "../preload";

import { sync, scanAccounts } from "../js-synchronisation";

const receive = makeAccountBridgeReceive();

const currencyBridge: CurrencyBridge = {
  getPreloadStrategy,
  preload,
  hydrate,
  scanAccounts,
};

const createTransaction = () => {
  throw new Error("createTransaction not implemented");
};

const prepareTransaction = () => {
  throw new Error("prepareTransaction not implemented");
};

const updateTransaction = () => {
  throw new Error("updateTransaction not implemented");
};

const getTransactionStatus = () => {
  throw new Error("getTransactionStatus not implemented");
};

const estimateMaxSpendable = () => {
  throw new Error("estimateMaxSpendable not implemented");
};

const signOperation = () => {
  throw new Error("signOperation not implemented");
};

const broadcast = () => {
  throw new Error("broadcast not implemented");
};

const accountBridge: AccountBridge<Transaction> = {
  estimateMaxSpendable,
  createTransaction,
  updateTransaction,
  getTransactionStatus,
  prepareTransaction,
  sync,
  receive,
  signOperation,
  broadcast,
};

export default { currencyBridge, accountBridge };
```

<!--  -->

{% include alert.html style="tip" text="You could implement all the methods in a single file, but for better readability and maintainability, you should split your code into multiple files." %}

<!--  -->

## Cache and performance (optional)

It is important to keep in mind that all currencies work independently and that Live Common provides a common framework to synchronize accounts with a polling strategy, and that network connectivity is not always stable and optimal.

Hence, the more cryptocurrencies Ledger Live is using, the more requests and calculations are executed, which can take time.

To avoid making the same requests several times, we recommend using a local cache in your implementation (e.g. fees estimations, some currency data to preload, etc in a `src/families/mycoin/cache.ts` file.

We have a [`src/cache.ts`](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/cache.ts) helper for creating Least-Recently-Used caches anywhere if needed.

See for example the [Polkadot's cache implementation](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/families/polkadot/cache.ts).

## React Hooks

If you are adding specific features to Ledger Live (like staking), you may need to access data through React hooks, that could provide common logic reusable for React components.

You are then free to add them in a `src/families/mycoin/react.ts` file.

See examples like sorting and filtering validators, subscribing to preloaded data observable, or waiting for a transaction to be reflected in account, in the [Polkadot React hooks](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/families/polkadot/react.ts).

