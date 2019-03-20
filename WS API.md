#                        WebSocket文档

## 1、概念

ws的接入方式，以wss(ws)://{域名}/message/realtime为地址，用户可以按着文档的步骤，进行接入。

topic：服务端支持订阅的主题（用户可以根据自己的需要订阅相关的主题，一旦订阅，当服务端产生相关的信息时，则会向该通道发送消息），包含普通的主题（不需要身份认证，像行情，订单薄等），私有主题（需要先进行身份认证，认证成功后，即可订阅该类主题，像私有订单变化，仓位变化等）。

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

code:为响应消息标记

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

签名串signatureString="请求路径"+当前的时间戳(毫秒)+secretKey

apiSignature = sha256_HMAC(signatureString,secretKey)

### topic：

用户订阅某个交易对或合约的主题时，格式为：topic:symbol.例如：CONTRACT_TICKER:BTCUSD

目前支持的主题有如下几种：

CONTRACT_TICKER:合约行情

CONTRACT_ORDERBOOK:合约订单薄，订阅完成，会立马返回全量的最新订单薄，后续当订单薄发生变化，会推送增量的订单薄

CONTRACT_ORDERBOOK10:合约订单薄，最新买卖10条，订阅完成，会立马返回全量的最新订单薄，后续当订单薄发生变化时，会推送最新的买卖10订单薄











