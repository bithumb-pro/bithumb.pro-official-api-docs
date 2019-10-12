catalog

[TOC]

## [General Rest Api Information]

- The base endpoint is: **https://global-openapi.bithumb.pro/openapi/v1/**
- All endpoints return either a JSON object or array.
- For `GET` endpoints, parameters must be sent as a `query string`.
- For POST endpoints, parameters must be send in the request body, with content type by JSON(application/json)
- Parameters may be sent in any order.

| Version | Description                                                  | Time    | Mark   |
| ------- | ------------------------------------------------------------ | ------- | ------ |
| V1.0.0  | include some rest api(virtual coin,contracts) and error code list. | 2019-03 | usable |
|         |                                                              |         |        |

## [Request Parameter Infomation] (Important)

### Public request params

| Field     | Description         | Required(Y or N) | Mark                                                         | Type   |
| --------- | ------------------- | ---------------- | ------------------------------------------------------------ | ------ |
| apiKey    | unique tag for user | N                | apply for the key in website.                        | String |
| timestamp | request timestamp   | N                | the paramter will be need in request which need to authenticate, if space of time too long between current and request will be reject. | Long  |
| signature | signature data      | N                | required by authentication of request.                       | String |
| msgNo     | request id          | N                | just for some api(place order),max length is 50 chars        | String |

### Response Information

Base style :

	{
		"info": {},
		"success": true,
		"msg": "",
		"code": "0",
		"params": []
	}

if need page, like:

```
{
	"info": {
        "num":xx, // total counts
        "list":[] // data collection 
	},
	"success": true,
	"msg": "",
	"code": "0",
	"params": []
}
```

## [Data of Signature] (Important)

- The part just for authentication of request, the pair of apiKey and secretKey can be apply in website (https://bithumb.pro)

- For authentication of request, the request paramters need to be sign  by HmacSHA256

- The prepared signature data is a json string which make up of  all request parameters which sort by dict and join with '&'

- The signature needs to be in lowercase

  signature example:

  request parameters like:

    	{
   		"apiKey" : "XXX",
   		"msgNo":"1234567890",
   		"timestamp":1534892332334,
   		"version":"V1.0.0"
  	}  

  String json = “apiKey=XXX&msgNo=1234567890&timestamp=1534892332334&version=V1.0.0”；

  String signature = HmacSHA256.encode (json , secretKey );

  last,the request parameters like:

    	{
   		"apiKey" : "XXX",
   		"msgNo":"1234567890",
   		"timestamp":1534892332334,
   		"version":"V1.0.0",
  	"signature": signature
        }  

## [Api Detail]

### [Normal]

#### 1. query server time

request uri：{requestUrl}/serverTime

request method：GET

request parameter infomation：none

response example：

```
{
"data": 1557134972000,
"success": true,
"msg": "",
"code": "0"
}
```

#### 2. config detail

request uri：{requestUrl}/spot/config

request method：GET

request parameter infomation：null

response description:

| Field          | Description     | Mark | Type   |
| -------------- | --------------- | ---- | ------ |
| spotConfig     | spot config     |      | Object |
| coinConfig     | coin config     |      | Object |
| contractConfig | contract config |      | Object |

coinConfig：

| Field          | Description           | Mark | Type   |
| -------------- | --------------------- | ---- | ------ |
| name           | coin name             |      | String |
| fullName       | full name             |      | String |
| depositStatus  | can deposit           |      | String |
| withdrawStatus | can withdraw          |      | String |
| minWithdraw    | min withdraw amount   |      | String |
| withdrawFee    | withdraw fee          |      | String |
| makerFeeRate   | maker transaction fee |      | String |
| takerFeeRate   | taker transaction fee |      | String |
| minTxAmt       | min transaction amount|      | String |

contractConfig：

| Field        | Description           | Mark | Type   |
| ------------ | --------------------- | ---- | ------ |
| symbol       |                       |      | String |
| makerFeeRate | maker transaction fee |      | String |
| takerFeeRate | taker transaction fee |      | String |

spotConfig：

| Field    | Description | Mark | Type     |
| -------- | ----------- | ---- | -------- |
| symbol   |             |      | String   |
| accuracy |             |      | String[] |

response example：

```
{
    "data": {
        "coinConfig": [
            {
                "makerFeeRate": "0.001",
                "minWithdraw": "10",
                "withdrawFee": "0.1",
                "name": "BXA",
                "depositStatus": "0",
                "fullName": "Exchange Alliance",
                "takerFeeRate": "0.001",
                "withdrawStatus": "0"
            },
            ...
        ],
        "contractConfig": [
            {
                "symbol": "TBTCUSD",
                "makerFeeRate": "-0.00025",
                "takerFeeRate": "0.00075"
            }
        ],
        "spotConfig": [
            {
                "symbol": "BTC-USDT",
                "accuracy": [
                    "2",
                    "6"
                ]
            },
            ...
    ]
},
"code": "0",
"msg": "success",
"timestamp": 1557200664263
}
```

### [Normal Authentication api]

#### 1. withdraw   (**need withdraw authentication**)

request uri: {requestUrl}/withdraw

request method: POST

request parameter infomation:

| Filed       | Description           | Required(Y or N) | Mark                     | Type   |
| ----------- | --------------------- | ---------------- | ------------------------ | ------ |
| coinType    |                       | Y                | e.g BTC                  | String |
| address     | target wallet address | Y                |                          | String |
| extendParam | memo or tag           | N                |                          | String |
| quantity    |                       | Y                |                          | String |
| mark        |                       | Y                | max support for 250 char | String |

#### 2. transfer asset for inner account   (**need withdraw authentication**)

request uri: {requestUrl}/transfer

request method: POST

request parameter infomation:

| Field    | Description                                 | Required(Y or N) | Mark    | Type   |
| -------- | ------------------------------------------- | ---------------- | ------- | ------ |
| coinType |                                             | Y                | e.g BTC | String |
| quantity |                                             | Y                |         | String |
| from     | from account type（WALLET,SPOT,CONTRACT）   | Y                |         | String |
| to       | target account type（WALLET,SPOT,CONTRACT） | Y                |         | String |

### [Normal api for spot]

#### 1. ticker

request uri：{requestUrl}/spot/ticker

request method：GET

request parameter infomation：

| Field  | Description             | Required(Y or N) | Mark                        | Type   |
| ------ | ----------------------- | ---------------- | --------------------------- | ------ |
| symbol | Unique tag(ex:ETH-USDT) | Y                | =ALL,get all symbol tickers | String |

response:

| Field | Description                        | Mark | Type   |
| ----- | ---------------------------------- | ---- | ------ |
| c     | last price in the past of 24 hours |      | String |
| h     | high price in the past of 24 hours |      | String |
| l     | low price in the past of 24 hours  |      | String |
| p     | price change in the past of hours  |      | String |
| v     | deal quantity in the past of hours |      | String |
| s     | symbol                             |      | String |

response example：

	{
	"data": [
	    {
	        "c": "3700.458408",
	        "h": "3700.458408",
	        "l": "3700.458408",
	        "p": "0.0000",
	        "v": "0.00",
	        "s": "BTC-USDT"
	    }
	    ],
	"success": true,
	"msg": "",
	"code": "0",
	"params": []
	}

#### 2. orderbook

request uri：{requestUrl}/spot/orderBook

request method：GET

request parameter infomation：

| Field  | Description             | Required(Y or N) | Mark | Type   |
| ------ | ----------------------- | ---------------- | ---- | ------ |
| symbol | Unique tag(ex:ETH-USDT) | Y                |      | String |

response:

| Field  | Description    | Mark | Type                                      |
| ------ | -------------- | ---- | ----------------------------------------- |
| b      | Bids           |      | String[2]（first:price, second:quantity） |
| s      | Asks           |      | String[2]（first:price, second:quantity） |
| ver    | version number |      | String                                    |
| symbol |                |      | String                                    |

response example：

	{
	"data": {
	    "b": [
	        [
	            "3700.458232",
	            "0.6189503471163566"
	        ]
	    ],
	    "s": [
	        [
	            "3700.458408",
	            "0.4235737788300525"
	        ],
	        [
	            "3700.46623",
	            "0.3"
	        ],
	        [
	            "3700.471159",
	            "0.11"
	        ]
	    ],
	    "ver":"1",
	    "symbol":"ETH-USDT"
	},
	"success": true,
	"msg": "",
	"code": "0",
	"params": []
	}

#### 3. trade records（last 100）

request uri：{requestUrl}/spot/trades

request method：GET

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response:

| Field | Description   | Mark        | Type   |
| ----- | ------------- | ----------- | ------ |
| p     | deal price    |             | String |
| s     | trade type    | buy or sell | String |
| v     | deal quantity |             | String |
| t     | timestamp     |             | String |
| ver   |               |             | String |

response example：
	
	{
	"data": [
		{
		   "p":"3700.458232",
		   "s":"buy",
		   "t":"1552616758",
		   "v":"0.3456"
		}
	 ],
	 "success": true,
	 "msg": "",
	 "code": "0",
	 "params": []
	}

#### 4. kline

request uri：{requestUrl}/spot/kline

request method：GET

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark                                                         | Type   |
| ------ | ----------- | ---------------- | ------------------------------------------------------------ | ------ |
| symbol |             | Y                |                                                              | String |
| type   | kline type  | Y                | m1,m3,m5,m15,m30,h1,h2,h4,h6,h8,h12,d1,d3,w1,M1(m=minute，h=hour,d=day,w=week,M=month) | String |
| start  | start time  | Y                | unit: second                                            | Long   |
| end    | end time    | Y                | unit: second                                            | Long   |

response:

| Field | Description         | Mark | Type   |
| ----- | ------------------- | ---- | ------ |
| c     | close price         |      | String |
| h     | hign price          |      | String |
| l     | low price           |      | String |
| o     | open price          |      | String |
| s     | total deal money    |      | String |
| t     | total deal times    |      | String |
| time  | timestamp           |      | String |
| v     | total deal quantity |      | String |

response example：
	
	{
	"data": [
	     {
		"c":"3700.458232",
		"h":"3700.458232",
		"l":"3700.458232",
		"o":"3700.458232",
		"s":"0.1234",
		"t":"1",
		"time":"1552616758",
		"v":"0.12"
	    }
	  ],
	 "success": true,
	 "msg": "",
	 "code": "0",
	 "params": []
	}

### [Authentication api for spot]

#### 1. create order for virtual coin (**need transaction authentication**)

request uri：{requestUrl}/spot/placeOrder

request method：POST

request parameter infomation：

| Field     | Description | Required(Y or N) | Mark                                          | Type   |
| --------- | ----------- | ---------------- | --------------------------------------------- | ------ |
| symbol    |             | Y                |                                               | String |
| type      | order type  | Y                | limit(limit price) or market(market price) | String |
| side      | order side  | Y                | buy or sell                                   | String |
| price     |             | Y                | when type is market, the value = -1           | String |
| quantity  |             | Y                | eg.BTC-USDT,normally point at the quantity of BTC,when type=market,side=buy,point at the quantity of USDT, and the quantity should greater than or equal to the minimum trading volume of BTC                                | String |
| timestamp |             | Y                |                                               | String |

response description:

| Field   | Description | Mark | Type   |
| ------- | ----------- | ---- | ------ |
| orderId |             |      | String |
| symbol  |             |      | String |

response example：

	{
	"data": {
		"orderId":"23132134242",
		"symbol":"BTC-USDT"
	   },
	"code": "0",
	"msg": "success",
	"timestamp": 1551346473238,
	"params": []
	}

#### 2. cancel order for virtual coin

request uri：{requestUrl}/spot/cancelOrder  (**need transaction authentication**)

request method：POST

request parameter infomation:

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |
| symbol  |             | Y                |      | String |

#### 3. query virtual coin asset account

request uri：{requestUrl}/spot/assetList

request method：POST

request parameter infomation:

| Field     | Description              | Required(Y or N) | Mark                                | Type   |
| --------- | ------------------------ | ---------------- | ----------------------------------- | ------ |
| coinType  | coin type                | N                | if null, response ALL virtual asset | String |
| assetType | asset type (spot,wallet) | Y                | spot for virtual                    | String |

response description:

| Field       | Description        | Mark                    | Type   |
| ----------- | ------------------ | ----------------------- | ------ |
| coinType    | coin type          |                         | String |
| count       | usable amount      |                         | String |
| frozen      | frozen amount      |                         | String |
| btcQuantity | probably equal BTC |                         | String |
| type        | type               | 1=virtual coin，2=legal |        |

response example：

	{
	"data": [
	     {
	      "coinType":"BTC",
		  "count":"100",
		  "frozen":"10",
		  "btcQuantity":"110",
		  "type":"1"
	    }
	  ],
	"code": "0",
	"msg": "success",
	"timestamp": 1551346473238,
	"params": []
	}

#### 4. trade for order detail

request uri：{requestUrl}/spot/orderDetail

request method：POST

request parameter infomation:

| Field   | Description        | Required(Y or N) | Mark         | Type   |
| ------- | ------------------ | ---------------- | ------------ | ------ |
| orderId |                    | Y                |              | String |
| symbol  |                    | Y                |              | String |
| page    | current page       | N                | default = 1  | String |
| count   | current page count | N                | default = 10 | String |

response description:

| Field | Description   | Mark | Type |
| ----- | ------------- | ---- | ---- |
| num   | total numbers |      | Long |
| list  | trade detail  |      | List |

response description (List):

| Field         | Description  | Mark                    | Type   |
| ------------- | ------------ | ----------------------- | ------ |
| orderId       |              |                         | String |
| orderSign     | order status | deal by taker or maker? | String |
| getCount      | get          |                         | String |
| getCountUnit  | coin type    |                         | String |
| loseCount     | lose         |                         | String |
| loseCountUnit | coin type    |                         | String |
| price         | deal price   |                         | String |
| priceUnit     | coin type    |                         | String |
| fee           | deal fee     | fee                     | String |
| feeUnit       | coin type    |                         | String |
| time          | time         |                         | Long   |
| fsymbol       | symbol       | BTC-USDT                | String |
| side          | order side   | buy or sell             | String |

response example：
	
	{
	"data":{
	    "num":"10",
	    "list":[
	         {
		     "orderId":"12300993210",
		     "orderSign":"taker",
	             "getCount":"0.1",
		     "getCountUnit":"BTC",
	             "loseCount":"370.01",
	             "loseCountUnit":"USDT",
		     "price":"3700.01",
		     "priceUnit":"USDT",
		     "fee":"0.0001",
		     "feeUnit":"BTC",
   		     "time":"1552878781",
		     "fsymbol":"BTC-USDT",
		     "side":"buy"
		 },
		 ...
	      ]
	 },
	"code": "0",
	"msg": "success",
	"timestamp": 1551346473238,
	"params": []
    }


#### 5. query history order list

request uri：{requestUrl}/spot/orderList

request method：POST

request parameter information:

| Field      | Description                                                  | Required(Y or N) | Mark | Type   |
| ---------- | ------------------------------------------------------------ | ---------------- | ---- | ------ |
| side       | order side（buy，sell）                                      | Y                |      | String |
| symbol     |                                                              | Y                |      | String |
| status     | order status（traded (history order)）                       | Y                |      | String |
| queryRange | the range of order（thisweek(in 7 day)，thisweekago(before 7 ago)） | Y                |      | String |
| page       | current page                                                 | N                |      | String |
| count      | current page count                                           | N                |      | String |

response description:

| Field | Description   | Mark | Type |
| ----- | ------------- | ---- | ---- |
| num   | total numbers |      | Long |
| list  | orde list     |      | List |

list description：

| Field      | Description        | Mark                           | Type    |
| ---------- | ------------------ | ------------------------------ | ------- |
| orderId    |                    |                                | String  |
| symbol     |                    |                                | String  |
| price      | order price        |                                | decimal |
| tradedNum  | completed quantity |                                | Decimal |
| quantity   | total quantity     |                                | Decimal |
| avgPrice   | average price      |                                | Decimal |
| status     | order status       | send，pending，success，cancel | String  |
| type       | order type         | market，limit                  | String  |
| side       | order side         | buy，sell                      | String  |
| createTime | order create time  |                                | Date    |
| tradeTotal |                    |                                | Decimal |

response example：

```
"data":{
    "num":"10",
    "list":[
         {
	    "orderId":"12300993210",
	    "symbol":"BTC-USDT",
	    "price":"3700",
	    "tradedNum":"0.01",
	    "quantity":"0.5",
	    "avgPrice":"0",
	    "status":"pending",
	    "type":"limit",
	    "side":"buy",
	    "createTime":"1552878781",
	    "tradeTotal":"0.5"
	 },
         ...
      ]
  },
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

#### 6. query single order

request uri：{requestUrl}/spot/singleOrder

request method：POST

request parameter infomation:

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |
| symbol  |             | Y                |      | String |

response description：

| Field      | Description        | Mark                           | Type    |
| ---------- | ------------------ | ------------------------------ | ------- |
| orderId    |                    |                                | String  |
| symbol     |                    |                                | String  |
| price      | order price        |                                | decimal |
| tradedNum  | completed quantity |                                | Decimal |
| quantity   | total quantity     |                                | Decimal |
| avgPrice   | average price      |                                | Decimal |
| status     | order status       | send，pending，success，cancel | String  |
| type       | order type         | market，limit                  | String  |
| side       | order side         | buy，sell                      | String  |
| createTime | create time        |                                | Date    |
| tradeTotal |                    |                                | Decimal |

response example：

```
"data":{
	"orderId":"12300993210",
	"symbol":"BTC-USDT",
	"price":"3700",
	"tradedNum":"0.01",
	"quantity":"0.5",
	"avgPrice":"0",
	"status":"pending",
	"type":"limit",
	"side":"buy",
	"createTime":"1552878781",
	"tradeTotal":"0.5"
  },
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

#### 7. query open order list

request uri：{requestUrl}/spot/openOrders

request method：POST

request parameter information:

| Field  | Description        | Required(Y or N) | Mark | Type   |
| ------ | ------------------ | ---------------- | ---- | ------ |
| symbol |                    | Y                |      | String |
| page   | current page       | N                |      | String |
| count  | current page count | N                |      | String |

response description:

| Field | Description   | Mark | Type |
| ----- | ------------- | ---- | ---- |
| num   | total numbers |      | Long |
| list  | orde list     |      | List |

list description：

| Field      | Description        | Mark                           | Type    |
| ---------- | ------------------ | ------------------------------ | ------- |
| orderId    |                    |                                | String  |
| symbol     |                    |                                | String  |
| price      | order price        |                                | decimal |
| tradedNum  | completed quantity |                                | Decimal |
| quantity   | total quantity     |                                | Decimal |
| avgPrice   | average price      |                                | Decimal |
| status     | order status       | send，pending，success，cancel | String  |
| type       | order type         | market，limit                  | String  |
| side       | order side         | buy，sell                      | String  |
| createTime | order create time  |                                | Date    |
| tradeTotal |                    |                                | Decimal |

response example：

```
"data":{
    "num":"10",
    "list":[
         {
	    "orderId":"12300993210",
	    "symbol":"BTC-USDT",
	    "price":"3700",
	    "tradedNum":"0.01",
	    "quantity":"0.5",
	    "avgPrice":"0",
	    "status":"pending",
	    "type":"limit",
	    "side":"buy",
	    "createTime":"1552878781",
	    "tradeTotal":"0.5"
	 },
         ...
      ]
  },
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

#### 8. query my trades

request uri：{requestUrl}/spot/myTrades

request method：POST

request parameter information:

| Field     | Description       | Required(Y or N) | Mark | Type    |
| --------- | ----------------- | ---------------- | ---- | ------- |
| symbol    |                   | Y                |      | String  |
| startTime | trades start time | N                |      | Long    |
| limit     |                   | N                |      | Integer |

response description:

| Field     | Description | Mark | Type       |
| --------- | ----------- | ---- | ---------- |
| id        |             |      | Long       |
| price     |             |      | BigDecimal |
| amount    |             |      | BigDecimal |
| side      |             |      | String     |
| direction |             |      | String     |
| time      |             |      | Date       |

response example：

```
"data":[
	    "id":1,
	    "price":100.01,
	    "amount":0.1,
	    "side":"buy",
	    "direction":"taker",
	    "time":1557047375000,
      ],
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

### [Normal api for contract]

#### 1. orderBook

request uri：{requestUrl}/contract/orderBook

request method：GET

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response description:

| Field | Description | Mark                                            | Type   |
| ----- | ----------- | ----------------------------------------------- | ------ |
| b     | Bids        | split by ":",first is price, second is quantity | String |
| s     | Asks        | split by ":",first is price, second is quantity | String |
| type  | type        |                                                 | String |

response example：

```
"data":{
	"b":["3701:1","3700:0.5"],
	"s":["3800:0.2","3801.1:0.3"],
	"type":""
   },
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

#### 2. ticker

request uri：{requestUrl}/contract/ticker

request method：GET

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response description:

| Field        | Description                                               | Mark | Type   |
| ------------ | --------------------------------------------------------- | ---- | ------ |
| symbol       | contract symbol                                           |      | String |
| type         | Type                                                      |      | String |
| lastPrice    | last price                                                |      | String |
| high         | high price in the past of 24 hours                        |      | String |
| low          | low price in the past of 24 hours                         |      | String |
| volume       | completed quantity in the past of 24 hours                |      | String |
| change       | need * 100                                                |      | String |
| openValue    | not completed value                                       |      | String |
| fundRate0    | contract fee change value in the next time                |      | String |
| fundTime0    | contract fee change time(million second) in the next time |      | String |
| adlRanker    | ADL range                                                 |      | String |
| ver          | version number                                            |      | String |
| openInterest |                                                           |      | String |
| turnover     |                                                           |      | String |

response example：

```
"data":{
	"symbol":"BTC-USDT",
	"type":"",
	"lastPrice":"3700",
	"high":"3800.1",
	"low":"3699.01",
	"volume":"23",
	"change":"0.0123",
	"openValue":"10",
	"fundRate0":"",
	"fundTime0":"",
	"adlRanker":"",
	"ver":"0",
	"openInterest":"",
	"turnover":""
    },
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

### [Authentication api for contract]

#### 1. create order for contract

request uri：{requestUrl}/contract/order/create

request method：POST

request parameter infomation：

| Field            | Description                 | Required(Y or N) | Mark                                                         | Type   |
| ---------------- | --------------------------- | ---------------- | ------------------------------------------------------------ | ------ |
| property         | current is normal           | Y                |                                                              | String |
| symbol           |                             | Y                | ex: BTCUSD                                                   | String |
| type             |                             | Y                | market or limit                                              | String |
| amount           |                             | Y                | must be whole number                                         | String |
| amountDisplay    | show quantity for ice order | Y                |                                                              | String |
| price            |                             | N                | don't need if type is market, else must be  an integral multiple of tickerPrice. | String |
| side             | order side                  | Y                | buy or sell                                                  | String |
| postOnly         | only do maker?              | Y                | false or true                                                | String |
| reduceOnly       | only for reduce position?   | Y                | false or true                                                | String |
| timeInForce      | order time inforce type     | Y                | default is GTC, 'GTC'（always valid until cancel), 'FOK'（all  completed or cancel）, 'IOC'（completed or cancel fast, or part completed） | String |
| leverage         | leverage value              | N                |                                                              | String |
| triggerPrice     | trigger price               | N                | when property is trigger.                                    | String |
| benchmarkPrice   |                             | N                |                                                              | String |
| triggerPriceType |                             | N                |                                                              | String |

response description:

| Field   | Description | Mark | Type   |
| ------- | ----------- | ---- | ------ |
| orderId |             |      | String |

response example：

```
"data":{
	"orderId":"12314342399321"
  },
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

#### 2. cancel order for contract

request uri：{requestUrl}/contract/order/cancel

request method：POST

request parameter infomation：

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |

response description：when code = "0" is success

#### 3. edit leverage

request uri：{requestUrl}/contract/leverageEdit

request method：POST

request parameter infomation：

| Field    | Description       | Required(Y or N) | Mark | Type   |
| -------- | ----------------- | ---------------- | ---- | ------ |
| symbol   |                   | Y                |      | String |
| leverage | new leverage info | Y                |      | String |

request description：when code="0" is success

#### 4. position info

request uri：{requestUrl}/contract/position

request method：POST

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response description：

| Field            | Description           | Mark | Type   |
| ---------------- | --------------------- | ---- | ------ |
| positionId       |                       |      | String |
| symbol           |                       |      | String |
| amount           | position amount       |      | String |
| margin           | position margin       |      | String |
| positionValue    | position value        |      | String |
| leverage         |                       |      | String |
| status           | position status       |      | String |
| openPositionTime | open position time    |      | String |
| flatPositionTime | flat position time    |      | String |
| realProfit       | completed profit      |      | String |
| liquidation      | force completed price |      | String |
| side             | position side         |      | String |
| frozen           | position frozen       |      | String |

#### 5. adjust margin

request uri：{requestUrl}/contract/margin/update

request method：POST

request parameter infomation：

| Field        | Description                              | Required(Y or N) | Mark | Type   |
| ------------ | ---------------------------------------- | ---------------- | ---- | ------ |
| symbol       |                                          | Y                |      | String |
| changeAmount | amount change with positive and negative | Y                |      | String |

response description：when code="0" is success

#### 6. query asset account for contract

request uri：{requestUrl}/contract/asset/info

request method：POST

request parameter infomation：

| Field      | Description | Required(Y or N) | Mark | Type   |
| ---------- | ----------- | ---------------- | ---- | ------ |
| page       |             | Y                |      | Int    |
| count      |             | Y                |      | Int    |
| coinIdLike | coin type   | N                |      | String |

response description：

| Field    | Description | Mark | Type   |
| -------- | ----------- | ---- | ------ |
| pageInfo |             |      | Object |
| records  |             |      | Array  |

pageInfo:（refer to orderList）

records：

| Field    | Description        | Mark | Type   |
| -------- | ------------------ | ---- | ------ |
| btcValue | probably equal BTC |      | String |
| coinId   | coin type          |      | String |
| count    | usable amount      |      | String |
| frozen   | frozen amount      |      | String |

#### 7. query user contract info

request url：{requestUrl}/contract/info

request method：POST

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response description:

| Field     | Description     | Mark | Type   |
| --------- | --------------- | ---- | ------ |
| symbol    |                 |      | String |
| leverage  | leverage number |      | String |
| fundRate0 |                 |      | String |
| riskLimit | risk limit      |      | String |

#### 8. query user contract account info 

request url：{requestUrl}/contract/account/info

request method：POST

request parameter information：

| Field | Description | Required(Y or N) | Mark | Type   |
| ----- | ----------- | ---------------- | ---- | ------ |
| coin  | coin symbol | Y                |      | String |

response description:

| Field                | Description      | Mark | Type   |
| -------------------- | ---------------- | ---- | ------ |
| coin                 | coin symbol      |      | String |
| totalAmount          | total amount     |      | String |
| remainMargin         | remain margin    |      | String |
| openPositionMargin   |                  |      | String |
| openOrderMarginTotal |                  |      | String |
| availableAmount      | available amount |      | String |

#### 9. query order list (open order or history order list)

request url：{requestUrl}/contract/orders

request method：POST

request parameter information：

| Field  | Description                   | Required(Y or N) | Mark       | Type    |
| ------ | ----------------------------- | ---------------- | ---------- | ------- |
| symbol |                               | Y                |            | String  |
| type   | order type（open or history） | Y                |            | String  |
| page   |                               | N                | default 1  | Integer |
| count  |                               | N                | default 10 | Integer |

response description:

| Field    | Description | Mark | Type   |
| -------- | ----------- | ---- | ------ |
| pageInfo |             |      | Object |
| records  |             |      | Array  |

pageInfo：

| Field       | Description | Mark | Type    |
| ----------- | ----------- | ---- | ------- |
| page        |             |      | Integer |
| count       |             |      | Integer |
| pageTotal   |             |      | Integer |
| recordTotal |             |      | Integer |

records：

| Field      | Description      | Mark                                      | Type   |
| ---------- | ---------------- | ----------------------------------------- | ------ |
| orderId    |                  |                                           | String |
| symbol     |                  |                                           | String |
| type       | order type       | limit or market                           | String |
| side       | order side       | buy or sell                               | String |
| price      | price            | when type=market is 0                     | String |
| amountReal | real amount      |                                           | String |
| amountFill | completed amount |                                           | String |
| status     | order status     | open、cancel、filled、rejected、untrigger | String |
| avgPrice   | average price    |                                           | String |
| time       | create time      |                                           | Long   |

#### 10. query user contract trades

request url：{requestUrl}/contract/trades

request method：POST

request parameter information：

| Field  | Description | Required(Y or N) | Mark       | Type    |
| ------ | ----------- | ---------------- | ---------- | ------- |
| symbol |             | Y                |            | String  |
| page   |             | N                | default=1  | Integer |
| count  |             | N                | default=10 | Integer |

response description:

| Field    | Description | Mark | Type   |
| -------- | ----------- | ---- | ------ |
| pageInfo |             |      | Object |
| records  |             |      | Array  |

pageInfo:(refer to orderList)

records：

| Field   | Description | Mark | Type    |
| ------- | ----------- | ---- | ------- |
| symbol  |             |      | String  |
| orderId |             |      | String  |
| isTaker |             |      | Boolean |
| side    | buy or sell |      | String  |
| fee     |             |      | String  |
| price   |             |      | String  |
| amount  |             |      | String  |
| time    |             |      | String  |
| version |             |      | Long    |



## [code list]

| code  | msg                                          | Description | Mark |
| ----- | -------------------------------------------- | ----------- | ---- |
| 0     | success                                      |             |      |
| 9000  | missing parameter                            |             |      |
| 9001  | version not matched                          |             |      |
| 9002  | verifySignature failed                       |             |      |
| 9004  | access denied                                |             |      |
| 9005  | key expired                                  |             |      |
| 9006  | no server                                    |             |      |
| 9999  | system error                                 |             |      |
|       |                                              |             |      |
|       |                                              |             |      |
| 20043 | price accuracy is wrong for placing order    |             |      |
| 20044 | quantity accuracy is wrong for placing order |             |      |
|       |                                              |             |      |
|       |                                              |             |      |
|       |                                              |             |      |
|       |                                              |             |      |
