#                        WebSocket Document

## 1、General information

- The base endpoint of websocket is wss://global-api.bithumb.pro/message/realtime,
- The websocket server supply some topics for api user, such as ticker,orderbook etc. user can get real time msg by subscribing the topic which include normal (no security, like ticker, orderbook etc.) and private (need authenticate, like order change,position change etc. ).
- The single connection between server and client can keepalive a long time, so keep connection need client send {"cmd":"ping"},if the websocket server receive the msg,then server will call back a {"code":"0","msg":"pong"} 
specially, some topic response include field "ver",the field mark this message's version, prevent message backtracking

## 2、Access Process

### Access：

The websocket server endpoint is wss(ws)://{server address}/message/realtime or wss(ws)://{server address}/message/realtime?subscribe=ORDERBOOK:BTC-USDT,TICKER:BTC-USDT,it's mean that client can subscribe topics when  create connection to server, and with encrypted msg in header for private identity authentication which can subscribe private topic.if authenticate failed,the connection will be closed by server.

encrypted msg in header：

	{
		"apiKey":"",(from website)
		"apiTimestamp":"1551848831",(the connect time（millisecond), type is string)
		"apiSignature":""(the signature data)
	}
**sign example**:

request path="/message/realtime"

signatureString=request path + current timestamp(millisecond) + apiKey

apiSignature = sha256_HMAC(signatureString,secretKey)

response msg style：

	{
		"code":4,
		"data":{},
		"timestamp":1552037368,
		"topic":"CONTRACT_TICKER"
	}

code: the tag for response 
data: response data
timestamp: response time
topic: topic type

### Commands：

the base command style ：

{"cmd":"<command>", "args":["args1","args2","args3"...]}

cmd have four type：
subscribe: for subscribe some of topic
unSubscribe: for cancel subscribe some of topic
authKey: for private identity authentication, client can send this cmd once the connection is created for get authority of private topic
ping: for heart cmd, client can send this cmd with a fixed time period(30 senonds), if timeout not send, the connection will be closed by server.

args requests：

- subscribe cmd：support multi topics, such as:["TICKER:BTC-USDT","ORDERBOOK10:BTC-USDT"]
- unSubscribe cmd: remove topic, as same as subscribe.
- authKey cmd：args is fixed,["apiKey","timestamp"(millisecond,type is string),"apiSignature"]
- ping cmd：no args

**sign example**:

 request path="/message/realtime"

 signatureString=request path + current timestamp(millisecond) + apiKey

 apiSignature = sha256_HMAC(signatureString,secretKey)

### topic：

topic for certain type msg,the style such as "topic" or "topic:symbol", example: TICKER:BTC-USDT, support public and private.

#### public topic：

TICKER: the last new spot ticker msg

response data：

| Field  | Description                            | Mark                         | Type   |
| ------ | -------------------------------------- | ---------------------------- | ------ |
| c      | the last new price                     |                              | String |
| h      | the highest price in the past 24 hours |                              | String |
| l      | the lowest price in the past 24 hours  |                              | String |
| p      | price changed in the past 24 hours     |                              | String |
| symbol |                                        |                              | String |
| v      | deal quantity in the past 24 hours     |                              | String |
| ver    | version number                         | prevent message backtracking | String |

example：

```
{
	"code":4,
	"data": {
		"c":"0.0015007503751875",
		"h":"4005",
		"l":"3998",
		"p":"0.01",
		"symbol":"TBTCUSD",
		"v":"3577",
		"ver":"314"
	},
	"timestamp":1553234681,
	"topic":"TICKER"
}
```

ORDERBOOK: the last spot order book changed data

import：The creation of a complete order book on the client side must strictly depend on the processing of the 'ver' field. When the client subscribes to this topic, the message of code = 00007 is the incremental change information of the order book, and the message of code = 00006 is the one-time complete order book information. However, the 'ver' of these two times may not be continuous. At this time, the message may be lost. The complete order book The creation process is as follows：

1. The client first subscribes to the order book subject of websocket of the specific transaction pair, and stores the change information of the order book with code = 00006 in the cache of the client;

2. The client requests the complete order book information once through the REST API;

3. Finally, the cached order book change information and the complete order book are merged once. The rules for merging as follows:

   a. Judge that the change version "ver" of the first order book received by websocket should be less than or equal to the complete order book version "ver" + 1, otherwise the above process should be resumed to rebuild the order book;

   b. Judge that the change version "ver" of each order book must be greater than the complete order book version "ver", otherwise, discard the change information of the order book.

example: https://github.com/bithumb-pro/java-api-client/blob/master/src/test/java/cn/bithumb/pro/api/example/BuildOrderBook.java

response data：

| Field  | Description    | Mark                         | Type                                         |
| ------ | -------------- | ---------------------------- | -------------------------------------------- |
| b      | bids           |                              | String[2],firt is price，second is quantity  |
| s      | asks           |                              | String[2],first is price，second is quantity |
| symbol |                |                              | String                                       |
| ver    | version number | prevent message backtracking | String                                       |

example：

```
{
  "code":4,
  "data":{
  	"b":[["4003.5","1"],["4001.5","890"],["4000.5","10"],["3997","36"],["3996","100"]],
  	"s":[["4005","100"],["4006","107"]],
  	"symbol":"TBTCUSD",
  	"ver":"375"
  },
  "timestamp":1553235407,
  "topic":"CONTRACT_ORDERBOOK"
}
```

TRADE：the last spot trade msg

response data：

| Field  | Description    | Mark                         | Type   |
| ------ | -------------- | ---------------------------- | ------ |
| p      | deal price     |                              | String |
| s      | trade type     | buy or sell                  | String |
| v      | deal quantity  |                              | String |
| t      | trade time     |                              | String |
| symbol |                |                              | String |
| ver    | version number | prevent message backtracking | String |

example：

```
{
  "code":4,
  "data":{
  	"p":"4003.5",
  	"s":"buy",
  	"v":"0.1",
  	"t":"",
  	"symbol":"TBTCUSD",
  	"ver":"375"
  },
  "timestamp":1553235407,
  "topic":"CONTRACT_ORDERBOOK"
}
```

CONTRACT_TICKER:the last new contract ticker msg

response data：

| Field  | Description                         | Mark | Type   |
| ------ | -------------------------------- | ---- | ------ |
| change | price changed in the past 24 hours |      | String |
| fundRate0 | the value at next funding fee rate changed |      | String |
| fundTime0 | the time at next funding fee rate changed |      | String |
| high  | high price in the past 24 hours |      | String |
| low    | low price in the past 24 hours |      | String |
| lastPrice | the last deal price   |      | String |
| openValue | open position |      | String |
| symbol |                          |      | String |
| volume | deal amount in the past 24 hours |      | String |
| ver    | version number             | mark orderbook is the last, prevent message backtracking | String |
| openInterest |  |  | String |
| turnover |  |  | String |

example：

    {
    	"code":4,
    	"data": {
    		"change":"0.0015007503751875",
    		"fundRate0":"0.00375",
    		"fundTime0":"1553241600",
    		"high":"4005",
    		"low":"3998",
    		"lastPrice":"4004",
    		"openValue":"1.1581583219035874",
    		"symbol":"TBTCUSD",
    		"volume":"3577",
    		"ver":"314",
    		"openInterest":"",
    		"turnover":""
    	},
    	"timestamp":1553234681,
    	"topic":"CONTRACT_TICKER"
    }

CONTRACT_ORDERBOOK:the last new contract orderbook msg,if client subscribe the topic,once orderbook changed, server will send msg which is orderbook changed to channel.specially,when quantity=0,mark the price in orderbook had not exist.

response data：

| Field  | Description    | Mark | Type                                         |
| ------ | -------------- | ---- | -------------------------------------------- |
| b      | bids           |      | String[2],first is price, second is quantity |
| s      | asks           |      | String[2],first is price,second is quantity  |
| symbol |                |      | String                                       |
| ver    | version number |      | String                                       |

example：

```
{
  "code":4,
  "data":{
  	"b":[["4003.5","1"],["4001.5","890"],["4000.5","10"],["3997","36"],["3996","100"]],
  	"s":[["4005","100"],["4006","107"]],
  	"symbol":"TBTCUSD",
  	"ver":"375"
  },
  "timestamp":1553235407,
  "topic":"CONTRACT_ORDERBOOK"
}
```

CONTRACT_ORDERBOOK10: the last new contract orderbook msg(include the last new 10 asks and 10 bids),if client subscribe the topic,once orderbook changed, server will send msg to channel.

response data refer to CONTRACT_ORDERBOOK

#### private topic：

ORDER: the last new private spot order msg, if client subscribe the topic,once user order changed, server will send msg to channel.

response data：

| Field          | Description            | Mark                                          | Type   |
| -------------- | ---------------------- | --------------------------------------------- | ------ |
| oId            | order id               |                                               | String |
| price          | order price            | if type is "market", the value is "-1"        | String |
| quantity       | order quantity         |                                               | String |
| side           |                        | buy or sell                                   | String |
| symbol         |                        |                                               | String |
| type           |                        | limit or market                               | String |
| status         |                        | created，partDealt，fullDealt，canceled       | String |
| dealPrice      | Last executed price    | if status = canceled, the value is "0"        | String |
| dealQuantity   | Last executed quantity | if status = canceled, the value is "0"        | String |
| dealVolume     | Last executed volume   | if status = canceled, the value is "0"        | String |
| fee            |                        | if status = canceled, the value is "0"        | String |
| feeType        |                        | if status = canceled, the value is ""         | String |
| cancelQuantity |                        | if status is not "canceled", the value is "0" | String |
| time           | order update time      |                                               | Long   |

example：

```
{
"code":"00007",
"data":{"cancelQuantity":"10060.7","dealPrice":"0","dealQuantity":"0","dealVolume":"0","fee":"0","feeType":"","oId":"69663509668139008","price":"100.607","quantity":"100","side":"buy","status":"canceled","symbol":"BTC-USDT","time":1560758352705,"type":"limit"},
"topic":"ORDER",
"timestamp":1560758352743}
```

CONTRACT_ORDER:  the last new private contract order msg, if client subscribe the topic,once user order changed, server will send msg to channel.

response data：

| Field  | Description    | Mark                  | Type                                     |
| ---------- | ------------------ | ----------------------------- | -------------------------------------------- |
| amountFill | filled amount   |                               | String |
| amountReal | total amount       |                               | String |
| avgPrice   | deal average price |                               | String                                       |
| msgNo      | user defined       |                               | String                                       |
| orderId    |                    |                               | String                                       |
| price      |                    |                               | String                                       |
| side       |                    | buy,sell                      | String                                       |
| status     | order status       | (open,filled,cancel,rejected) | String                                       |
| symbol     |                    |                               | String                                       |
| type       | order type         | limit,market                  | String                                       |
| time | create time |  | Long |

example：

```
{
	"code":4,
	"data":{
		"amountFill":"0",
		"amountReal":"1",
		"avgPrice":"4004",
		"msgNo":"",
		"orderId":"38112649639268352",
		"price":"4004",
		"side":"buy",
		"status":"open",
		"symbol":"TBTCUSD",
		"type":"limit",
		"time":1553236866000
	},
	"topic":"CONTRACT_ORDER",
	"timestamp":1553235866
}
```

CONTRACT_ASSET: the last new private contract asset account msg,if client subscribe the topic,once user asset account changed, server will send msg to channel.

response data：

| Field         | Description     | Mark | Type  |
| ------------ | -------- | ---- | ------ |
| availableAmount | Available Amount |      | String |
| totalAmount | Total Amount |      | String |
| coin | Coin type | | String |
| openPositionMargin |  | | String |
| openOrderMarginTotal | Open order margin | | String |
| remainMargin |  | | String |

example：

```
{
	"code":4,
	"data":{
		"availableAmount":"100000.0068646403170306",
		"totalAmount":"100000.0096280041581585",
		"coin":"BTC",
		"openPositionMargin":"0.0000200306255019",
		"openOrderMarginTotal":"0.0027432190040949",
		"remainMargin":"0.0027633638411279"
	},
	"topic":"CONTRACT_ASSET",
	"timestamp":1553236515}
```

CONTRACT_POSITION:  the last new private contract position  msg,if client subscribe the topic,once position changed, server will send msg to channel.

response data：

| Field                    | Description                                      | Mark | Type   |
| ------------------------ | ------------------------------------------------ | ---- | ------ |
| symbol                   |                                                  |      | String |
| positionId               | position id                                      |      | String |
| amount                   | position amount                                  |      | String |
| side                     | buy or sell                                      |      | String |
| entryPrice               | open price                                       |      | String |
| liquiPrice               | Forced liquidation                               |      | String |
| frozen                   | frozen amount                                    |      | String |
| margin                   | position margin                                  |      | String |
| positionValue            | position value                                   |      | String |
| markPrice                | mark price                                       |      | String |
| maxRemovableMarginAmount | the max can removed margin amount                |      | String |
| lastUpdateTime           | the last position changed time                   |      | String |
| status                   | position status,  newOpen(first init),open,close |      | String |
| realProfit               | acquired profit                                  |      | String |
| leverage                 | leverage value                                   |      | String |

example：

```
{
	"code":4,
	"data":{
		"symbol":"BTCUSD",
		"positionId":"100001",
		"amount":"100",
		"entryPrice":"4800",
		"liquiPrice":"4820",
		"frozen":"0.02083",
		"margin":"0.03",
		"positionValue":"0.02083",
		"markPrice":"4802",
		"maxRemovableMarginAmount":"0.001",
		"lastUpdateTime":"1553580895",
		"status":"open",
		"realProfit":"0.01",
		"leverage":"1"
	},
	"topic":"CONTRACT_POSITION",
	"timestamp":1553236515
}
```

CONTRACT_INFO: the last new private contract normal msg, if client subscribe the topic,once position、order、asset changed, server will send msg to channel.

response data：

| Field     | Description | Mark | Type   |
| --------- | ----------- | ---- | ------ |
| symbol    |             |      | String |
| riskLimit | risk limit  |      | String |
| leverage  |             |      | String |
| fundRate0 |             |      | String |

example：

```
{
	"code":4,
	"data":{
		"symbol":"100000.0068646403170306",
		"riskLimit":"100000.0096280041581585",
		"leverage":"BTC",
		"fundRate0":"0.0000200306255019"
	},
	"topic":"CONTRACT_INFO",
	"timestamp":1553236515
}
```

### code detail：

code style is String，the success response code  is below 10000, or is wrong response

| code  | msg                 | mark |
| ----- | ------------------- | ---- |
| 0     | pong                |      |
| 00000 | Auth key success    |      |
| 00001 | Subscribe success   |      |
| 00002 | Connect success     |      |
| 00003 | UnSubscribe success |      |
| 00006 | Init msg            |      |
| 00007 | Normal msg          |      |
|       |                     |      |
|       |                     |      |
| 10000 | No cmd              |      |
| 10001 | No apiKey           |      |
| 10002 | Invalid apiKey      |      |
| 10003 | Signature Fail      |      |
| 10004 | Need verify apiKey  |      |
| 10005 | No topic            |      |
|       |                     |      |
|       |                     |      |
| 99999 | UnKnow error        |      |
