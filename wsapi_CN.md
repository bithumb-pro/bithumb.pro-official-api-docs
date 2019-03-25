#                        WebSocket文档

## 1、概念

- ws的接入方式，以wss(ws)://{域名}/message/realtime为地址，用户可以按着文档的步骤，进行接入。

- topic：服务端支持订阅的主题（用户可以根据自己的需要订阅相关的主题，一旦订阅，当服务端产生相关的信息时，则会向该通道发送消息），包含普通的主题（不需要身份认证，像行情，订单薄等），私有主题（需要先进行身份认证，认证成功后，即可订阅该类主题，像私有订单变化，仓位变化等）。

- 有些topic的响应信息里可能会包含ver字段，这个字段是用来防治消息出现回溯的情况

## 2、接入流程

### 接入：

ws的请求地址为：wss(ws)://{域名}/message/realtime或者wss(ws)://{域名}/message/realtime?subscribe=CONTRACT_ORDERBOOK:BTCUSD,CONTRACT_TICKER:BTCUSD

用户可以在创建连接的时候，订阅主题信息，并完成身份认证，通过header传递加密后的身份信息，服务端对传上来的信息进行校验，一旦验证通过，即可订阅私有主题，否则会立马关闭通道.

header的信息如下：

{

"apiKey":"",(从网页上获取)

"apiTimestamp":1551848831,(连接发起的时间戳（秒）)

"apiSignature":""(签名数据)

}

响应的消息格式如下：

{"code":4,"data":{""},"timestamp":1552037368,"topic":"CONTRACT_TICKER"}

code:为响应消息标记(详情请见code详解)

data:响应的消息体

timestamp:服务器发送的消息时间

topic:主题

### 所有指令：

基本的命令发送的格式：

{"cmd":"<command>", "args":["args1","args2","args3"...]}

cmd有两种指令：

subscribe:订阅

authKey:身份认证，用户可以在连接建立后，通过发送该指令，获取订阅私有topic的权限

ping:心跳指令,客户端可设置为30s发送一次。客户端用来维持和服务器的连接的状态，超时未发送该指令时，服务端会清除掉此连接。

args参数数组：

subscribe指令：支持多个主题，例如:["CONTRACT_TICKER:BTCUSD","CONTRAC_ORDERBOOK10:BTCUSD"]

authKey指令：args数组是固定的,["apiKey",timestamp(毫秒),"apiSignature"]

ping指令：不需要args

签名串signatureString="请求路径"+当前的时间戳(毫秒)+apiKey

apiSignature = sha256_HMAC(signatureString,secretKey)

### topic：

用户订阅某个交易对或合约的主题时，格式为：topic:symbol.例如：CONTRACT_TICKER:BTCUSD

目前支持的主题有如下几种,包含公共主题和私有主题：

公共主题：

CONTRACT_TICKER:推送最新的合约行情消息

消息体如下：

| 字段   | 说明                             | 备注                                         | 类型   |
| ------ | -------------------------------- | -------------------------------------------- | ------ |
| c      | 24小时涨跌幅(需要*100才是百分数) |                                              | String |
| fr     | 下次合约费率交换值               |                                              | String |
| ft     | 下次费率交换的时间               |                                              | String |
| h      | 24小时最高价                     |                                              | String |
| l      | 24小时最低价                     |                                              | String |
| lp     | 最新成交价                       |                                              | String |
| op     | 未平仓数量                       |                                              | String |
| symbol | 合约符号                         |                                              | String |
| v      | 24小时成交量                     |                                              | String |
| ver    | 版本号                           | 该字段是单调递增的，用来保证消息不会出现回溯 | String |

示例：

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

CONTRACT_ORDERBOOK:推送最新的合约订单薄消息，订阅完成，会立马返回全量的最新订单薄，后续当订单薄发生变化，会推送增量的订单薄变化信息，特别的当quantity=0时，表示订单薄里该档已经不存在了

消息体如下：

| 字段   | 说明       | 备注 | 类型                                      |
| ------ | ---------- | ---- | ----------------------------------------- |
| b      | bids订单薄 |      | String[2],第一个为price，第二个为quantity |
| s      | asks订单薄 |      | String[2],第一个为price，第二个为quantity |
| symbol | 合约符号   |      | String                                    |
| ver    | 版本号     |      | String                                    |

示例：

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

CONTRACT_ORDERBOOK10:推送最新的合约订单薄消息，包含最新的买卖10条，订阅完成，会立马返回全量的最新订单薄，后续当订单薄发生变化时，会立即推送最新的买卖10订单薄

消息体参考CONTRACT_ORDERBOOK

私有主题：

CONTRACT_ORDER: 推送用户私有订单的消息，当其订单发生变化时，如果用户订阅了此主题，则立即会向订阅的channel里发送消息

消息体如下：

| 字段       | 说明                 | 备注                                                         | 类型                                      |
| ---------- | -------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| amountFill | 已成交的数量         |                                                              | String[2],第一个为price，第二个为quantity |
| amountReal | 委托数量             |                                                              | String[2],第一个为price，第二个为quantity |
| avgPrice   | 成交均价             |                                                              | String                                    |
| msgNo      | 用户自定义的请求信息 |                                                              | String                                    |
| orderId    | 订单ID               |                                                              | String                                    |
| price      | 委托价格             |                                                              | String                                    |
| side       | 方向                 | buy,sell                                                     | String                                    |
| status     | 订单状态             | (open(挂单中),filled(已成交),cancel(已取消),rejected(已拒绝)) | String                                    |
| symbol     | 合约符号             |                                                              | String                                    |
| type       | 订单类型             | limit,market                                                 | String                                    |

示例：

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

CONTRACT_ASSET: 推送用户私有资产的消息，当用户资产发生变动时，如果用户订阅了此主题，则立即会向订阅的channel里发送消息

消息体如下：

| 字段         | 说明     | 备注 | 类型   |
| ------------ | -------- | ---- | ------ |
| availableAmt | 可用余额 |      | String |
| totalAmt     | 总资产   |      | String |

示例：

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



### code详解：

code格式为String，10000以下的code表示为服务端产生的正确的响应信息，10000以上的表示为错误的响应

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







