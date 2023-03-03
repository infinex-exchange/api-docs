# Introduction

 - The base endpoint is: **wss://stream.infinex.cc**
 - Private streams will require an API Key, which can be generated in the Infinex account settings
 - **Never share your API key to ANYONE**

# Request/response
Each request sent to the WebSocket server must contain a type of operation (`op`) and a unique request ID (`id`). It's best to start with 1 and after each request increase this value by 1.

Sample request:
```javascript
{
	"op": "ping",
	"id": 220
}
```
Each response from the websocket server contains `class`=`resp`, the same `id` as previously sent in the request, and a `success` (boolean) field. When `success` is `false`, the `error` (string) field with the error description is also included.

Sample response for success:
```javascript
{
	"class": "resp",
	"id": 220,
	"success": true
}
```

Sample response for error:
```javascript
{
	"class": "resp",
	"id": 220,
	"success": false,
	"error": "Missing data"
}
```

# Available operations

The WebSocket server supports only 4 basic operations (`op`):

## Ping
Check if the connection is still alive.

**Request parameters:**
None

**Response:**
Only success/error.

**Request example:**
```javascript
{
	"op": "ping",
	"id": 1
}
```

**Response example:**
```javascript
{
	"class": "resp",
	"id": 1,
	"success": true
}
```

## Auth
Log in to be able to subscribe to private streams.

**Request parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `api_key` | `string` | **Required**. Account API key. |

**Response:**
Only success/error.

**Request example:**
```javascript
{
	"op": "auth",
	"api_key": "00000000000000000000000000000",
	"id": 2
}
```

**Response example:**
```javascript
{
	"class": "resp",
	"id": 2,
	"success": true
}
```
## Subscribe
Subscribe to the stream or many streams.

**Request parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `streams` | `string`/`array<string>` | **Required**. Stream names. |

**Response:**
Only success/error.

**Request example:**
```javascript
{
	"op": "sub",
	"streams": [
		"BPX/USDT@tickerEx",
		"BTC/USDT@ticker"
	],
	"id": 3
}
```

**Response example:**
```javascript
{
	"class": "resp",
	"id": 3,
	"success": true
}
```
## Unsubscribe
Unsubscribe from the stream or many streams.

**Request parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `streams` | `string`/`array<string>` | **Required**. Stream names. |

**Response:**
Only success/error.

**Request example:**
```javascript
{
	"op": "unsub",
	"streams": "BPX/USDT@tickerEx",
	"id": 4
}
```

**Response example:**
```javascript
{
	"class": "resp",
	"id": 4,
	"success": true
}
```

# Event streaming

When you subscribe to a stream, the WebSocket server will push updates to you when an event occurs on the stream you subscribed to.

Each such message contains `class`=`data` and the stream name (`stream`).

Sample event message:
```javascript
{
	"class": "data",
	"stream": "BPX/USDT@ticker",
	"pair": "BPX/USDT",
	"price": "0.0123",
	"change": 10,
	"previous": "0.0122"
}
```

# Available streams
* [Market trade](#market-trade)
* [Candlestick](#candlestick)
* [Ticker](#ticker)
* [Extended ticker](#extended-ticker)
* [Order book](#order-book)
* [My orders (private)](#my-orders)
* [My trades (private)](#my-trades)

## Market trade

```
<pair>@marketTrade
```

**Example stream names:**
```
BPX/USDT@marketTrade
BTC/USDT@marketTrade
ETH/BTC@marketTrade
```
You will receive a message about each trade on a selected pair.

**Message fields:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `time` | `int` | UNIX timestamp of trade |
| `price` | `string` | Trade price |
| `amount` | `string` | Trade amount in base asset. |
| `total` | `string` | Trade total value in quote asset. |
| `side` | `string` | Taker `BUY` or `SELL` |
| `pair` | `string` | Trading pair symbol. |

**Example message:**
```
{
	"class": "data",
	"stream": "BPX/USDT@marketTrade",
	"time": 1677849500,
	"price": "0.0123",
	"amount": "100",
	"total": "1.23",
	"pair": "BPX/USDT"
}
```
## Candlestick

```
<pair>@candleStick/<resolution>
```

**Example stream names:**
```
BPX/USDT@candleStick/1
BTC/USDT@candleStick/60
ETH/BTC@candleStick/1D
```
You will receive live OHLCV feed. This stream name contain additional parameter - resolution, which can take values of: `1` - 1 minute, `60` - 1 hour or `1D` - 1 day.

**Message fields:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `time` | `int` | UNIX timestamp of candle start |
| `pair` | `string` | Trading pair symbol. |
| `open` | `string` | Candle open price. |
| `high` | `string` | Candle highest price. |
| `low` | `string` | Candle lowest price. |
| `close` | `string` | Candle close price. |
| `volume` | `string` | Candle volume in base asset. |

**Example message:**
```
{
	"class": "data",
	"stream": "BPX/USDT@candleStick/1D",
	"time": 1677849500,
	"pair": "BPX/USDT",
	"open": "0.0123",
	"close": "0.0215",
	"high": "0.03",
	"low": "0.0123",
	"volume": "15015.3",
}
```
## Ticker

```
<pair>@ticker
```

**Example stream names:**
```
BPX/USDT@ticker
BTC/USDT@ticker
ETH/BTC@ticker
```
You will receive basic information about the market price changes.

**Message fields:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `pair` | `string` | Trading pair symbol. |
| `price` | `string` | New market price |
| `change` | `int` | Percentage price change in last 24 hours |
| `previous` | `string` | Previous market price. |

**Example message:**
```
{
	"class": "data",
	"stream": "BPX/USDT@ticker",
	"pair": "BPX/USDT",
	"price": "0.0123",
	"change": -50,
	"previous": "0.022",
}
```
## Extended ticker

```
<pair>@tickerEx
```

**Example stream names:**
```
BPX/USDT@tickerEx
BTC/USDT@tickerEx
ETH/BTC@tickerEx
```
You will receive all events from `@ticker` stream + some extended informations.

**Message fields:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `pair` | `string` | Trading pair symbol. |
| `price` | `string` | New market price |
| `change` | `int` | Percentage price change in last 24 hours |
| `previous` | `string` | Previous market price. |
| `high` | `string` | The highest price in last 24 hours. |
| `low` | `string` | The lowest price in last 24 hours. |
| `vol_base` | `string` | Last 24 hours volume in base asset. |
| `vol_quote` | `string` | Last 24 hours volume in quote asset. |

**Example message:**
```
{
	"class": "data",
	"stream": "BPX/USDT@ticker",
	"pair": "BPX/USDT",
	"price": "0.0123",
	"change": -50,
	"previous": "0.022",
	"high": "0.031",
	"low": "0.0123",
	"vol_base": "15000.55",
	"vol_quote": "330.01"
}
```
## Order book

```
<pair>@orderBook
```

**Example stream names:**
```
BPX/USDT@orderBook
BTC/USDT@orderBook
ETH/BTC@orderBook
```
You will receive all necessary informations to keep your local orderbook up to date after it has been initially loaded.

**Message fields:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `pair` | `string` | Trading pair symbol. |
| `amount` | `string` | New amount for given price level. A value of zero means that the price level no longer exists. |
| `price` | `string` | Price level |
| `side` | `string` | Orderbook side: `BUY` or `SELL` |

**Example message:**
```
{
	"class": "data",
	"stream": "BPX/USDT@orderBook",
	"pair": "BPX/USDT",
	"amount": "0",
	"price": "0.022",
	"side": "SELL"
}
```


## My orders
**This stream is private. You cannot subscribe it before performing the `auth` operation.**

```
myOrders
```
This stream contains updates about your own orders.

**Message fields:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `event` | `string` | Event type: `orderAccepted`, `orderRejected`, `orderUpdate`. |
| `pair` | `string` | Trading pair symbol. |
| `obid` | `int` | *Optional.* Order unique ID. |
| `side` | `string` | *Optional.* Order side: `BUY` or `SELL` |
| `type` | `string` | *Optional.* Order type: `MARKET`, `LIMIT`, `STOP_LIMIT` |
| `time` | `int` | *Optional.* UNIX timestamp of event. |
| `time_in_force` | `string` | *Optional.* `GTC`, `IOC` or `FOK` |
| `base` | `string` | *Optional.* Base asset symbol. |
| `quote` | `string` | *Optional.* Quote asset symbol. |
| `price` | `string` | *Optional.* Order price. |
| `amount` | `string` | *Optional.* Order amount. |
| `total` | `string` | *Optional.* Order total value. |
| `stop` | `string` | *Optional.* Order stop price. |
| `status` | `string` | *Optional.* Order status: `OPEN`, `FILLED`, `KILLED`, `CANCELED`. |
| `triggered` | `bool` | *Optional.* True means that stop limit order was just triggered . |
| `filled` | `string` | *Optional.* Filled amount. |
| `reason` | `string` | *Optional.* Reason of order rejection. |

**Example message:**
```
{
	amount: "1500",
	class: "data",
	event: "orderUpdate",
	obid: 951,
	pair: "BPX/USDT",
	price: "0.04",
	side: "BUY",
	status: "CANCELED",
	stream: "myOrders"
}
```


## My trades
**This stream is private. You cannot subscribe it before performing the `auth` operation.**

```
myTrades
```
This stream contains your own trades.

**Message fields:**

| Field | Type | Description |
| :--- | :--- | :--- |
| `time` | `int` | UNIX timestamp of the trade. |
| `price` | `string` | Trade price. |
| `amount` | `string` | Trade amount. |
| `total` | `string` | Trade total value. |
| `side` | `string` | Trade side: `BUY` or `SELL` |
| `pair` | `string` | Trading pair symbol. |
| `obid` | `int` | Order unique ID. |
| `role` | `string` | `MAKER` or `TAKER` |
| `fee` | `string` | Trade fee |

**Example message:**
```
{
	amount: "30"
	class: "data"
	fee: "0"
	obid: 957
	pair: "BPX/USDT"
	price: "0.1"
	role: "TAKER"
	side: "BUY"
	stream: "myTrades"
	time: "1677879562.317149"
	total: "3"
}
```
