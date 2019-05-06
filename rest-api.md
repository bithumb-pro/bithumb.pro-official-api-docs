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
| v1.0.0  | include some rest api(virtual coin,contracts) and error code list. | 2019-03 | usable |
|         |                                                              |         |        |

## [Request Parameter Infomation] (Important)

### Public request params

| Field     | Description         | Required(Y or N) | Mark                                                         | Type   |
| --------- | ------------------- | ---------------- | ------------------------------------------------------------ | ------ |
| apiKey    | unique tag for user | N                | apply for the key in website by user.                        | String |
| timestamp | request timestamp   | N                | the paramter will be need in request which need to authenticate, if space of time too long between current and request will be reject. | Long  |
| signature | signature data      | N                | required by authentication of request.                       | String |
| msgNo     | requst id           | N                | just for some api(place order),max length is 50 chars        | String |

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

- The prepare  signature data is a json string which make up of  all request parameters which sort by dict.

  signature example:

  request parameters like:

    	{
   		"apiKey" : "XXXXXXXXXXXX",
   		"msgNo":"1234567890",
   		"timestamp":15348923323343,
   		"version":"V1.0.0"
  	}  

  String json = “apiKey=XXXXXXX&msgNo=123456789&timestamp=1455323333332&version=v1.0.0”；

  String signature = HmacSHA256.encode (json , secretKey );

  last,the request parameters like:

    	{
   		"apiKey" : "XXXXXXXXXXXX",
   		"msgNo":"1234567890",
   		"timestamp":15348923323343,
   		"version":"V1.0.0",
  	"signature": signature
        }  

## [Api Detail]

### [Normal api for spot]

#### 1.ticker

request uri：{requestUrl}/spot/ticker

request method：GET

request parameter infomation：

| Field  | Description             | Required(Y or N) | Mark                        | Type   |
| ------ | ----------------------- | ---------------- | --------------------------- | ------ |
| symbol | Unique tag(ex:ETH-USDT) | Y                | =ALL,get all symbol tickers | String |

response:

| Field | Description                        | Required(Y or N) | Mark | Type   |
| ----- | ---------------------------------- | ---------------- | ---- | ------ |
| c     | last price in the past of 24 hours | Y                |      | String |
| h     | high price in the past of 24 hours | Y                |      | String |
| l     | low price in the past of 24 hours  | Y                |      | String |
| p     | price change in the past of hours  | Y                |      | String |
| v     | deal quantity in the past of hours | Y                |      | String |
| s     | symbol                             | Y                |      | String |

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

#### 2.orderbook

request uri：{requestUrl}/spot/orderBook

request method：GET

request parameter infomation：

| Field  | Description             | Required(Y or N) | Mark | Type   |
| ------ | ----------------------- | ---------------- | ---- | ------ |
| symbol | Unique tag(ex:ETH-USDT) | Y                |      | String |

response:

| Field  | Description    | Required(Y or N) | Mark | Type                                      |
| ------ | -------------- | ---------------- | ---- | ----------------------------------------- |
| b      | Bids           | Y                |      | String[2]（first:price, second:quantity） |
| s      | Asks           | Y                |      | String[2]（first:price, second:quantity） |
| ver    | version number | Y                |      | String                                    |
| symbol |                | Y                |      | String                                    |

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

#### 3.trade records（last 100）

request uri：{requestUrl}/spot/trades

request method：GET

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response:

| Field | Description   | Required(Y or N) | Mark        | Type   |
| ----- | ------------- | ---------------- | ----------- | ------ |
| p     | deal price    | Y                |             | String |
| s     | trade type    | Y                | buy or sell | String |
| v     | deal quantity | Y                |             | String |
| t     | timestamp     | Y                |             | String |
| ver   |               | Y                |             | String |

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

#### 4.kline

request uri：{requestUrl}/spot/kline

request method：GET

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark                                                         | Type   |
| ------ | ----------- | ---------------- | ------------------------------------------------------------ | ------ |
| symbol |             | Y                |                                                              | String |
| type   | kline type  | Y                | m1,m3,m5,m15,m30,h1,h3,h4,h6,h8,h12,d1,d3,w1,M1(m=minute，h=hour,d=day,w=week,M=month) | String |
| start  | start time  | Y                | 10 number(second)                                            | Long   |
| end    | end time    | Y                | 10 number(second)                                            | Long   |

response:

| Field | Description         | Required(Y or N) | Mark | Type   |
| ----- | ------------------- | ---------------- | ---- | ------ |
| c     | close price         | Y                |      | String |
| h     | hign price          | Y                |      | String |
| l     | low price           | Y                |      | String |
| o     | open price          | Y                |      | String |
| s     | total deal money    | Y                |      | String |
| t     | total deal times    | Y                |      | String |
| time  | timestamp           | Y                |      | String |
| v     | total deal quantity | Y                |      | String |

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

#### 5.symbol config

request uri：{requestUrl}/spot/config

request method：GET

request parameter infomation：null

response description:

| Field        | Description       | Required(Y or N) | Mark | Type   |
| ------------ | ----------------- | ---------------- | ---- | ------ |
| symbolConfig | config for symbol | Y                |      | Object |

symbolConfig description：

| Field    | Description | Required(Y or N) | Mark                           | Type                                    |
| -------- | ----------- | ---------------- | ------------------------------ | --------------------------------------- |
| symbol   |             | Y                |                                | String                                  |
| accuracy | accuracy    | Y                | accuracy of price and quantity | String[2](first:price，second:quantity) |

response example：

	{
	"data": {
	    "symbolConfig": [
	        {
	            "symbol": "BTC-USDT",
	            "accuracy": [
	                "8",
	                "8"
	            ]
	        },
	        {
	            "symbol": "ETH-USDT",
	            "accuracy": [
	                "8",
	                "8"
	            ]
	        }
	    ]
	},
	"code": "0",
	"msg": "success",
	"timestamp": 1551346473238,
	"params": []
	}

### [Authentication api for spot]

#### 1. create order for virtual coin

request uri：{requestUrl}/spot/placeOrder

request method：POST

request parameter infomation：

| Field     | Description | Required(Y or N) | Mark                                          | Type   |
| --------- | ----------- | ---------------- | --------------------------------------------- | ------ |
| symbol    |             | Y                |                                               | String |
| type      | order type  | Y                | limit (limit price) or  market (market price) | String |
| side      | order side  | Y                | buy or sell                                   | String |
| price     |             | Y                | when type is market, the value = -1           | String |
| quantity  |             | Y                | deal quantity                                 | String |
| timestamp |             | Y                |                                               | String |

response description:

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |
| symbol  |             | Y                |      | String |

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

request uri：{requestUrl}/spot/cancelOrder

request method：POST

request parameter infomation:

| Field      | Description | Required(Y or N) | Mark | Type   |
| ---------- | ----------- | ---------------- | ---- | ------ |
| orderId    |             | Y                |      | String |
| coinType   |             | Y                |      | String |
| marketType |             | Y                |      | String |

#### 3. query virtual coin asset account

request uri：{requestUrl}/spot/assetList

request method：POST

request parameter infomation:

| Field     | Description              | Required(Y or N) | Mark                                | Type   |
| --------- | ------------------------ | ---------------- | ----------------------------------- | ------ |
| coinType  | coin type                | N                | if null, response ALL virtual asset | String |
| assetType | asset type (spot,wallet) | Y                | spot for virtual                    | String |

response description:

| Field       | Description        | Required(Y or N) | Mark                    | Type   |
| ----------- | ------------------ | ---------------- | ----------------------- | ------ |
| coinType    | coin type          | Y                |                         | String |
| count       | usable amount      | Y                |                         | String |
| frozen      | frozen amount      | Y                |                         | String |
| btcQuantity | probably equal BTC | Y                |                         | String |
| type        | type               | Y                | 1=virtual coin，2=legal |        |

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

#### 4. trade of order detail

request uri：{requestUrl}/spot/orderDetail

request method：POST

request parameter infomation:

| Field      | Description        | Required(Y or N) | Mark         | Type   |
| ---------- | ------------------ | ---------------- | ------------ | ------ |
| orderId    |                    | Y                |              | String |
| coinType   |                    | Y                |              | String |
| marketType |                    | Y                |              | String |
| page       | current page       | N                | default = 1  | String |
| count      | current page count | N                | default = 10 | String |

response description:

| Field | Description   | Required(Y or N) | Mark | Type |
| ----- | ------------- | ---------------- | ---- | ---- |
| num   | total numbers | Y                |      | Long |
| list  | trade detail  | Y                |      | List |

response description (List):

| Field         | Description  | Required(Y or N) | Mark                    | Type   |
| ------------- | ------------ | ---------------- | ----------------------- | ------ |
| orderId       |              | Y                |                         | String |
| orderSign     | order status | Y                | deal by taker or maker? | String |
| getCount      | get          | Y                |                         | String |
| getCountUnit  | coin type    | Y                |                         | String |
| loseCount     | lose         | Y                |                         | String |
| loseCountUnit | coin type    | Y                |                         | String |
| price         | deal price   | Y                |                         | String |
| priceUnit     | coin type    | Y                |                         | String |
| fee           | deal fee     | Y                | fee                     | String |
| feeUnit       | coin type    | Y                |                         | String |
| time          | time         | Y                |                         | Long   |
| fsymbol       | symbol       | Y                | BTC-USDT                | String |
| side          | order side   | Y                | buy or sell             | String |

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
| coinType   | coin type                                                    | Y                |      | String |
| marketType | market type                                                  | Y                |      | String |
| status     | order status（traded (history order)）                       | Y                |      | String |
| queryRange | the range of order（thisweek(in 7 day)，thisweekago(before 7 ago)） | Y                |      | String |
| page       | current page                                                 | N                |      | String |
| count      | current page count                                           | N                |      | String |

response description:

| Field | Description   | Required(Y or N) | Mark | Type |
| ----- | ------------- | ---------------- | ---- | ---- |
| num   | total numbers | Y                |      | Long |
| list  | orde list     | Y                |      | List |

list description：

| Field      | Description        | Required(Y or N) | Mark                           | Type    |
| ---------- | ------------------ | ---------------- | ------------------------------ | ------- |
| orderId    |                    | Y                |                                | String  |
| marketType | market type        | Y                |                                | String  |
| coinType   | coin type          | Y                |                                | String  |
| price      | order price        | Y                |                                | decimal |
| tradedNum  | completed quantity | Y                |                                | Decimal |
| quantity   | total quantity     | Y                |                                | Decimal |
| avgPrice   | average price      | Y                |                                | Decimal |
| status     | order status       | Y                | send，pending，success，cancel | String  |
| type       | order type         | Y                | market，limit                  | String  |
| side       | order side         | Y                | buy，sell                      | String  |
| createTime | order create time  | Y                |                                | Date    |
| tradeTotal |                    | Y                |                                | Decimal |

response example：

```
"data":{
    "num":"10",
    "list":[
         {
	    "orderId":"12300993210",
	    "marketType":"USDT",
	    "coinType":"BTC",
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

| Field      | Description | Required(Y or N) | Mark | Type   |
| ---------- | ----------- | ---------------- | ---- | ------ |
| orderId    |             | Y                |      | String |
| coinType   |             | Y                |      | String |
| marketType |             | Y                |      | String |

response description：

| Field      | Description        | Required(Y or N) | Mark                           | Type    |
| ---------- | ------------------ | ---------------- | ------------------------------ | ------- |
| orderId    |                    | Y                |                                | String  |
| marketType | market type        | Y                |                                | String  |
| coinType   | coin type          | Y                |                                | String  |
| price      | order price        | Y                |                                | decimal |
| tradedNum  | completed quantity | Y                |                                | Decimal |
| quantity   | total quantity     | Y                |                                | Decimal |
| avgPrice   | average price      | Y                |                                | Decimal |
| status     | order status       | Y                | send，pending，success，cancel | String  |
| type       | order type         | Y                | market，limit                  | String  |
| side       | order side         | Y                | buy，sell                      | String  |
| createTime | create time        | Y                |                                | Date    |
| tradeTotal |                    | Y                |                                | Decimal |

response example：

```
"data":{
	"orderId":"12300993210",
	"marketType":"USDT",
   	"coinType":"BTC",
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

| Field | Description   | Required(Y or N) | Mark | Type |
| ----- | ------------- | ---------------- | ---- | ---- |
| num   | total numbers | Y                |      | Long |
| list  | orde list     | Y                |      | List |

list description：

| Field      | Description        | Required(Y or N) | Mark                           | Type    |
| ---------- | ------------------ | ---------------- | ------------------------------ | ------- |
| orderId    |                    | Y                |                                | String  |
| marketType | market type        | Y                |                                | String  |
| coinType   | coin type          | Y                |                                | String  |
| price      | order price        | Y                |                                | decimal |
| tradedNum  | completed quantity | Y                |                                | Decimal |
| quantity   | total quantity     | Y                |                                | Decimal |
| avgPrice   | average price      | Y                |                                | Decimal |
| status     | order status       | Y                | send，pending，success，cancel | String  |
| type       | order type         | Y                | market，limit                  | String  |
| side       | order side         | Y                | buy，sell                      | String  |
| createTime | order create time  | Y                |                                | Date    |
| tradeTotal |                    | Y                |                                | Decimal |

response example：

```
"data":{
    "num":"10",
    "list":[
         {
	    "orderId":"12300993210",
	    "marketType":"USDT",
	    "coinType":"BTC",
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

| Field     | Description | Required(Y or N) | Mark | Type       |
| --------- | ----------- | ---------------- | ---- | ---------- |
| id        |             | Y                |      | Long       |
| price     |             | Y                |      | BigDecimal |
| amount    |             | Y                |      | BigDecimal |
| side      |             | Y                |      | String     |
| direction |             | Y                |      | String     |
| time      |             | Y                |      | Date       |

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

#### 1.orderBook

request uri：{requestUrl}/contract/orderBook

request method：GET

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response description:

| Field | Description | Required(Y or N) | Mark                                            | Type   |
| ----- | ----------- | ---------------- | ----------------------------------------------- | ------ |
| b     | Bids        | Y                | split by ":",first is price, second is quantity | String |
| s     | Asks        | Y                | split by ":",first is price, second is quantity | String |
| type  | type        | Y                |                                                 | String |

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

#### 2.ticker

request uri：{requestUrl}/contract/ticker

request method：GET

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response description:

| Field        | Description                                               | Required(Y or N) | Mark | Type   |
| ------------ | --------------------------------------------------------- | ---------------- | ---- | ------ |
| symbol       | contract symbol                                           | Y                |      | String |
| type         | Type                                                      | Y                |      | String |
| lastPrice    | last price                                                | Y                |      | String |
| high         | high price in the past of 24 hours                        | Y                |      | String |
| low          | low price in the past of 24 hours                         | Y                |      | String |
| volume       | completed quantity in the past of 24 hours                | Y                |      | String |
| change       | need * 100                                                | Y                |      | String |
| openValue    | not completed value                                       | Y                |      | String |
| fundRate0    | contract fee change value in the next time                | Y                |      | String |
| fundTime0    | contract fee change time(million second) in the next time | Y                |      | String |
| adlRanker    | ADL range                                                 | Y                |      | String |
| ver          | version number                                            | Y                |      | String |
| openInterest |                                                           | Y                |      | String |
| turnover     |                                                           | Y                |      | String |

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

#### 1.create order for contract

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

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |

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

#### 2.cancel order for contract

request uri：{requestUrl}/contract/order/cancel

request method：POST

request parameter infomation：

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |

response description：when code = "0" is success

#### 3.edit leverage

request uri：{requestUrl}/contract/leverageEdit

request method：POST

request parameter infomation：

| Field    | Description       | Required(Y or N) | Mark | Type   |
| -------- | ----------------- | ---------------- | ---- | ------ |
| symbol   |                   | Y                |      | String |
| leverage | new leverage info | Y                |      | String |

request description：when code="0" is success

#### 4.position info

request uri：{requestUrl}/contract/position

request method：POST

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response description：

| Field            | Description           | Required(Y or N) | Mark | Type   |
| ---------------- | --------------------- | ---------------- | ---- | ------ |
| positionId       |                       | Y                |      | String |
| symbol           |                       | Y                |      | String |
| amount           | position amount       | Y                |      | String |
| margin           | position margin       | Y                |      | String |
| positionValue    | position value        | Y                |      | String |
| leverage         |                       | Y                |      | String |
| status           | position status       | Y                |      | String |
| openPositionTime | open position time    | Y                |      | String |
| flatPositionTime | flat position time    | Y                |      | String |
| realProfit       | completed profit      | Y                |      | String |
| liquidation      | force completed price | N                |      | String |
| side             | position side         | Y                |      | String |
| frozen           | position frozen       | Y                |      | String |

#### 5.adjust margin

request uri：{requestUrl}/contract/margin/update

request method：POST

request parameter infomation：

| Field        | Description                              | Required(Y or N) | Mark | Type   |
| ------------ | ---------------------------------------- | ---------------- | ---- | ------ |
| symbol       |                                          | Y                |      | String |
| changeAmount | amount change with positive and negative | Y                |      | String |

response description：when code="0" is success

#### 6.query asset account for contract

request uri：{requestUrl}/contract/asset/info

request method：POST

request parameter infomation：

| Field      | Description | Required(Y or N) | Mark | Type   |
| ---------- | ----------- | ---------------- | ---- | ------ |
| page       |             | Y                |      | Int    |
| count      |             | Y                |      | Int    |
| coinIdLike | coin type   | N                |      | String |

response description：

| Field    | Description | Required(Y or N) | Mark | Type   |
| -------- | ----------- | ---------------- | ---- | ------ |
| pageInfo |             | Y                |      | Object |
| records  |             | Y                |      | Array  |

pageInfo:（refer to orderList）

records：

| Field    | Description        | Required(Y or N) | Mark | Type   |
| -------- | ------------------ | ---------------- | ---- | ------ |
| btcValue | probably equal BTC | Y                |      | String |
| coinId   | coin type          | Y                |      | String |
| count    | usable amount      | Y                |      | String |
| frozen   | frozen amount      | Y                |      | String |

#### 7.query user contract info

request url：{requestUrl}/contract/info

request method：POST

request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response description:

| Field     | Description     | Required(Y or N) | Mark | Type   |
| --------- | --------------- | ---------------- | ---- | ------ |
| symbol    |                 | Y                |      | String |
| leverage  | leverage number | Y                |      | String |
| fundRate0 |                 | Y                |      | String |
| riskLimit | risk limit      | Ys               |      | String |

#### 8.query user contract account info 

request url：{requestUrl}/contract/account/info

request method：POST

request parameter information：

| Field | Description | Required(Y or N) | Mark | Type   |
| ----- | ----------- | ---------------- | ---- | ------ |
| coin  | coin symbol | Y                |      | String |

response description:

| Field                | Description      | Required(Y or N) | Mark | Type   |
| -------------------- | ---------------- | ---------------- | ---- | ------ |
| coin                 | coin symbol      | Y                |      | String |
| totalAmount          | total amount     | Y                |      | String |
| remainMargin         | remain margin    | Y                |      | String |
| openPositionMargin   |                  | Y                |      | String |
| openOrderMarginTotal |                  | Y                |      | String |
| availableAmount      | available amount | Y                |      | String |

#### 9.query order list (open order or history order list)

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

| Field    | Description | Required(Y or N) | Mark | Type   |
| -------- | ----------- | ---------------- | ---- | ------ |
| pageInfo |             | Y                |      | Object |
| records  |             | Y                |      | Array  |

pageInfo：

| Field       | Description | Required(Y or N) | Mark | Type    |
| ----------- | ----------- | ---------------- | ---- | ------- |
| page        |             | Y                |      | Integer |
| count       |             | Y                |      | Integer |
| pageTotal   |             | Y                |      | Integer |
| recordTotal |             | Y                |      | Integer |

records：

| Field      | Description      | Required(Y or N) | Mark                                      | Type   |
| ---------- | ---------------- | ---------------- | ----------------------------------------- | ------ |
| orderId    |                  | Y                |                                           | String |
| symbol     |                  | Y                |                                           | String |
| type       | order type       | Y                | limit or market                           | String |
| side       | order side       | Y                | buy or sell                               | String |
| price      | price            | Y                | when type=market is 0                     | String |
| amountReal | real amount      | Y                |                                           | String |
| amountFill | completed amount | Y                |                                           | String |
| status     | order status     | Y                | open、cancel、filled、rejected、untrigger | String |
| avgPrice   | average price    | Y                |                                           | String |
| time       | create time      | Y                |                                           | Long   |



## [code list]

| code | msg                    | Description | Mark |
| ---- | ---------------------- | ----------- | ---- |
| 0    | success                |             |      |
| 9000 | missing parameter      |             |      |
| 9001 | version not matched    |             |      |
| 9002 | verifySignature failed |             |      |
| 9003 | msgNo is exist         |             |      |
| 9004 | access denied          |             |      |
| 9005 | key expired            |             |      |
| 9006 | no server              |             |      |
| 9999 | system error           |             |      |
|      |                        |             |      |
|      |                        |             |      |
|      |                        |             |      |
|      |                        |             |      |
|      |                        |             |      |
|      |                        |             |      |
|      |                        |             |      |
|      |                        |             |      |
