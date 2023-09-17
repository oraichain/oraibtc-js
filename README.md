<div align="center">
  
<img src="nbtc.svg" alt="nBTC" width="100" height="100">

  <h1><code>nomic-bitcoin</code></h1>

<strong>A JavaScript library for accepting Bitcoin deposits with Interchain Deposits to any <a
  href="https://cosmos.network">IBC-enabled blockchain,</a> powered by <a
  href="https://nomic.io">Nomic</a>.</strong>

</div>

## Installation

```
npm install nomic-bitcoin
```

## Usage

```typescript
import { generateDepositAddress } from 'nomic-bitcoin'

let depositInfo = await generateDepositAddress({
  relayers: ['https://testnet-relayer.nomic.io:8443'],
  channel: 'channel-0', // IBC channel ID on Nomic
  network: 'testnet',
  receiver: 'cosmos1...', // bech32 address of the depositing user
})

console.log(depositInfo)
/*
{
  code: 0,
  bitcoinAddress: "tb1q73yhgsjedp2uuwjew6zcj0kurryyue2zqjdgn5g5cf7w4krwgtusgsmpku",
  expirationTimeMs: 1624296000000,
  bridgeFeeRate: 0.015,
  minerFeeRate: 0.0001,
  qrCodeData: "data:image/svg+xml;base64,...",
}
*/
```

Bitcoin sent to `bitcoinAddress` before the expiration date will be automatically IBC-transferred over the specified channel and should appear in the user's account with no further interaction required.


![QR code example](qr-code-styling.png)


### Capacity limit

The bridge currently has a capacity limit, which is the maximum amount of BTC that can be held in the bridge at any given time. When the capacity limit is reached, relayers will reject newly-broadcasted deposit addresses.

If the bridge is over capacity, the response code in `depositInfo` will be `2`.

```typescript
let depositInfo = await generateDepositAddress(opts)
if(depositInfo.code === 2) {
  let { capacity, utilization } = depositInfo
  console.error(`Capacity limit reached`)
}
```

Partner chains should communicate clearly to the user that a deposit address could not be safely generated because the bridge is currently over capacity.

### Deposit address expiration

When a deposit address is successfully generated, an expiration time in milliseconds is returned in `depositInfo`.

```typescript
let depositInfo = await generateDepositAddress(opts)
if (depositInfo.code === 0) {
  let { expirationTimeMs, bitcoinAddress } = depositInfo
  console.log(
    `Deposit address ${bitcoinAddress} expires at ${expirationTimeMs}`
  )
}
```

It is critical that the user understands that deposits to this Bitcoin address **will be lost** if they are sent after the expiration time. Addresses typically expire 4-5 days after creation. Do not save the address for later use, and warn the user not to reuse the address, even though multiple deposits to the same address will work as expected before the address expires.

### Fee rates

The Nomic bridge will deduct a fee from incoming deposits. The fee rate is currently a percentage of the deposit amount, and is returned in `depositInfo`.

```typescript
let depositInfo = await generateDepositAddress(opts)
if (depositInfo.code === 0) {
  let { feeRate, bitcoinAddress } = depositInfo
  console.log(
    `The fee rate for deposits to ${bitcoinAddress} is ${feeRate * 100}%`
  )
}
```

Additionally, a small fixed fee will deducted by Bitcoin miners before the deposit is processed.

These fees should be communicated clearly to the user as "Bridge fee" (a percentage) and "Bitcoin miner fee" respectively.

### Usage guidelines

- Include multiple relayers. See [the Nomic docs](https://github.com/nomic-io/nomic) for information about running a relayer.
- Display a deposit address QR code on desktop for mobile Bitcoin wallets.
- Communicate bridge and miner fees.
- Display the deposit address expiration time.

## Status

Interchain Deposits and Withdrawls will be enabled in a release pushing October 27, 2023 with an estimated activation on October 30, 2023.
