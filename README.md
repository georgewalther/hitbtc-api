## Summary

This document provides the complete reference for [hitbtc](https://hitbtc.com) API.


The following symbols are traded on hitbtc exchange.

| Symbol	| Lot size | Price step
| --- | --- | --- |
| BTCUSD |	0.01 BTC |	0.01 |
| BTCEUR |	0.01 BTC |	0.01 |
| LTCBTC | 0.1 LTC |	0.00001 |
| LTCUSD | 0.1 LTC |	0.001 |
| LTCEUR | 0.1 LTC |	0.001 |
| EURUSD | 1 EUR |	0.0001 |

Size values in messages are represented in lots.

## Market data RESTful API

Endpoint URL: [http://api.hitbtc.com](http://api.hitbtc.com)

### /api/1/public/time

Request: `GET /api/1/public/time`

Example: `/api/1/public/time`
``` json
{
    "timestamp": 1393492619000
}
```

### /api/1/public/symbols

Request: `GET /api/1/public/symbols`

Example: `/api/1/public/symbols`
``` json
{
    "symbols": [
        {
            "symbol": "BTCUSD"
            "step": "0.01",
            "lot": "0.01",
        },
        {
            "symbol": "BTCEUR"
            "step": "0.01",
            "lot": "0.01",
        },
        ...
    ]
}
```

### /api/1/public/:symbol/ticker

Request: `GET /api/1/public/:symbol/ticker`

Example: `/api/1/public/BTCUSD/ticker`

``` json
{
    "last": "550.73",
    "bid": "549.56",
    "ask": "554.12",
    "high": "600.1",
    "low": "400.7",
    "volume": "567.9",
    "timestamp": 1393492619000
}
```

* 24h means last 24h + last incomplete minute
* high - highest trade price / 24 h
* low - lowest trade price / 24 h
* volume - volume / 24h

### /api/1/public/:symbol/orderbook

Request: `GET /api/1/public/:symbol/orderbook`

Alias: `/api/1/request/:symbol/orderbook.json`

Example: `/api/1/public/BTCUSD/orderbook`
``` json
{
    "asks": [
        [ "405.71", "0.09" ],
        [ "406.65", "0.06" ],
        [ "409.51", "0.15" ],
        [ "413.93", "51.6" ],
        [ "414.59", "47.1" ]
    ],
    "bids": [
        [ "398.3", "0.15" ],
        [ "396.99", "0.13" ],
        [ "395", "0.5" ],
        [ "391.93", "42.4" ],
        [ "383.67", "145.4" ]
    ]
}
```

### /api/1/public/:symbol/trades

Request: `GET /api/1/public/:symbol/trades`

Alias: `/api/1/request/:symbol/trades.json?since={trade_id}`

Parameters:

| Parameter | Description |
| --- | --- |
| from | required, int, trade_id or timestamp |
| till | optional, int, trade_id or timestamp |
| by | required, filter and sort by `trade_id` or `ts` (timestamp) |
| sort | optional, `asc` (default) or `desc` |
| start_index | required, int |
| max_results | required, int, max value = 1000 |

Example: `/api/1/public/BTCUSD/trades?from=0&by=trade_id&sort=desc&start_index=0&max_results=100`
``` json
{
    "trades": [
        [ 3814483, "575.64", "0.02", 1393492619000 ],
        [ 3814482, "574.3", "0.12", 1393492619000 ],
        [ 3814481, "573.67", "3.8", 1393492619000 ],
        [ 3814479, "571", "0.01", 1393492619000 ],
        ...
    ]
}
```


## Market data streaming end-point

Streaming API is based on [WebSocket protocol](http://en.wikipedia.org/wiki/WebSocket). All messages are in JSON format.

URL: [ws://api.hitbtc.com](ws://api.hitbtc.com)

Once client connects to this URL the session is started. 

The server broadcasts the following types of messages:

* [MarketDataSnapshotFullRefresh](#MarketDataSnapshotFullRefresh) message contains a full snapshot of the order book.
* [MarketDataIncrementalRefresh](#MarketDataIncrementalRefresh) message contains incremental changes

Some recommendations to consider:

* The application could receive the first snapshot and maintain the order book by applying incremental updates.
* It's recommended to invalidate a state of the application periodically using snapshots.
* It's recommended to check sequence numbers and to drop updates with non-monotonous sequence numbers.

<a name="MarketDataSnapshotFullRefresh"/>
### MarketDataSnapshotFullRefresh

MarketDataSnapshotFullRefresh message contains a full snapshot of the order book.

Example message:
```json
{"MarketDataSnapshotFullRefresh": {
    "snapshotSeqNo": 899009,
    "symbol": "BTCUSD",
    "exchangeStatus": "working",
    "ask": [
        {
            "price": 101.42,
            "size": 7
        },
        {
            "price": 101.85,
            "size": 5
        },
        {
            "price": 102.59,
            "size": 1
        },
        {
            "price": 114.53,
            "size": 3
        },
        {
            "price": 114.54,
            "size": 6
        },
        {
            "price": 114.55,
            "size": 19
        }
    ],
    "bid": [
        {
            "price": 89.72,
            "size": 79
        },
        {
            "price": 89.71,
            "size": 158
        },
        {
            "price": 89.7,
            "size": 166
        },
        {
            "price": 89.69,
            "size": 231
        },
        {
            "price": 89.68,
            "size": 169
        },
        {
            "price": 89.67,
            "size": 186
        },
        {
            "price": 89.66,
            "size": 178
        }
    ]
}}
```

Fields:

| Field | Description |
| --- | --- |
| seqNo | monotone increasing number, each symbol has an own sequence |
| timestamp | millisecond timestamp UTC |
| symbol | |
| exchangeStatus | `on` or `off`, `off` means the trading is suspended |
| ask, bid | sorted arrays of price levels in the order book; full snapshot (all price levels) is provided |

<a name="MarketDataIncrementalRefresh"/>
### MarketDataIncrementalRefresh

MarketDataIncrementalRefresh contains incremental changes of the order book and individual trades.

Example message:
```json
{"MarketDataIncrementalRefresh": {
    "seqNo": 546693,
    "timestamp": 1381394357861,
    "symbol": "BTCUSD",
    "exchangeStatus": "on",
    "ask": [],
    "bid": [
        {
            "price": 100.98,
            "size": 3
        }
    ],
    "trade": [
        {
            "price": 100.98,
            "size": 5,
            "timestamp": 1346691273926
        }
    ]
}}
```

Fields:

| Field | Description |
| --- | --- |
| seqNo	| monotone increasing number, each symbol has an own sequence
| timestamp |	millisecond timestamp UTC |
| symbol	| |
| exchangeStatus |  `on` or `off`, `off` means the trading is suspended |
| ask, bid, trade | an array of changes in the order book; <br> `size` means new size, `size`=0 means price level has been removed |

## Trading streaming end-point

Streaming API is based on [WebSocket protocol](http://en.wikipedia.org/wiki/WebSocket). All messages are in JSON format.

URL: <wss://api.hitbtc.com:8080>

Trading endpoint requires sending login message after connection esteblished. All client messages should be signed and should contain valid and active API key

The following message types are supported:

| Type |  |
| --- | --- | --- |
| [Login](#Login) | Client -> Server |
| [NewOrder](#NewOrder) | Client -> Server |
| [OrderCancel](#OrderCancel) | Client -> Server  |
| [ExecutionReport](#ExecutionReport) | Server -> Client |
| [CancelReject](#CancelReject) | Server -> Client |


### API keys and message signatures

All client messages should be signed in the following manner:

```json
{
    "apikey": "e418f5b4a15608b78185540ef583b9fc",
    "signature": "FN6/9dnMfLh3wZj+cAFr82HcSvmwuniMQqUlRxSQ9WxRqFpYrjY2xlvDzLC5+qSZAHts8R7KR7HbjiI3SzVxHg==",
    "message":{
        "nonce": 12,
        "payload": {
            "Login": {}
        }
    }
}
```

| Field | Description |
| --- | --- |
| nonce | should be monotonous within the same connection |
| signature | base64 [hmac-sha512](http://en.wikipedia.org/wiki/Hash-based_message_authentication_code)(binary representation of the message) |

<a name="Login"/>
### Login 

Example:
```json
{
	"apikey": "e418f5b4a15608b78185540ef583b9fc",
	"signature": "FN6/9dnMfLh3wZj+cAFr82HcSvmwuniMQqUlRxSQ9WxRqFpYrjY2xlvDzLC5+qSZAHts8R7KR7HbjiI3SzVxHg==",
	"message":{
		"nonce": 12, 
		"payload": {
			"Login": {}
		}
	}
}
```

Parameters: no parameters

If client doesn't send valid logon message in 10 second the connection will be dropped.

<a name="NewOrder"/>
### NewOrder

Example:

```json
{
    "apikey": "e418f5b4a15608b78185540ef583b9fc",
    "signature": "FN6/9dnMfLh3wZj+cAFr82HcSvmwuniMQqUlRxSQ9WxRqFpYrjY2xlvDzLC5+qSZAHts8R7KR7HbjiI3SzVxHg==",
    "message":{
        "nonce": 12,
        "payload": {
            "NewOrder": {
                "clientOrderId": "68f82819-723a-4b60-ad6b",
                "symbol": "BTCUSD",
                "side": "buy",
                "quantity": 10,
                "type": "limit",
                "price": 788.12,
                "timeInForce": "GTC"
            }
        }
    }
}
```

Parameters:

| Parameter	| Description | Type / Enum |
| --- | --- | --- |
| clientOrderId | should be unique, <= 30 characters | |
| symbol | | |
| side | order side | `buy`, `sell` |
| quantity | quantity in lots | integer |
| type | order type	| only `limit` orders are currently supported |
| price	| price (in currency) | decimal, consider price steps |
| timeInForce | time in force | `GTC` - Good-Til-Canceled <br>`IOK` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill |

<a name="OrderCancel"/>
### OrderCancel

Example:

```json
{
    "apikey": "e418f5b4a15608b78185540ef583b9fc",
    "signature": "FN6/9dnMfLh3wZj+cAFr82HcSvmwuniMQqUlRxSQ9WxRqFpYrjY2xlvDzLC5+qSZAHts8R7KR7HbjiI3SzVxHg==",
    "message":{
        "nonce": 12,
        "payload": {
            "OrderCancel": {
                "clientOrderId": "68f82819-723a-4b60-ad6b",
                "cancelRequestClientOrderId":   "2c4d7127-6fbc-450c-b851-c6c1e8954545",
                "symbol": "BTCUSD",
                "side": "buy"
            }
        }
    }
}
```

Parameters:

| Parameter	| Description | Type / Enum |
| --- | --- | --- |
| clientOrderId | clientOrderId sent in NewOrder message | |
| cancelRequestClientOrderId | <= 30 characters | |
| symbol | | |
| side | order side | `buy`, `sell` |
| type | order type	| only `limit` orders are currently supported |

<a name="ExecutionReport"/>
### ExecutionReport

Example:

```json
{
    "ExecutionReport":{
        "orderId": "64283442",
        "clientOrderId": "68f82819-723a-4b60-ad6b",
        "execReportType": "new",
        "orderStatus": "new"
        "symbol": "BTCUSD",
        "side": "buy",
        "timestamp": 1346691273926,
        "price": 690.99,
        "quantity": 0.1,
        "type": "limit",
        "timeInForce": "GTC"
    }
}
```

Fields:

| Field	| Description | Type / Enum | Required |
| --- | --- | --- | --- |
| orderId | Order ID on the Exchange | string | required |
| clientOrderId | clientOrderId sent in NewOrder message | string | required |
| execReportType | execution report type | `new` <br> `canceled` <br> `rejected` <br> `expired` <br> `trade` <br> `status` | required |
| orderStatus | order status | `new` <br> `partiallyFilled` <br> `filled` <br> `canceled` <br> `rejected` <br> `expired` | required |
| orderRejectReason | Relevant only for the orders in rejected state | `unknownSymbol` <br> `exchangeClosed` <br>`orderExceedsLimit` <br> `unknownOrder` <br> `duplicateOrder` <br> `unsupportedOrder` <br> `unknownAccount` <br> `other`| |
| symbol | | | required |
| side | | `buy` or `sell` | required |
| timestamp | UTC timestamp in milliseconds | | |
| price | | decimal | |
| quantity | | integer | required |
| type | | only `limit` orders are currently supported | required |
| timeInForce | time in force | `GTC` - Good-Til-Canceled <br>`IOK` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill | required |
| tradeId | Trade ID on the exchange | | for trades |
| lastQuantity | | integer | for trades |
| lastPrice | | decimal | for trades |
| leavesQuantity | | integer |  |
| cumQuantity | | integer | |
| averagePrice | | decimal, will be 0 if 'cumQuantity'=0 | |

<a name="CancelReject"/>
### CancelReject

Example:
```json
{"CancelReject": {
    "clientOrderId": "68f82819-723a-4b60-ad6b",
    "cancelRequestClientOrderId": "2c4d7127-6fbc-450c-b851",
    "rejectReasonCode": "orderNotFound",
    "rejectReasonText": "Order not found",
    "timestamp": 726892347829
}}
```

Fields:

| Field	| Description | Type / Enum | Required |
| --- | --- | --- | --- |
| cancelRequestClientOrderId | `cancelRequestClientOrderId` from OrderCancel | string | required |
| clientOrderId | `clientOrderId` from OrderCancel | string | required |
| rejectReasonCode | | `orderNotFound` <br> `unknownSymbol` <br> `unknownUser` <br> `other` | required |
| rejectReasonText | Optional reject reason text | string | |

## Useful tools

Chrome extension Simple WebSocket Client (https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo)

## Sample code

### Node.js snippet: message signature

```javascript
    var crypto = require('crypto');
    
    ...
    
    var msg = {
        'apikey': apikey,
        'signature': '',
        'message': {
            'nonce': nonce,
            'payload': {
                'NewOrder': {
                    'clientOrderId': clientOrderId,
                    'symbol': symbol,
                    'side': side,
                    'quantity': quantity,
                    'type': type,
                    'price': price,
                    'timeInForce': timeInForce
                }
            }
        }
    };
    msg.signature = crypto.createHmac('sha512', secretkey).update(JSON.stringify(msg.message)).digest('base64');
    return JSON.stringify(msg);
```

