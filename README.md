# Introduction

 - The base endpoint is: **https://api.vayamos.cc**
 - All requests should be sent in JSON format using the POST method
 - Some endpoints will require an API Key, which can be generated in the Vayamos account settings
 - **Never share your API key/secret key to ANYONE**

# Error handling

Each API response contains a `success` (boolean) field. When `success` is `false`, the `message` (string) field with the error description is also included.

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
curl -X POST https://api.vayamos.cc/spot/markets -H 'Content-Type: application/json' -d '{"offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "markets": [
        {
            "pair": "BPX\/USDT",
            "base": "BPX",
            "quote": "USDT",
            "icon_url": "\/ico\/bpx.svg",
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
            "pair": "BTC\/BPX",
            "base": "BTC",
            "quote": "BPX",
            "icon_url": "\/ico\/btc.svg",
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
curl -X POST https://api.vayamos.cc/spot/orderbook -H 'Content-Type: application/json' -d '{"pair": "BPX/USDT"}'
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
curl -X POST https://api.vayamos.cc/spot/trades -H 'Content-Type: application/json' -d '{"pair": "BPX/USDT", "offset": 0}'
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
curl -X POST https://api.vayamos.cc/spot/candlestick -H 'Content-Type: application/json' -d '{"pair": "BPX/USDT", "res": "1D", "from": 1641031490, "to": 1656673495}'
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
curl -X POST https://api.vayamos.cc/spot/open_orders -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "orders": [
        {
            "obid": 903,
            "pair": "DASH\/BTC",
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
curl -X POST https://api.vayamos.cc/spot/orders_history -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "offset": 0}'
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
            "pair": "BTC\/USDT",
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
            "pair": "DASH\/BTC",
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
curl -X POST https://api.vayamos.cc/spot/trades_history -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "offset": 0}'
```

**Response example:**
```
{
    "success": true,
    "trades": [
        {
            "time": "1653823558.258593",
            "pair": "BTC\/USDT",
            "fee": "0.00000027",
            "price": "25500",
            "amount": "0.0003",
            "total": "7.65",
            "role": "TAKER",
            "side": "BUY"
        },
        {
            "time": "1653823555.164274",
            "pair": "BTC\/USDT",
            "fee": "0.00000027",
            "price": "25500",
            "amount": "0.0003",
            "total": "7.65",
            "role": "TAKER",
            "side": "BUY"
        },
        {
            "time": "1653823552.044859",
            "pair": "BTC\/USDT",
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
curl -X POST https://api.vayamos.cc/spot/open_orders/new -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "pair": "BPX/USDT", "side": "BUY", "type": "MARKET", "time_in_force": "FOK", "amount": "100.50"}'
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
curl -X POST https://api.vayamos.cc/spot/open_orders/cancel -H 'Content-Type: application/json' -d '{"api_key": "00000000000000000000", "obid": 20}'
```

**Response example:**
```
{
    "success": true
}
```
