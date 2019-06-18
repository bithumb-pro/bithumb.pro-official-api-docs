#                        WebSocket文档

## 1、概念

- ws的接入方式，以wss://global-api.bithumb.pro/message/realtime为地址，用户可以按着文档的步骤，进行接入。
- topic：服务端支持订阅的主题（用户可以根据自己的需要订阅相关的主题，一旦订阅，当服务端产生相关的信息时，则会向该通道发送消息），包含普通的主题（不需要身份认证，像行情，订单薄等），私有主题（需要先进行身份认证，认证成功后，即可订阅该类主题，像私有订单变化，仓位变化等）。
- 有些topic的响应信息里可能会包含ver字段，这个字段是用来防治消息出现回溯的情况

## 2、接入流程

### 接入：

ws的请求方式可以为：wss(ws)://{域名}/message/realtime
或者以如下的方式：
wss(ws)://{域名}/message/realtime?subscribe=ORDERBOOK:BTC-USDT,TICKER:BTC-USDT

用户可以在创建连接的时候，订阅主题信息，并完成身份认证，通过header传递加密后的身份信息，服务端对传上来的信息进行校验，一旦验证通过，即可订阅私有主题，否则会立马关闭通道.

header的信息如下：

```
{
	"apiKey":"",(从网页上获取)
	"apiTimestamp":"1551848831",(连接发起的时间戳（毫秒）,类型为string)
	"apiSignature":""(签名数据)
}
```

响应的消息格式如下：

```
{
	"code":4,
	"data":{},
	"timestamp":1552037368,
	"topic":"CONTRACT_TICKER"
}
```

code:为响应消息标记(详情请见code详解)

data:响应的消息体

timestamp:服务器发送的消息时间

topic:主题

### 所有指令：

基本的命令发送的格式：

{"cmd":"<command>", "args":["args1","args2","args3"...]}

cmd有四种指令：

subscribe: 用于订阅topic

unSubscribe: 用于取消订阅topic

authKey: 身份认证，用户可以在连接建立后，通过发送该指令，获取订阅私有topic的权限

ping: 心跳指令,客户端可设置为30s发送一次。客户端用来维持和服务器的连接的状态，超时未发送该指令时，服务端会清除掉此连接。

args参数数组：

subscribe指令：支持多个主题，例如:["TICKER:BTC-USDT","ORDERBOOK10:BTC-USDT"]

authKey指令：args数组是固定的,["apiKey","timestamp(毫秒)","apiSignature"]

ping指令：不需要args

'''签名串signatureString'''="请求路径"+当前的时间戳(毫秒,类型为String)+apiKey

apiSignature = sha256_HMAC(signatureString,secretKey)

### topic：

用户订阅某个交易对或合约的主题时，格式为：topic:symbol.例如：TICKER:BTC-USDT

目前支持的主题有如下几种,包含公共主题和私有主题：

#### 公共主题：

TICKER: 推送最新的虚拟币行情消息

消息体如下：

| 字段   | 说明         | 备注                                         | 类型   |
| ------ | ------------ | -------------------------------------------- | ------ |
| c      | 最新成交价   |                                              | String |
| h      | 24小时最高价 |                                              | String |
| l      | 24小时最低价 |                                              | String |
| p      | 24小时涨跌幅 |                                              | String |
| symbol | 合约符号     |                                              | String |
| v      | 24小时成交量 |                                              | String |
| ver    | 版本号       | 该字段是单调递增的，用来保证消息不会出现回溯 | String |

示例：

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

ORDERBOOK: 推送最新的ORDER BOOK(深度)的变化

消息体如下：

| 字段   | 说明             | 备注                                         | 类型                                      |
| ------ | ---------------- | -------------------------------------------- | ----------------------------------------- |
| b      | bids（买订单薄） |                                              | String[2],第一个为price，第二个为quantity |
| s      | asks（卖订单薄） |                                              | String[2],第一个为price，第二个为quantity |
| symbol | 合约符号         |                                              | String                                    |
| ver    | 版本号           | 该字段是单调递增的，用来保证消息不会出现回溯 | String                                    |

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

TRADE：推送最新的交易信息

消息体如下：

| 字段   | 说明       | 备注                                         | 类型   |
| ------ | ---------- | -------------------------------------------- | ------ |
| p      | 成交价格   |                                              | String |
| s      | 交易类型   | buy or sell                                  | String |
| v      | 成交量     |                                              | String |
| t      | 成交时间戳 |                                              | String |
| symbol | 合约符号   |                                              | String |
| ver    | 版本号     | 该字段是单调递增的，用来保证消息不会出现回溯 | String |

示例：

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

CONTRACT_TICKER: 推送最新的合约行情消息

消息体如下：

| 字段         | 说明                             | 备注                                         | 类型   |
| ------------ | -------------------------------- | -------------------------------------------- | ------ |
| change       | 24小时涨跌幅(需要*100才是百分数) |                                              | String |
| fundRate0    | 下次合约费率交换值               |                                              | String |
| fundTime0    | 下次费率交换的时间               |                                              | String |
| high         | 24小时最高价                     |                                              | String |
| low          | 24小时最低价                     |                                              | String |
| lastPrice    | 最新成交价                       |                                              | String |
| openValue    | 未平仓数量                       |                                              | String |
| symbol       | 合约符号                         |                                              | String |
| volume       | 24小时成交量                     |                                              | String |
| ver          | 版本号                           | 该字段是单调递增的，用来保证消息不会出现回溯 | String |
| openInterest |                                  |                                              | String |
| turnover     |                                  |                                              | String |

示例：

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

#### 私有主题：

ORDER: 实时推送用户现货订单的消息，当其订单发生变化时，如果用户订阅了此主题，则立即会向订阅的channel里发送消息

消息体如下：

| 字段           | 说明             | 备注                                                         | 类型   |
| -------------- | ---------------- | ------------------------------------------------------------ | ------ |
| oId            | 订单id           |                                                              | String |
| price          | 订单委托价格     | 当为“-1”时，表示市价                                         | String |
| quantity       | 订单委托数量     |                                                              | String |
| side           | 订单方向         | buy or sell                                                  | String |
| symbol         |                  |                                                              | String |
| type           | 订单类型         | limit or market                                              | String |
| status         | 订单状态         | created(已创建)，partDealt(部分成交)，fullDealt(完全成交)，canceled(已撤单) | String |
| dealPrice      | 成交价格         | 撤单时为0                                                    | String |
| dealQuantity   | 成交数量         | 撤单时为0                                                    | String |
| dealVolume     | 成交额           | 撤单时为0                                                    | String |
| fee            | 手续费           | 撤单时为0                                                    | String |
| feeType        | 手续费的单位     | 撤单时为""                                                   | String |
| cancelQuantity | 撤单的数量       |                                                              | String |
| time           | 订单变化发生时间 |                                                              | Long   |

示例：

```
{
"code":"00007",
"data":{"cancelQuantity":"10060.7","dealPrice":"0","dealQuantity":"0","dealVolume":"0","fee":"0","feeType":"","oId":"69663509668139008","price":"100.607","quantity":"100","side":"buy","status":"canceled","symbol":"BTC-USDT","time":1560758352705,"type":"limit"},
"topic":"ORDER",
"timestamp":1560758352743}
```

CONTRACT_ORDER: 推送用户私有合约订单的消息，当其订单发生变化时，如果用户订阅了此主题，则立即会向订阅的channel里发送消息

消息体如下：

| 字段       | 说明                 | 备注                                                         | 类型   |
| ---------- | -------------------- | ------------------------------------------------------------ | ------ |
| amountFill | 已成交的数量         |                                                              | String |
| amountReal | 委托数量             |                                                              | String |
| avgPrice   | 成交均价             |                                                              | String |
| msgNo      | 用户自定义的请求信息 |                                                              | String |
| orderId    | 订单ID               |                                                              | String |
| price      | 委托价格             |                                                              | String |
| side       | 方向                 | buy,sell                                                     | String |
| status     | 订单状态             | (open(挂单中),filled(已成交),cancel(已取消),rejected(已拒绝)) | String |
| symbol     | 合约符号             |                                                              | String |
| type       | 订单类型             | limit,market                                                 | String |
| time       | 下单时间             |                                                              | Long   |

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
		"type":"limit",
		"time":1553245866000
	},
	"topic":"CONTRACT_ORDER",
	"timestamp":1553235866
}
```

CONTRACT_ASSET: 推送用户私有资产的消息，当用户资产发生变动时，如果用户订阅了此主题，则立即会向订阅的channel里发送消息

消息体如下：

| 字段                 | 说明       | 备注 | 类型   |
| -------------------- | ---------- | ---- | ------ |
| availableAmount      | 可用余额   |      | String |
| totalAmount          | 总资产     |      | String |
| coin                 | 资产类型   |      | String |
| openPositionMargin   | 仓位保证金 |      | String |
| openOrderMarginTotal | 委托保证金 |      | String |
| remainMargin         | 保证金余额 |      | String |

示例：

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
	"timestamp":1553236515
}
```

CONTRACT_POSITION: 推送用户合约私有仓位信息，当用户的合约仓位发生变动时，如果用户订阅了此主题，则立即会向订阅的channel里发送消息

消息体如下：

| 字段                     | 说明                                 | 备注 | 类型   |
| ------------------------ | ------------------------------------ | ---- | ------ |
| symbol                   | 合约符号                             |      | String |
| positionId               | 仓位ID                               |      | String |
| amount                   | 仓位数量                             |      | String |
| side                     | buy or sell                          |      | String |
| entryPrice               | 开仓价格                             |      | String |
| liquiPrice               | 强平价格                             |      | String |
| frozen                   | 冻结的金额                           |      | String |
| margin                   | 仓位保证金                           |      | String |
| positionValue            | 仓位价值                             |      | String |
| markPrice                | 计算的mark价格                       |      | String |
| maxRemovableMarginAmount | 最大可移除保证金                     |      | String |
| lastUpdateTime           | position最后变动的时间戳             |      | String |
| status                   | 仓位状态，newOpen(初始化),open,close |      | String |
| realProfit               | 已实现盈亏                           |      | String |
| leverage                 | 杠杆值                               |      | String |

示例：

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

CONTRACT_INFO: 推送用户私有的通用消息，当用户仓位，订单，资产发生变动时，如果用户订阅了此主题，则立即会向订阅的channel里发送消息

消息体如下：

| 字段      | 说明     | 备注 | 类型   |
| --------- | -------- | ---- | ------ |
| symbol    | 合约符号 |      | String |
| riskLimit | 风险限额 |      | String |
| leverage  | 杠杆倍数 |      | String |
| fundRate0 | 资金费用 |      | String |

示例：

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

### code详解：

code格式为String，10000以下的code表示为服务端产生的正确的响应信息，10000以上的表示为错误的响应

| code  | msg                 | mark |
| ----- | ------------------- | ---- |
| 0     | Pong                |      |
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







