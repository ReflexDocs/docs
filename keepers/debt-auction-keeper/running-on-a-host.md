---
description: Running a debt auction keeper directly on a host
---

# Running on a Host

## Prerequisites

Python 3.6+

### Get RAI

Buy RAI from Uniswap v2 or [open a SAFE](https://app.gitbook.com/@reflexer-labs/s/geb/pyflex/safe-management/opening-a-safe) and generate it.

### Clone

```
git clone https://github.com/reflexer-labs/auction-keeper.git
cd auction-keeper
git submodule update --init --recursive
```

### Install

This creates a virtual environment and installs all the keeper dependencies:

`./install.sh`

## 1) Start virtualenv

`source _virtualenv/bin/activate`

## 2) Modify model file as needed

A basic debt auction bidding model can be found in `models/debt_model.py`. This model retrieves the latest FLX/USD price from coingecko and will automatically place bids in an auction.

You probably want to modify the following variables in `models/debt_model.py`:

`MAXIMUM_FLX_MULTIPLIER`: The maximum acceptable FLX price to use when bidding. Default: `0.90` meaning the maximum price to pay when buying FLX(in RAI) is 90% of the current FLX/USD market price

`MY_BID_DECREASE`: The amount of bid decrease(in FLX) to make when outbidding another bidder. If value is less than the auction house' `bidDecrease`, then it will use the auction house setting. Example: A value of `1.10` will create bid decreases of 10%. Note: Current `bidDecrease` on mainnet is `1.03`. Default: `1.03`

### Then:

`chmod +x debt_model.py`

For more information about bidding models, see [Bidding Models](broken-reference)

## 3) Modify keeper run file

Modify the following variables in `run_debt_keeper_host.sh`

`KEEPER_ADDRESS` - the keeper's address. It should be in checksummed format (not lowercase)

`ETH_RPC_URL` - the URL of your Ethereum RPC connection

`KEYSTORE_DIR` - the full path of the directory where your keystore file is

`MODEL_DIR` - the full path of directory where your `debt_model.py` file is



`KEYSTORE_FILE` - your Ethereum UTC JSON keystore filename

For more information about this keystore format and how to generate them, check:

* [Ethereum UTC / JSON Wallet Encryption](https://wizardforcel.gitbooks.io/practical-cryptography-for-developers-book/content/symmetric-key-ciphers/ethereum-wallet-encryption.html)
* [keythereum](https://github.com/ethereumjs/keythereum)

`GAS_MAXIMUM` -maximum gas price, in GWEI

### Then:

`chmod +x run_debt_keeper_host.sh`

## 4) Start the keeper and enter your keystore file password

`./run_debt_keeper_host.sh`

```
$ ./run_debt_keeper_host.sh
Password for /keystore/key.json:
```

## Debt Auction Output

#### Pop Debt from the Queue

If debt from collateral auctions still exists after `AccountingEngine.pop_debt_delay()`, the surplus _auction-keeper_ will call `popDebtFromQueue()`

```
2021-01-20 04:16:22,462 INFO     Sent transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').popDebtFromQueue(1611116032) with nonce=1519, gas=159390, gas_price=10000000000 (tx_hash=0x7266984842ce52d8df52a775d889bb7040e0bac742d51f369e0c23e3e89dd560)
```

#### Settle Debt

Before starting a debt auction, the keeper will use any surplus to offset any existing debt. It does this by calling `settleDebt()`

```
2021-01-20 04:16:32,983 INFO     Settling debt coin balance=192.720512165128879832575769589252154674198975198 unqueued_enauctioned_debt=686.568045684729064016811194323877100504725525932
2021-01-20 04:16:33,793 INFO     Sent transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').settleDebt(192720512165128879832575769589252154674198975198) with nonce=1520, gas=171354, gas_price=10000000000 (tx_hash=0x92d16108db353e25b70a743eefb84568f640f3ab4241d5cfa43c147126080d4b)
```

#### Starting a Debt Auction

Finally, if debt still exists and is enough to start an auction, the _auction-keeper_ will call `auctionDebt`

```
2021-01-20 04:16:40,596 INFO     Initiating a debt auction with unqueued_unauctioned_debt=493.847533519600184184235424734624945830526550734
2021-01-20 04:16:41,382 INFO     Sent transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').auctionDebt() with nonce=1521, gas=306628, gas_price=1
0000000000 (tx_hash=0x9ff7a02e85a30c361768639729678df615eba7f10a69505c1aaa4cc88818c73d)
```

#### Full Log Output

```
[ec2-user@ip-172-31-40-135 ~]$ ./run_debt_auction_keeper.sh
upstream: Pulling from reflexer/auction-keeper
Digest: sha256:d222e4d8d948262af15fbc09e64973a6df65eb0f96023ea3ec7179674d87f28c
Status: Image is up to date for reflexer/auction-keeper:upstream
docker.io/reflexer/auction-keeper:upstream
Password for /keystore/keystore.json: 
2021-01-20 04:16:01,302 INFO     Keeper connected to RPC connection https://myparitynode.com
2021-01-20 04:16:01,302 INFO     Keeper operating as 0xdD1693BD8E307eCfDbe51D246562fc4109f871f8
2021-01-20 04:16:01,646 INFO     Executing keeper startup logic
2021-01-20 04:16:02,714 INFO     Checking if internal system coin balance needs to be rebalanced
2021-01-20 04:16:02,749 INFO     Joining 1480.613947077334748390 system coin to the SAFE Engine
2021-01-20 04:16:02,816 INFO     Sent transaction <pyflex.gf.CoinJoin object at 0x7f56f75ae358>.join('0xdD1693BD8E307eCfDbe51D246562fc4109f871f8', 1480613947077334748390) with nonce=1518, gas=159975, gas_price=10000000000 (tx_hash=0x79982c65d2aece7a71e4c475e1a90afd8056811dbec6f6a82a844e637192c47e)
2021-01-20 04:16:12,665 INFO     Transaction <pyflex.gf.CoinJoin object at 0x7f56f75ae358>.join('0xdD1693BD8E307eCfDbe51D246562fc4109f871f8', 1480613947077334748390) was successful (tx_hash=0x79982c65d2aece7a71e4c475e1a90afd8056811dbec6f6a82a844e637192c47e)
2021-01-20 04:16:12,666 INFO     Keeper will perform the following operation(s) in parallel:
2021-01-20 04:16:12,666 INFO     --> Check thresholds in Accounting Engine Contract and start new debt auctions once reached
2021-01-20 04:16:12,666 INFO     --> Check all auctions being monitored and evaluate bidding opportunity every 4.0 seconds
2021-01-20 04:16:12,667 INFO     --> Check all auctions and settle for 0xdD1693BD8E307eCfDbe51D246562fc4109f871f8
2021-01-20 04:16:12,671 INFO     Keeper will use Node gas price (currently 10.0 Gwei, changes over time) with initial multiplier 1.0 and will multiply by 1.125 every 42s to a maximum of 2000.0 Gwei for transactions and bids unless model instructs otherwise
2021-01-20 04:16:13,898 INFO     Watching for new blocks
2021-01-20 04:16:13,900 INFO     Started 2 timer(s)
2021-01-20 04:16:22,462 INFO     Sent transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').popDebtFromQueue(1611116032) with nonce=1519, gas=159390, gas_price=10000000000 (tx_hash=0x7266984842ce52d8df52a775d889bb7040e0bac742d51f369e0c23e3e89dd560)
2021-01-20 04:16:32,881 INFO     Transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').popDebtFromQueue(1611116032) was successful (tx_hash=0x7266984842ce52d8df52a775d889bb7040e0bac742d51f369e0c23e3e89dd560)
2021-01-20 04:16:32,983 INFO     Settling debt coin balance=192.720512165128879832575769589252154674198975198 unqueued_enauctioned_debt=686.568045684729064016811194323877100504725525932
2021-01-20 04:16:33,793 INFO     Sent transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').settleDebt(192720512165128879832575769589252154674198975198) with nonce=1520, gas=171354, gas_price=10000000000 (tx_hash=0x92d16108db353e25b70a743eefb84568f640f3ab4241d5cfa43c147126080d4b)
2021-01-20 04:16:40,536 INFO     Transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').settleDebt(192720512165128879832575769589252154674198975198
) was successful (tx_hash=0x92d16108db353e25b70a743eefb84568f640f3ab4241d5cfa43c147126080d4b)
2021-01-20 04:16:40,596 INFO     Initiating a debt auction with unqueued_unauctioned_debt=493.847533519600184184235424734624945830526550734
2021-01-20 04:16:41,382 INFO     Sent transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').auctionDebt() with nonce=1521, gas=306628, gas_price=1
0000000000 (tx_hash=0x9ff7a02e85a30c361768639729678df615eba7f10a69505c1aaa4cc88818c73d)
2021-01-20 04:16:44,491 INFO     Transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').auctionDebt() was successful (tx_hash=0x9ff7a02e85a30c36176
8639729678df615eba7f10a69505c1aaa4cc88818c73d)
2021-01-20 04:16:44,636 INFO     Started monitoring auction #7
2021-01-20 04:16:44,637 INFO     Instantiated model using process '/models/debt_model.sh --id 7 --debt_auction_house 0xBD0E4aC6061Df1eA95CaDfb04707892cCb750531'
2021-01-20 04:16:44,652 INFO     Process '/models/debt_model.sh --id 7 --debt_auction_house 0xBD0E4aC6061Df1eA95CaDfb04707892cCb750531' (pid #35) started
2021-01-20 04:16:44,664 INFO     Checked auctions 0 to 7 in 0 seconds
2021-01-20 04:16:44,977 INFO     Initiating a debt auction with unqueued_unauctioned_debt=408.847533519600184184235424734624945830526550734
2021-01-20 04:16:45,210 INFO     Sent transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').auctionDebt() with nonce=1522, gas=276628, gas_price=1
0000000000 (tx_hash=0xd87e9c0a2c6512340602426b9530c822b6c58b30eaa61021a3c314c7c30fe41c)
2021-01-20 04:16:46,965 INFO     Sending new bid @100.000000000000000000 for auction 7
2021-01-20 04:16:47,020 INFO     Sent transaction DebtAuctionHouse('0xBD0E4aC6061Df1eA95CaDfb04707892cCb750531').decreaseSoldAmount(7, 850000000000000000, 850000000000
00000000000000000000000000000000000) with nonce=1523, gas=234777, gas_price=10000000000 (tx_hash=0xaffbe93a1f901d1a43ed331284cb76f5e01b64cddc2dc1b6978b0473021b8072)
2021-01-20 04:16:53,038 INFO     Transaction AccountingEngine('0x73EC2a627655134886477D10A41275f54556E0e2').auctionDebt() was successful (tx_hash=0xd87e9c0a2c651234060
2426b9530c822b6c58b30eaa61021a3c314c7c30fe41c)
2021-01-20 04:16:53,040 INFO     Transaction DebtAuctionHouse('0xBD0E4aC6061Df1eA95CaDfb04707892cCb750531').decreaseSoldAmount(7, 850000000000000000, 85000000000000000
000000000000000000000000000000) was successful (tx_hash=0xaffbe93a1f901d1a43ed331284cb76f5e01b64cddc2dc1b6978b0473021b8072)
2021-01-20 04:16:53,216 INFO     Started monitoring auction #8
2021-01-20 04:16:53,216 INFO     Instantiated model using process '/models/debt_model.sh --id 8 --debt_auction_house 0xBD0E4aC6061Df1eA95CaDfb04707892cCb750531'
2021-01-20 04:16:53,236 INFO     Process '/models/debt_model.sh --id 8 --debt_auction_house 0xBD0E4aC6061Df1eA95CaDfb04707892cCb750531' (pid #44) started
2021-01-20 04:16:53,252 INFO     Checked auctions 0 to 8 in 0 seconds
2021-01-20 04:16:54,094 INFO     Initiating a debt auction with unqueued_unauctioned_debt=3
```
