---
description: Intro guide to using GEB.js
---

# Getting Started with GEB.js

GEB.js is a library used to interact with the GEB smart contracts. Developers can manage their SAFEs, mint RAI, inspect the system state and much more.

The library is written in Typescript and has full typing support. It gives access to a low level API used to directly interact with GEB contracts.

## Install

```text
npm install geb.js
```

## Dependencies

At the moment, GEB.js uses [Ether.js](https://www.npmjs.com/package/ethers) V5. In the future it will also support Web3.js.

```text
npm install ethers
```

## Examples

Deploy a GEB proxy:

```typescript
const tx = geb.deployProxy()
await wallet.sendTransaction(tx)
```

This is how you can inspect a SAFE and also open a new one using your own proxy:

```typescript
import { ethers, utils as ethersUtils } from 'ethers'
import { Geb, utils } from 'geb.js'

// Setup Ether.js
const provider = new ethers.providers.JsonRpcProvider(
    'http://kovan.infura.io/<API KEY>'
)
const wallet = new ethers.Wallet('0xdeadbeef...', provider)

// Create the main GEB object
const geb = new Geb('kovan', provider)

// Get a SAFE
const safe = await geb.getSafe(4)
console.log(`Safe id 4 has: ${utils.wadToFixed(safe.debt).toString()} RAI of debt.`)
console.log(`It will get liquidated if ETH price falls below ${(await safe.liquidationPrice())?.toString()} USD.`)

// Open a new SAFE, lock ETH and draw RAI in a single transaction using a proxy
// Note: Before doing this you need to create your own proxy
const proxy = await geb.getProxyAction(wallet.address)
const tx = await proxy.openLockETHAndGenerateDebt(
    ethersUtils.parseEther('1'), // Lock 1 Ether
    utils.ETH_A,                 // Of collateral ETH-A
    ethersUtils.parseEther('15') // And draw 15 RAI
)

tx.gasPrice = ethers.BigNumber.from('80').mul('1000000000') // Set the gas price to 80 Gwei
const pending = await wallet.sendTransaction(tx) // Send the transaction
console.log(`Transaction ${pending.hash} waiting to be mined...`)
await pending.wait() // Wait for it to be mined
console.log('Transaction mined, safe opened!')
```

Use the low level API to make direct contract calls and fetch data:

```typescript
// Fetch some system parameters from their respective contracts
const surplusBuffer = await geb.contracts.accountingEngine.surplusBuffer()
const { stabilityFee } = await geb.contracts.taxCollector.collateralTypes(utils.ETH_A)

// Liquidate a Safe
const tx = geb.contracts.liquidationEngine.liquidateSAFE(utils.ETH_A, "0xdefidream...");
await wallet.sendTransaction(tx)
```

Use multicall to bundle RPC requests into one:

```typescript
const [ globalDebt, collateralInfo ] = await geb.multiCall([
    geb.contracts.safeEngine.globalDebt(true), // !! Note the last parameter set to true.
    geb.contracts.safeEngine.collateralTypes(utils.ETH_A, true),
])
```
