# Create an account

**An account is an object that makes it easier to send and receive transactions. Accounts store data such as addresses and pending bundle hashes in a local database. This data allows you to interact with an IOTA network without worrying about reusing addresses or promoting and reattaching pending transactions.**

In accounts, all addresses are more than simple IOTA addresses. These addresses are called [conditional deposit addresses (CDAs)](../how-to-guides/create-and-manage-cda.md). A CDA defines not only the 81-tryte address, but also the conditions in which that address may be used in a [transfer bundle](root://getting-started/0.1/introduction/what-is-a-bundle.md).

## Seed state

The data that accounts store in a local database is called the seed state. Accounts use this data to keep a history of activity and to avoid making unnecessary API calls to nodes.

|**Data**| **Purpose**|
|:-----------------|:----------|
|The last key index that was used to create a CDA| Create a new CDA that has never been used before|
|All active CDAs|Stop withdrawals from CDAs that may receive deposits|
|Pending transfers| Monitor pending transactions and rebroadcast or reattach them if necessary|

## Create a new account

1. Install the libraries

      ```bash
      npm install @iota/account
      ```

2. Create an `account` object with a new seed and connect to a node
   
      ```js
      import { createAccount } from '@iota/account';

      const seed = 'ASFITGPSD9ASDFKRWE...';

      // Local node to connect to;
      const provider = 'http://<node-url>:14265';

      const account = createAccount({
            seed,
            provider
      });
      ```
      :::info:
      You can create multiple accounts, and each one can manage the state of only one unique seed.
      :::

      :::danger:Important
      You must not create multiple accounts with the same seed. Doing so could lead to a race condition where the seed state would be overwritten.

      If you have never created an account before, you must create a new seed. Existing seeds can't be used in an account because their states are unknown.
      :::

3. **Optional:** Pass a **`persistenceAdapter`** factory to your account. This adapter creates a local database object to which the account can save the seed state and its history. By default, the local databases are saved in the root of the project. You can change the path to the local database in the `persistencePath` field.

      ```js
      import { createPersistenceAdapter } from '@iota/persistence-adapter-level';

      const account = createAccount({
            seed,
            provider,
            persistencePath: './',
            stateAdapter: createPersistenceAdapter,
            historyAdapter: createPersistenceAdapter
      });
      ```

      :::info:
      The [@iota/persistence-adapter-level](https://github.com/iotaledger/iota.js/tree/next/packages/persistence-adapter-level) adapter is the default. This adapter stores the seed state and seed history in the `leveldown` flavor of the [LevelDB database](http://leveldb.org/). You can customize this adapter to use a different database.

      You can't use one adapter instance for multiple accounts at the same time. A private adapter is created for each new account.
      :::

4. **Optional** Pass a `timeSource` method that outputs the current time in milliseconds to your `account` object

      ```js
      const account = createAccount({
            seed,
            provider,
            timeSource: () => {
                  // Get time with NTP
                  // const time = ...
                  return time
            }
      })
      ```

:::success:Congratulations! :tada:
You've created an account that will automatically promote and reattach transactions as well as manage the state of your CDAs.
:::