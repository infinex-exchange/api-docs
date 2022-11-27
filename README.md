
# Introduction

 - The base endpoint is: **https://api.infinex.cc**
 - All requests should be sent in JSON format using the POST method
 - Some endpoints will require an API Key, which can be generated in the Infinex account settings
 - **Never share your API key to ANYONE**

# Error handling

Each API response contains a `success` (boolean) field. When `success` is `false`, the `error` (string) field with the error description is also included.

Sample response for success:
```javascript
{
	"success": true
}
```

Sample response for error:
```javascript
{
	"success": false,
	"error": "Missing data"
}
```

# Contents
 * [Wallet endpoints](#wallet-endpoints)
   - [Assets list](#assets-list)
   - [Networks list](#networks-list)
   - [Wallet balances](#wallet-balances)
   - [Wallet transactions](#wallet-transactions)
   - [Get deposit address](#get-deposit-address)
   - [Get withdrawal informations](#get-withdrawal-informations)
   - [Validate withdrawal address](#validate-withdrawal-address)
   - [Request withdrawal](#request-withdrawal)
   - [Query address book](#query-address-book)
   - [Rename address book item](#rename-address-book-item)
   - [Delete address book item](#delete-address-book-item)
* [Spot exchange endpoints](#spot-exchange-endpoints)
  - [Markets list and price tickers](#markets-list-and-price-tickers)
  - [Aggregated order book](#aggregated-order-book)
  - [Market trades](#market-trades)
  - [Candlestick / K-Lines / OHLCV data](#candlestick--k-lines--ohlcv-data)
  - [My open orders](#my-open-orders)
  - [My orders history](#my-orders-history)
  - [My trades history](#my-trades-history)
  - [Post new order](#post-new-order)
  - [Cancel order](#cancel-order)

# Wallet endpoints

## Assets list

```http
POST /wallet/assets
```
Get list of assets supported by exchange
This endpoints returns a maximum of 50 records. Use the offset field to get the next part of data.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `symbols` | `array<string>` | *Optional*. Provide asset symbols if you want to get informations only about these assets |
| `search` | `string` | *Optional*. Provide arbitrary string to search in assets list |
| `offset` | `int` | *Optional*. Number of rows to skip. |

 - `offset` is required if no `symbols` is set
 - Only one of `symbols` or `search` can be specified in single request

**Response:**
`assets` - associative array of objects, indexed by asset symbols. The asset objects contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `name` | `string` | Asset full name |
| `icon_url` | `string` | Asset icon URL (svg or png file) |

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/assets -H 'Content-Type: application/json' -d '{"offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "assets": {
        "BCH": {
            "name": "Bitcoin Cash",
            "icon_url": "/ico/bch.svg"
        },
        "BPX": {
            "name": "BPX",
            "icon_url": "/ico/bpx.svg"
        },
        "BSV": {
            "name": "Bitcoin SV",
            "icon_url": "/ico/bsv.svg"
        },
        "BTC": {
            "name": "Bitcoin",
            "icon_url": "/ico/btc.svg"
        }
    ]
}
```

## Networks list

```http
POST /wallet/networks
```
Get list of networks related to given asset

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `asset` | `string` | **Required**. Asset symbol |

**Response:**
`networks` - associative array of objects, indexed by network symbols. The network objects contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `description` | `string` | Network description |
| `icon_url` | `string` | Network icon URL (svg or png file) |

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/networks -H 'Content-Type: application/json' -d '{"asset": "USDT"}'
```

**Response example:**
```
{
    "success": true,
    "networks": {
        "ETH": {
            "description": "Ethereum / ERC20",
            "icon_url": "/ico/eth.svg"
        },
        "TRX": {
	        "description": "Tron / TRC20",
	        "icon_url": "/ico/trx.svg"
        }
    }
}
```

## Wallet balances

```http
POST /wallet/balances
POST /wallet/balances_ex
```
Get user account balances.
This endpoints returns a maximum of 50 records. Use the offset field to get the next part of data.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `symbols` | `array<string>` | *Optional*. Provide asset symbols if you want to get informations only about these assets |
| `search` | `string` | *Optional*. Provide arbitrary string to search in assets list |
| `offset` | `int` | *Optional*. Number of rows to skip. |

 - `offset` is required if no `symbols` is set
 - Only one of `symbols` or `search` can be specified in single request

**Response:**
`balances` - associative array of objects, indexed by asset symbols. The balance objects contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `total` | `string` | Total account balance |
| `locked` | `string` | Locked amount (e.g. open spot orders, pending withdrawals) |
| `avbl` | `string` | Balance available to spend (avbl = total - locked) |

`balances_ex` endpoint includes additional data and it is a combination of `/wallet/assets` and `/wallet/balances` endpoints:

| Field | Type | Description |
| :--- | :--- | :--- |
| `name` | `string` | Asset full name |
| `icon_url` | `string` | Asset icon URL (svg or png file) |

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/balances -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "balances": {
        "BTC": {
            "total": "899999856.2147932498",
            "locked": "0",
            "avbl": "899999856.2147932498"
        },
        "TRX": {
            "total": "900101825.5753",
            "locked": "0",
            "avbl": "900101825.5753"
        },
        "ETC": {
            "total": "100000",
            "locked": "0",
            "avbl": "100000"
        },
        "XCH": {
            "total": "99914.283",
            "locked": "0",
            "avbl": "99914.283"
        }
    ]
}
```

## Wallet transactions

```http
POST /wallet/transactions
```
Query the wallet transactions history.
This endpoints returns a maximum of 50 records. Use the offset field to get the next part of data.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `offset` | `int` | **Required**. Number of rows to skip. |
| `asset` | `string` | *Optional*. Asset symbol to filter only this asset transactions. |
| `type` | `string` | *Optional*. Transaction type to filter transactions: `DEPOSIT` or `WITHDRAWAL`. |
| `status` | `string` | *Optional*. Transaction status to filter transactions: `PENDING` or `DONE`. |

**Response:**
`transactions` - array of transaction objects which contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `xid` | `int` | Transaction unique id. |
| `type` | `string` | `DEPOSIT` or `WITHDRAWAL` |
| `asset` | `string` | Asset symbol |
| `network` | `string` | Network symbol |
| `amount` | `string` | Transaction amount |
| `status` | `string` | `PENDING` or `DONE` |
| `create_time` | `string` | Unix timestamp of transaction |
| `address` | `string` | Deposit / withdrawal address |
| `network_description` | `string` | Description of transaction network |
| `icon_url` | `string` | Asset icon url |
| `memo` | `string` | Transaction memo / routing key. **Optional: only for networks which requires memo (like Ripple)** |
| `exec_time` | `string` | Unix timestamp. For deposit it's confirmation time. For withdrawal it's execute time. **Optional: only if transaction is confirmed / executed** |
| `confirms` | `int` | Current confirmations count. **Optional: only if transaction type is deposit and transaction status is pending** |
| `confirms_target` | `int` | Target confirmations count. **Optional: only if transaction type is deposit and transaction status is pending** |
| `txid` | `string` | Blockchain transaction id. **Optional: absent if transaction type is withdrawal and transaction status is pending** |
| `height` | `int` | Deposit transaction block height. **Optional: absent if transaction type is not deposit** |
| `wd_fee_this` | `string` | Withdrawal fee. **Optional: absent if transaction type is not withdrawal or transaction status is pending** |
| `memo_name` | `string` | If additional information is needed for the deposit / withdrawal in addition to the address, here is the name of this information (`Memo` / `Routing Key`), to be displayed to the user. **Optional: absent if additional information is not needed** |

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/transactions -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "transactions": [
        {
            "xid": 12,
            "type": "WITHDRAWAL",
            "asset": "BPX",
            "network": "BPX",
            "amount": "2642.10169065",
            "status": "PENDING",
            "create_time": "1653485615.879955",
            "address": "bpx1000000000000000000000000000000000000000000",
            "network_description": "BPX",
            "fee": "0.0002",
            "icon_url": "/ico/bpx.svg"
        }
    ]
}
```

## Get deposit address

```http
POST /wallet/deposit
```
Get account deposit address for given asset and network.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `asset` | `string` | **Required**. Asset symbol. |
| `network` | `string` | **Required**. Network symbol. |

**Response:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `operating` | `bool` | `true` if there are no problems with connection to given network. `false` if the full node of the exchange does not respond to the ping or is out of sync. |
| `confirms_target` | `int` | Number of confirmations required to approve a deposit. |
| `address` | `string` | Deposit address |
| `memo` | `string` | Transaction memo / routing key. **Optional: only for networks which requires memo (like Ripple)** |
| `memo_name` | `string` | If additional information is needed for the deposit / withdrawal in addition to the address, here is the name of this information (`Memo` / `Routing Key`), to be displayed to the user. **Optional: absent if additional information is not needed** |
| `qr_content` | `string` | Full payment URL for QR code to be scanned by the wallet. **Optional: If the given network does not support the payment URL, this field is missing. You can generate a QR code containing just the wallet address.** |

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/deposit -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "asset": "BPX", "network": "BPX"}'
```

**Response example:**
```
{
    "success": true,
    "operating": true,
    "confirms_target": 32,
    "address": "bpx1000000000000000000000000000000000000000000",
    "qr_content": "bpx_addr://bpx1000000000000000000000000000000000000000000"
}
```

## Get withdrawal informations

```http
POST /wallet/withdraw/info
```
Get the informations you need before withdrawal.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `asset` | `string` | **Required**. Asset symbol. |
| `network` | `string` | **Required**. Network symbol. |

**Response:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `operating` | `bool` | `true` if there are no problems with connection to given network. `false` if the full node of the exchange does not respond to the ping or is out of sync. |
| `fee_min` | `string` | Minimal withdrawal fee. |
| `fee_max` | `string` | Maximal withdrawal fee. |
| `prec` | `int` | Number of decimal places for withdrawal amount. |
| `memo_name` | `string` | If additional information is needed for the deposit / withdrawal in addition to the address, here is the name of this information (`Memo` / `Routing Key`), to be displayed to the user. **Optional: absent if additional information is not needed** |

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/withdraw/info -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "asset": "BPX", "network": "BPX"}'
```

**Response example:**
```
{
    "success": true,
    "operating": true,
    "fee_min": "0",
    "fee_max": "0.0005",
    "prec": 12
}
```

## Validate withdrawal address

```http
POST /wallet/withdraw/validate
```
Validate withdrawal address (and Memo / Routing Key if needed).

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `asset` | `string` | **Required**. Asset symbol. |
| `network` | `string` | **Required**. Network symbol. |
| `address` | `string` | *Optional*. Address to validate. |
| `memo` | `string` | *Optional*. Memo / routing key to validate. |

 - One of `address` or `memo` or must be specified
 - Both `address` and `memo` can be specified

**Response:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `valid_address` | `bool` | Is address valid. **Optional: absent if `address` not specified in request** |
| `valid_memo` | `bool` | Is memo / routing key valid. **Optional: absent if `memo` not specified in request** |

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/withdraw/validate -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "asset": "BPX", "network": "BPX", "address": "AAAAAAAAAAAAAA"}'
```

**Response example:**
```
{
	"success": true,
    "valid_address": false
}
```

## Request withdrawal

```http
POST /wallet/withdraw
```
Request funds withdrawal.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `asset` | `string` | **Required**. Asset symbol. |
| `network` | `string` | **Required**. Network symbol. |
| `address` | `string` | **Required**. Withdrawal wallet address. |
| `amount` | `string` | **Required**. Withdrawal amount. |
| `fee` | `string` | **Required**. Withdrawal fee amount. |
| `memo` | `string` | *Optional*. Withdrawal wallet memo / routing key if needed. |
| `adbk_name` | `string` | *Optional*. If a name is entered here, the withdrawal address will be saved in the address book under that name. |

**Response:**

`xid` - unique wallet transaction ID

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/withdraw -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "asset": "BPX", "network": "BPX", "address": "AAAAAAAAAAAAAA", "amount": "5.5", "fee": "0", "adbk_name": "my wallet"}'
```

**Response example:**
```
{
	"success": true,
    "xid": 2510
}
```

## Query address book

```http
POST /wallet/addressbook
```
Query users address book.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `asset` | `string` | *Optional*. Asset symbol to filter only this asset addresses. |
| `network` | `string` | *Optional*. Network symbol to filter only this network addresses. |

 - `asset` and `network` must be specified both or neither

**Response:**
`addressbook` - associative array of address book objects, indexed by unique `adbkid`. Address book object contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `asset` | `string` | Asset symbol |
| `network` | `string` | Network symbol |
| `address` | `string` | Saved address |
| `name` | `string` | Saved address name |
| `icon_url` | `string` | Asset icon url |
| `network_description` | `string` | User friendly name of network |
| `memo` | `string` | Saved memo / routing key. **Optional: only for networks which requires memo (like Ripple)** |
| `memo_name` | `string` | If memo / routing key is saved with the address, here is the name of this information (`Memo` / `Routing Key`), to be displayed to the user. **Optional: absent if additional information is not needed** |

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/addressbook -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000"}'
```

**Response example:**
```
{
    "success": true,
    "addressbook": {
        "3": {
            "asset": "BPX",
            "network": "BPX",
            "address": "bpx1000000000000000000000000000000000000000000",
            "name": "my wallet",
            "icon_url": "/ico/bpx.svg",
            "network_description": "BPX"
        }
    }
}
```

## Rename address book item

```http
POST /wallet/addressbook/rename
```
Rename address book item.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `adbkid` | `int` | **Required**. Unique address book item ID. |
| `new_name` | `string` | **Required**. New name. |

**Response:**
Only success/error.

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/addressbook/rename -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "adbkid": 55, "new_name": "mobile wallet"}'
```

**Response example:**
```
{
    "success": true
}
```

## Delete address book item

```http
POST /wallet/addressbook/delete
```
Delete address book item.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `adbkid` | `int` | **Required**. Unique address book item ID. |

**Response:**
Only success/error.

**Request example:**
```
curl -X POST https://api.infinex.cc/wallet/addressbook/delete -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "adbkid": 55}'
```

**Response example:**
```
{
    "success": true
}
```

# Spot exchange endpoints

## Markets list and price tickers

```http
POST /spot/markets
POST /spot/markets_ex
```
Get information about the trading markets with current prices.
`markets_ex` provide more detailed data.
This endpoints returns a maximum of 50 records. Use the offset field to get the next part of data.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `pair` | `string` | *Optional*. Provide pair symbol if you want to get informations only about single pair |
| `quote` | `string` | *Optional*. Provide quote asset symbol if you want to get informations only about pair quoted to this asset |
| `search` | `string` | *Optional*. Provide arbitrary string to search in markets list |
| `sort` | `string` | *Optional*. Sort response items by: `name`, `price`, `change` or `volume` |
| `sort_dir` | `string` | *Optional*. Sort direction: `asc` or `desc` |
| `offset` | `int` | *Optional*. Number of rows to skip. |

 - `offset` is required if no `pair` is set
 - Only one of `pair`, `quote`, `search` can be specified in single request
 - `sort` can't be used if `pair` is specified

**Response:**
`markets` - array of market objects which contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `pair` | `string` | Pair symbol |
| `base` | `string` | Base asset symbol |
| `quote` | `string` | Quote asset symbol |
| `icon_url` | `string` | Base asset icon URL |
| `price` | `string` | Current market price |
| `change` | `string` | Percentage price change in last 24 hours, can be negative |
| `previous` | `string` | Previous market price, can be used to determinate color of displayed price |

`markets_ex` endpoint includes additional data:

| Field | Type | Description |
| :--- | :--- | :--- |
| `base_name` | `string` | Full name of base asset |
| `quote_name` | `string` | Full name of quote asset |
| `base_precision` | `int` | Number of decimal places of base asset |
| `quote_precision` | `int` | Number of decimal places of quote asset |
| `high` | `string` | Highest price for last 24 hours |
| `low` | `string` | Lowest price for last 24 hours |
| `vol_base` | `string` | 24 hours trading volume in base asset |
| `vol_quote` | `string` | 24 hours trading volume in quote asset |
| `min_order` | `string` | Minimal order quantity in quote asset |

**Request example:**
```
curl -X POST https://api.infinex.cc/spot/markets_ex -H 'Content-Type: application/json' -d '{"offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "markets": [
        {
            "pair": "BPX/USDT",
            "base": "BPX",
            "quote": "USDT",
            "icon_url": "/ico/bpx.svg",
            "price": "0.001",
            "change": "0",
            "previous": "0.001",
            "base_name": "BPX",
            "quote_name": "Tether",
            "base_precision": 2,
            "quote_precision": 6,
            "high": "0.001",
            "low": "0.001",
            "vol_base": "0",
            "vol_quote": "0",
            "min_order": "3"
        },
        {
            "pair": "BTC/BPX",
            "base": "BTC",
            "quote": "BPX",
            "icon_url": "/ico/btc.svg",
            "price": "2.34",
            "change": "0",
            "previous": "2.34",
            "base_name": "Bitcoin",
            "quote_name": "BPX",
            "base_precision": 3,
            "quote_precision": 1,
            "high": "2.34",
            "low": "2.34",
            "vol_base": "0",
            "vol_quote": "0",
            "min_order": "10"
        }
    ]
}
```

## Aggregated order book

```http
POST /spot/orderbook
```
Query the spot exchange orderbook

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `pair` | `string` | **Required**. Trading pair symbol |

**Response:**
`bids` and `asks` - arrays of orderbook row objects which contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `price` | `string` | Price level |
| `amount` | `string` | Aggregated orders amount |

**Request example:**
```
curl -X POST https://api.infinex.cc/spot/orderbook -H 'Content-Type: application/json' -d '{"pair": "BPX/USDT"}'
```

**Response example:**
```
{
    "success": true,
    "bids": [
        {
            "price": "0.05",
            "amount": "474755.0000"
        },
        {
            "price": "0.02",
            "amount": "150.0000"
        }
    ],
    "asks": [
        {
            "price": "0.06",
            "amount": "1890.1668"
        },
        {
            "price": "0.07",
            "amount": "2500.0000"
        },
        {
            "price": "0.10",
            "amount": "10000.0000"
        }
    ]
}
```

## Market trades

```http
POST /spot/trades
```
Query the spot exchange market trades history.
This endpoints returns a maximum of 50 records. Use the offset field to get the next part of data.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `pair` | `string` | **Required**. Trading pair symbol. |
| `offset` | `int` | **Required**. Number of rows to skip. |

**Response:**
`trades` - array of trade objects which contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `time` | `string` | Unix timestamp |
| `price` | `string` | Trade price |
| `amount` | `string` | Trade amount in base asset |
| `total` | `string` | Trade amount in quote asset |
| `side` | `string` | `BUY` or `SELL`* |

***Each trade is both a sale and a purchase, but this field contains information about what type of order triggered this trade. This information is needed to set the proper color of the text.**

**Request example:**
```
curl -X POST https://api.infinex.cc/spot/trades -H 'Content-Type: application/json' -d '{"pair": "BPX/USDT", "offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "trades": [
        {
            "time": "1653823541.689083",
            "price": "0.06",
            "amount": "166.6666",
            "total": "10.00",
            "side": "BUY"
        },
        {
            "time": "1653780231.798744",
            "price": "0.06",
            "amount": "53.0000",
            "total": "3.18",
            "side": "BUY"
        },
        {
            "time": "1653741501.810590",
            "price": "0.06",
            "amount": "69.1666",
            "total": "4.15",
            "side": "BUY"
        }
    ]
}
```

## Candlestick / K-Lines / OHLCV data

```http
POST /spot/candlestick
```
Query the spot exchange Open, High, Low, Close, Volume data.
This endpoints returns a maximum of 500 records. Use the `from` and `to` fields to get next part of data.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `pair` | `string` | **Required**. Trading pair symbol. |
| `res` | `string` | **Required**. Time buckets resolution: `1` - 1 minute, `60` - 1 hour, `1D` - 1 day. |
| `from` | `int` | **Required**. Unix timestamp. Fetch the data from this point in time. |
| `to` | `int` | **Required**. Unix timestamp. Fetch the data to this point in time. |

**Response:**
`candlestick` - array of candlestick objects which contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `time` | `string` | Time bucket unix timestamp |
| `open` | `string` | Open price of time bucket |
| `high` | `string` | Highest price in time bucket |
| `low` | `string` | Lowest price in time bucket |
| `close` | `string` | Close price of time bucket |
| `volume` | `string` | Trading volume during time bucket |

**Request example:**
```
curl -X POST https://api.infinex.cc/spot/candlestick -H 'Content-Type: application/json' -d '{"pair": "BPX/USDT", "res": "1D", "from": 1641031490, "to": 1656673495}'
```

**Response example:**
```
{
    "success": true,
    "candlestick": [
        {
            "time": "1653177600.000000",
            "open": "20",
            "high": "1515",
            "low": "20",
            "close": "125",
            "volume": "8543.0098"
        },
        {
            "time": "1653264000.000000",
            "open": "147",
            "high": "1514.9",
            "low": "1.01",
            "close": "1.01",
            "volume": "967.8646"
        }
    ]
}
```

## My open orders

```http
POST /spot/open_orders
```
Get list of users open orders.
This endpoints returns a maximum of 50 records. Use the `offset` field to get next part of data.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `offset` | `int` | **Required**. Number of rows to skip. |
| `filter_pair` | `string` | *Optional*. Provide pair symbol to filter only this pair orders. |
| `sort_pair` | `string` | *Optional*. Provide pair symbol to push orders of this pair to the top of the list. |

 - `filter_pair` and `sort_pair` can't be used at once

**Response:**
`orders` - array of order objects which contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `obid` | `int` | Unique order ID |
| `pair` | `string` | Pair symbol |
| `time` | `string` | Unix timestamp |
| `side` | `string` | `BUY` or `SELL` |
| `price` | `string` | Order price |
| `amount` | `string` | Order amount |
| `time_in_force` | `string` | `GTC`, `FOK` or `IOC` - order time in force setting |
| `type` | `string` | Order type: `LIMIT`, `MARKET` or `STOP_LIMIT` |
| `quote_prec` | `string` | Number of decimal places of quote asset |
| `filled` | `string` | Filled order amount. **Optional: does not occur for a stop limit order that has not yet been triggered** |
| `stop` | `string` | Stop price. **Optional: does not occur for orders other than stop limit** |

**Request example:**
```
curl -X POST https://api.infinex.cc/spot/open_orders -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "orders": [
        {
            "obid": 903,
            "pair": "DASH/BTC",
            "time": "1653821715.093741",
            "side": "SELL",
            "price": "0.0000000003",
            "amount": "10",
            "filled": "0",
            "time_in_force": "GTC",
            "type": "LIMIT",
            "quote_prec": 10
        }
    ]
}
```

## My orders history

```http
POST /spot/orders_history
```
Get list of users orders history.
This endpoints returns a maximum of 50 records. Use the `offset` field to get next part of data.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `offset` | `int` | **Required**. Number of rows to skip. |
| `filter_pair` | `string` | *Optional*. Provide pair symbol to filter only this pair orders. |

**Response:**
`orders` - array of order objects which contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `trades` | `array<object>` | Trades related to this order. |
| `obid` | `int` | Unique order ID |
| `pair` | `string` | Pair symbol |
| `time` | `string` | Unix timestamp |
| `side` | `string` | `BUY` or `SELL` |
| `time_in_force` | `string` | `GTC`, `FOK` or `IOC` - order time in force setting |
| `type` | `string` | Order type: `LIMIT`, `MARKET` or `STOP_LIMIT` |
| `status` | `string` | Order status: `OPEN`, `FILLED`, `CANCELED` or `KILLED` |
| `quote_prec` | `string` | Number of decimal places of quote asset |
| `base` | `string` | Base asset symbol |
| `quote` | `string` | Quote asset symbol |
| `price` | `string` | Order price. **Optional: does not occur for market price orders** |
| `amount` | `string` | Order amount. **Optional: does not occur for market orders, when the user has specified the total value he wants to receive instead of amount** |
| `total` | `string` | Order total value. **Optional: only in market orders based on total value instead of amount** |
| `filled` | `string` | Filled order amount. **Optional: does not occur for a stop limit order that has not yet been triggered** |
| `stop` | `string` | Stop price. **Optional: does not occur for orders other than stop limit** |

`trades` is an array of trade objects which contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `time` | `string` | Unix timestamp |
| `fee` | `string` | Maker/taker fee amount |
| `price` | `string` | Trade price |
| `amount` | `string` | Trade amount |
| `total` | `string` | Trade total value |
| `role` | `string` | `MAKER` or `TAKER` |

**Request example:**
```
curl -X POST https://api.infinex.cc/spot/orders_history -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "orders": [
        {
            "trades": [
                {
                    "time": "1653823558.258593",
                    "fee": "0.00000027",
                    "price": "25500",
                    "amount": "0.0003",
                    "total": "7.65",
                    "role": "TAKER"
                }
            ],
            "obid": 920,
            "pair": "BTC/USDT",
            "time": "1653823558.243453",
            "side": "BUY",
            "total": "10",
            "filled": "0.0003",
            "time_in_force": "FOK",
            "type": "MARKET",
            "status": "FILLED",
            "quote_prec": 6,
            "base": "BTC",
            "quote": "USDT"
        },
        {
            "trades": [],
            "obid": 903,
            "pair": "DASH/BTC",
            "time": "1653821715.093741",
            "side": "SELL",
            "price": "0.0000000003",
            "amount": "10",
            "filled": "0",
            "time_in_force": "GTC",
            "type": "LIMIT",
            "status": "OPEN",
            "quote_prec": 10,
            "base": "DASH",
            "quote": "BTC"
        }
    ]
}
```

## My trades history

```http
POST /spot/trades_history
```
Get users trading history.
This endpoints returns a maximum of 50 records. Use the `offset` field to get next part of data.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `offset` | `int` | **Required**. Number of rows to skip. |
| `filter_pair` | `string` | *Optional*. Provide pair symbol to filter only this pair orders. |

**Response:**
`trades` - array of trade objects which contains following fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `time` | `string` | Unix timestamp |
| `pair` | `string` | Pair symbol |
| `fee` | `string` | Maker/taker fee amount |
| `price` | `string` | Trade price |
| `amount` | `string` | Trade amount |
| `total` | `string` | Trade total value |
| `role` | `string` | `MAKER` or `TAKER` |
| `side` | `string` | `BUY` or `SELL` |

**Request example:**
```
curl -X POST https://api.infinex.cc/spot/trades_history -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "trades": [
        {
            "time": "1653823558.258593",
            "pair": "BTC/USDT",
            "fee": "0.00000027",
            "price": "25500",
            "amount": "0.0003",
            "total": "7.65",
            "role": "TAKER",
            "side": "BUY"
        },
        {
            "time": "1653823555.164274",
            "pair": "BTC/USDT",
            "fee": "0.00000027",
            "price": "25500",
            "amount": "0.0003",
            "total": "7.65",
            "role": "TAKER",
            "side": "BUY"
        },
        {
            "time": "1653823552.044859",
            "pair": "BTC/USDT",
            "fee": "0.00000027",
            "price": "25500",
            "amount": "0.0003",
            "total": "7.65",
            "role": "TAKER",
            "side": "BUY"
        }
    ]
}
```

## Post new order

```http
POST /spot/open_orders/new
```
Post new spot order.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `pair` | `string` | **Required**. Pair symbol. |
| `side` | `string` | **Required.** `BUY` or `SELL`  |
| `type` | `string` | **Required.** `MARKET`, `LIMIT` or `STOP_LIMIT`  |
| `time_in_force` | `string` | **Required.** `GTC`, `FOK` or `IOC`  |
| `price` | `string` | *Optional.* Order price. Not required for market price order. |
| `amount` | `string` | *Optional.* Order amount. Not required for market price order if `total` was provided. |
| `total` | `string` | *Optional.* Order total value. Not required for limit and stop limit orders, and for market order if `amount` was provided. |
| `stop` | `string` | *Optional.* Order stop price. Not required for orders other than stop limit. |

**Response:**
This endpoint returns success if the order has been correctly sent to the Matching Engine. Any errors occured during order processing by the matching engine will not be included in the response of this function. Success response does not guarantee that the order was accepted. You should subscribe to the event stream from the matching engine.

**Request example:**
```
curl -X POST https://api.infinex.cc/spot/open_orders/new -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "pair": "BPX/USDT", "side": "BUY", "type": "MARKET", "time_in_force": "FOK", "amount": "100.50"}'
```

**Response example:**
```
{
    "success": true
}
```

## Cancel order

```http
POST /spot/open_orders/cancel
```
Cancel spot order.

**Request parameters:**

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |
| `obid` | `int` | **Required**. Unique order id. |

**Response:**
This endpoint returns success if the order cancel request has been correctly sent to the Matching Engine. Any errors occured during order cancelation processing by the matching engine will not be included in the response of this function. Success response does not guarantee that the order was canceled. You should subscribe to the event stream from the matching engine.

**Request example:**
```
curl -X POST https://api.infinex.cc/spot/open_orders/cancel -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "obid": 20}'
```

**Response example:**
```
{
    "success": true
}
```
