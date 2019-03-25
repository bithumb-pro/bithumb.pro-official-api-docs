#                        WebSocket Document

## 1、General information

- The base endpoint of websocket is wss(ws)://{server address}/message/realtime,
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
		"data":{""},
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

cmd have two type：
subscribe: for subscribe
authKey: for private identity authentication, client can send this cmd once the connection is created for get authority of private topic
ping: for heart cmd, client can send this cmd with a fixed time period(30 senonds), if timeout not send, the connection will be closed by server.

args requests：

subscribe cmd：support multi topics, such as:["CONTRACT_TICKER:BTCUSD","CONTRAC_ORDERBOOK10:BTCUSD"]

authKey cmd：args is fixed,["apiKey",timestamp(millionsecond),"apiSignature"]
ping cmd：no args

signatureString=request path+current timestamp(millionsecond)+apiKey
apiSignature = sha256_HMAC(signatureString,secretKey)

### topic：

topic for certain type msg,the style such as "topic" or "topic:symbol", example: CONTRACT_TICKER:BTCUSD, support public and private.

#### public topic：

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

CONTRACT_ORDERBOOK:the last new contract orderbook msg,if client subscribe the topic,once orderbook is changed, server will send msg which is orderbook changed to channel.special,when quantity=0,mark the price in orderbook had not exist.

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

CONTRACT_ORDERBOOK10: the last new contract orderbook msg(include the last new 10 asks and 10 bids),if client subscribe the topic,once orderbook is changed, server will send msg to channel.

response data refer to CONTRACT_ORDERBOOK

#### private topic：

CONTRACT_ORDER:  the last new private contract order msg, if client subscribe the topic,once user order is changed, server will send msg to channel.

response data：

| 字段       | 说明               | 备注                          | 类型                                         |
| ---------- | ------------------ | ----------------------------- | -------------------------------------------- |
| amountFill | filled amount   |                               | String[2],first is price, second is quantity |
| amountReal | total amount       |                               | String[2],first is price, second is quantity |
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

CONTRACT_ASSET: the last new private contract asset account msg,if client subscribe the topic,once user asset account is changed, server will send msg to channel.

response data：

| Field         | Description     | Mark | Type  |
| ------------ | -------- | ---- | ------ |
| availableAmt | Available Amount |      | String |
| totalAmt     | Total Amount |      | String |

example：

```
{
	"code":4,
	"data":{
		"availableAmt":"0.9997228830328933",
		"totalAmt":"0.999999019221543"
	},
	"topic":"CONTRACT_ASSET",
	"timestamp":1553236515}
```

