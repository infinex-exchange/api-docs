
# Introduction
Infinex API is also available through WebSockets. This allows multiple requests to be made asynchronously over single connection. WebSocket connection reduces the amount of data needed to be transfered (you don't need to send and receive all HTTP headers on every single request) and significantly reduces the latency. 

 - The base endpoint is: **wss://mux.infinex.cc**
 - All [HTTP API endpoints](api.md) are also supported by WebSocket server

# General information
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

# Ping
Use the `ping` operation to check if the connection is still alive.

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

# API request
Use the `req` operation to make any request to the Infinex API

**Request parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `url` | `string` | **Required**. API endpoint |
| `post` | `object` | *Optional*. Endpoint parameters |

**Response:**
Success/error applies only to WebSocket transport itself. The only error that can occur is `API unavailable`. The `body` field contains the actual API response, which contains nested success/error fields relating to the success of this endpoint call.

**Request example:**
```javascript
{
	"op": "req",
	"url": "/wallet/networks",
	"post": {
		"asset": "USDT"
	},
	"id": 2
}
```
The above request is equivalent to the following HTTP API request:
```
curl -X POST https://api.infinex.cc/wallet/networks -H 'Content-Type: application/json' -d '{"asset": "USDT"}'
```

**Response example:**
```javascript
{
	"class": "resp",
	"id": 2,
	"success": true,
	"body": {
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
}
```
