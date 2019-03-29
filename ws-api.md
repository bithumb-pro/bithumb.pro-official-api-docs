#                        WebSocket Document

## 1、General information

- The base endpoint of websocket is wss://global-api.dcex.world/message/realtime,
- The websocket server supply some topics for api user, such as ticker,orderbook etc. user can get real time msg by subscribing the topic which include normal (no security, like ticker, orderbook etc.) and private (need authenticate, like order change,position change etc. ).
- The single connection between server and client can keepalive a long time, so keep connection need client send {"cmd":"ping"},if the websocket server receive the msg,then server will call back a {"code":"0","msg":"pong"} 
specially, some topic response include field "ver",the field mark this message's version, prevent message backtracking

## 2、Access Process

### Access：

The websocket server endpoint is wss(ws)://{server address}/message/realtime or wss(ws)://{server address}/message/realtime?subscribe=CONTRACT_ORDERBOOK:BTCUSD,CONTRACT_TICKER:BTCUSD,it's mean that client can subscribe topics when  create connection to server, and with encrypted msg in header for private identity authentication which can subscribe private topic.if authenticate failed,the connection will be closed by server.

encrypted msg in header：

	{
		"apiKey":"",(from website)
		"apiTimestamp":1551848831,(the connect time（second）)
		"apiSignature":""(the signature data)
	}

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

- subscribe cmd：support multi topics, such as:["CONTRACT_TICKER:BTCUSD","CONTRAC_ORDERBOOK10:BTCUSD"]
- unSubscribe cmd: remove topic, as same as subscribe.
- authKey cmd：args is fixed,["apiKey",timestamp(millionsecond),"apiSignature"]
- ping cmd：no args

example:
 signatureString=request path+current timestamp(millionsecond)+apiKey
 apiSignature = sha256_HMAC(signatureString,secretKey)

### topic：

topic for certain type msg,the style such as "topic" or "topic:symbol", example: CONTRACT_TICKER:BTCUSD, support public and private.

#### public topic：

TICKER: the last new ticker msg

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

ORDERBOOK: the last order book changed data

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

TRADE：the last trade msg

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
| c      | price changed in the past 24 hours |      | String |
| fr     | the value at next funding fee rate changed |      | String |
| ft     | the time at next funding fee rate changed |      | String |
| h      | high price in the past 24 hours |      | String |
| l      | low price in the past 24 hours |      | String |
| lp     | the last deal price   |      | String |
| op     | open position |      | String |
| symbol |                          |      | String |
| v      | deal amount in the past 24 hours |      | String |
| ver    | version number             | mark orderbook is the last, prevent message backtracking | String |

example：

    {
    	"code":4,
    	"data": {
    		"c":"0.0015007503751875",
    		"fr":"0.00375",
    		"ft":"1553241600",
    		"h":"4005",
    		"l":"3998",
    		"lp":"4004",
    		"op":"1.1581583219035874",
    		"symbol":"TBTCUSD",
    		"v":"3577",
    		"ver":"314"
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

CONTRACT_ORDER:  the last new private contract order msg, if client subscribe the topic,once user order changed, server will send msg to channel.

response data：

| 字段       | 说明               | 备注                          | 类型                                         |
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
		"type":"limit"
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

example：

```
{
	"code":4,
	"data":{
		"availableAmount":"0.9997228830328933",
		"totalAmount":"0.999999019221543"
	},
	"topic":"CONTRACT_ASSET",
	"timestamp":1553236515}
```

CONTRACT_POSITION:  the last new private contract position  msg,if client subscribe the topic,once position changed, server will send msg to channel.

response data：

| Field          | Description                                      | Mark | Type   |
| -------------- | ------------------------------------------------ | ---- | ------ |
| symbol         |                                                  |      | String |
| positionId     | position id                                      |      | String |
| amount         | position amount                                  |      | String |
| side           | buy or sell                                      |      | String |
| entryPrice     | open price                                       |      | String |
| liquiPrice     | Forced liquidation                               |      | String |
| frozen         | frozen amount                                    |      | String |
| margin         | position margin                                  |      | String |
| positionValue  | position value                                   |      | String |
| markPrice      | mark price                                       |      | String |
| maxReMrgAmount | the max can removed margin amount                |      | String |
| lastUpdateTime | the last position changed time                   |      | String |
| status         | position status,  newOpen(first init),open,close |      | String |
| realProfit     | acquired profit                                  |      | String |
| leverage       | leverage value                                   |      | String |

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
		"maxReMrgAmount":"0.001",
		"lastUpdateTime":"1553580895",
		"status":"open",
		"realProfit":"0.01",
		"leverage":"1"
	},
	"topic":"CONTRACT_ASSET",
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