catalog

[TOC]

## [General Rest Api Information]

- The base endpoint is: **https://global-openapi.dcex.world/**
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
| version   | version number      | N                | current version number                                       | String |
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

- The part just for authentication of request, the pair of apiKey and secretKey can be apply in website (https://global.dcex.world)

- For authentication of request, the request paramters need to be sign  by HmacSHA256

- The prepare  signature data is a json string which make up of  all request parameters which sort by dict.

  signature example:

  request parameters like:

    	{
   		"apiKey" : "XXXXXXXXXXXX",
   		"bizCode" :"PLACE_ORDER_COIN", 
   		"msgNo":"1234567890",
   		"timestamp":15348923323343,
   		"version":"V1.0.0"
  	}  

  String json = “apiKey=XXXXXXX&bizCode=PLACE_ORDER_SPOT&msgNo=123456789&timestamp=1455323333332&version=v1.0.0”；

  String signature = HmacSHA256.encode (json , secretKey );

  last,the request parameters like:

    	{
   		"apiKey" : "XXXXXXXXXXXX",
   		"bizCode" :"PLACE_ORDER_COIN", 
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
	"info": [
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

| Field | Description | Required(Y or N) | Mark | Type                                      |
| ----- | ----------- | ---------------- | ---- | ----------------------------------------- |
| b     | Bids        | Y                |      | String[2]（first:price, second:quantity） |
| s     | Asks        | Y                |      | String[2]（first:price, second:quantity） |

response example：

	{
	"info": {
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
	    ]
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

response example：
	{
	"info": [
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
| start  | start time  | Y                | 11 number(second)                                            | Long   |
| end    | end time    | Y                | 11 number(second)                                            | Long   |

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
	"info": [
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
request parameter infomationDescription：null
responseDescription:

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
	"info": {
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
request parameter infomationDescription：

| Field     | Description | Required(Y or N) | Mark                                          | Type   |
| --------- | ----------- | ---------------- | --------------------------------------------- | ------ |
| symbol    |             | Y                |                                               | String |
| type      | order type  | Y                | limit (limit price) or  market (market price) | String |
| side      | order side  | Y                | buy or sell                                   | String |
| price     |             | Y                | when type is market, the value = -1           | String |
| quantity  |             | Y                | deal quantity                                 | String |
| timestamp |             | Y                |                                               | String |

responseDescription:

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |
| symbol  |             | Y                |      | String |

response example：

	{
	"info": {
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
request parameter infomationDescription:

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |

#### 3. query virtual coin asset account

request uri：{requestUrl}/spot/assetList
request method：POST
request parameter infomationDescription:

| Field    | Description | Required(Y or N) | Mark                                  | Type   |
| -------- | ----------- | ---------------- | ------------------------------------- | ------ |
| coinType | coin type   | N                | if null, response ALL virtual account | String |

response Description:

| Field       | Description        | Required(Y or N) | Mark                    | Type   |
| ----------- | ------------------ | ---------------- | ----------------------- | ------ |
| coinType    | coin type          | Y                |                         | String |
| count       | usable amount      | Y                |                         | String |
| frozen      | frozen amount      | Y                |                         | String |
| btcQuantity | probably equal BTC | Y                |                         | String |
| type        | Type               | Y                | 1=virtual coin，2=legal |        |

response example：

	{
	"info": [{
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
request parameter infomationDescription:

| Field   | Description        | Required(Y or N) | Mark         | Type   |
| ------- | ------------------ | ---------------- | ------------ | ------ |
| orderId |                    | Y                |              | String |
| page    | current page       | N                | default = 1  | String |
| count   | current page count | N                | default = 10 | String |

response Description:

| Field | Description   | Required(Y or N) | Mark | Type |
| ----- | ------------- | ---------------- | ---- | ---- |
| num   | total numbers | Y                |      | Long |
| list  | trade detail  | Y                |      | List |

response Description (List):

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
	"info":{
	    "num":"10",
	    "list":[{
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


#### 5. order list

request uri：{requestUrl}/orderList
request method：POST
request parameter information:

| Field      | Description                                                  | Required(Y or N) | Mark | Type   |
| ---------- | ------------------------------------------------------------ | ---------------- | ---- | ------ |
| side       | order side（buy，sell）                                      | Y                |      | String |
| coinType   | coin type                                                    | Y                |      | String |
| marketType | market type                                                  | Y                |      | String |
| status     | order status（traded (history order)、trading(open order)）  | Y                |      | String |
| queryRange | the range of order（thisweek(in 7 day)，thisweekago(before 7 ago)） | Y                |      | String |
| page       | current page                                                 | N                |      | String |
| count      | current page count                                           | N                |      | String |

response Description:

| Field      | Description           | Required(Y or N) | Mark | Type |
| ---------- | --------------------- | ---------------- | ---- | ---- |
| num        | total numbers         | Y                |      | Long |
| tradingNum | open order numbers    | Y                |      | Long |
| tradedNum  | history order numbers | Y                |      | Long |
| list       | orde list             | Y                |      | List |

list Description：

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
"info":{
    "num":"10",
    "tradingNum":"10",
    "tradedNum":"100"
    "list":[{
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

#### 6. query for single order

request uri：{requestUrl}/singleOrder
request method：POST
request parameter infomationDescription:

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |

response Description：

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
"info":{
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

### [Normal api for contract]

#### 1.orderBook

request uri：{requestUrl}/contract/orderBook
request method：GET
request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

responseDescription:

| Field | Description | Required(Y or N) | Mark                                            | Type   |
| ----- | ----------- | ---------------- | ----------------------------------------------- | ------ |
| b     | Bids        | Y                | split by ":",first is price, second is quantity | String |
| s     | Asks        | Y                | split by ":",first is price, second is quantity | String |
| type  | Type        |                  |                                                 |        |

response example：

```
"info":{
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

response Description:

| Field     | Description                                               | Required(Y or N) | Mark | Type   |
| --------- | --------------------------------------------------------- | ---------------- | ---- | ------ |
| symbol    | contract symbol                                           | Y                |      | String |
| type      | Type                                                      | Y                |      | String |
| lastPrice | last price                                                | Y                |      | String |
| high      | high price in the past of 24 hours                        | Y                |      | String |
| low       | low price in the past of 24 hours                         | Y                |      | String |
| volume    | completed quantity in the past of 24 hours                | Y                |      | String |
| change    | need * 100                                                | Y                |      | String |
| openValue | not completed value                                       | Y                |      | String |
| fundRate0 | contract fee change value in the next time                | Y                |      | String |
| fundTime0 | contract fee change time(million second) in the next time | Y                |      | String |
| adlRanker | ADL range                                                 | Y                |      | String |
| ver       | version number                                            | Y                |      | String |

response example：

```
"info":{
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
		"ver":
	},
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

### [Authentication api for contract]

#### 1.create order for contract

request uri：{requestUrl}/contract/order
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
| timeinforce      | order time inforce type     | Y                | default is GTC, 'GTC'（always valid until cancel), 'FOK'（all  completed or cancel）, 'IOC'（completed or cancel fast, or part completed） | String |
| leverage         | leverage value              | N                |                                                              | String |
| triggerPrice     | trigger price               | N                | when property is trigger.                                    | String |
| benchmarkPrice   |                             | N                |                                                              | String |
| triggerPriceType |                             | N                |                                                              | String |

response Description:

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |

response example：

```
"info":{
		"orderId":"12314342399321"
	},
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

#### 2.cancel order for contract

request uri：{requestUrl}/contract/orderCancel
request method：POST
request parameter infomation：

| Field   | Description | Required(Y or N) | Mark | Type   |
| ------- | ----------- | ---------------- | ---- | ------ |
| orderId |             | Y                |      | String |

response description：when code = "0" is success

#### 3.query leverage info

request uri：{requestUrl}/contract/leverageInfo
request method：POST
request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |

response description：

| Field    | Description | Required(Y or N) | Mark | Type   |
| -------- | ----------- | ---------------- | ---- | ------ |
| symbol   |             | Y                |      | String |
| leverage |             | Y                |      | String |

#### 4.edit leverage

request uri：{requestUrl}/contract/leverageEdit
request method：POST
request parameter infomation：

| Field    | Description       | Required(Y or N) | Mark | Type   |
| -------- | ----------------- | ---------------- | ---- | ------ |
| symbol   |                   | Y                |      | String |
| leverage | new leverage info | Y                |      | String |

request description：when code="0" is success

#### 5.query order detail

request uri：{requestUrl}/contract/orderDetail
request method：POST
request parameter infomation：

| Field  | Description | Required(Y or N) | Mark | Type   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol |             | Y                |      | String |
| page   |             | Y                |      | Int    |
| count  |             | Y                |      | Int    |

response description：

| Field    | Description | Required(Y or N) | Mark | Type   |
| -------- | ----------- | ---------------- | ---- | ------ |
| pageInfo |             | Y                |      | Object |
| fills    |             | Y                |      | Array  |

pageInfo:(refer to orderList）

fills：

| Field   | Description                 | Required(Y or N) | Mark | Type   |
| ------- | --------------------------- | ---------------- | ---- | ------ |
| orderId |                             | Y                |      | String |
| symbol  |                             | Y                |      | String |
| amount  | completed amount            | Y                |      | String |
| price   | deal price                  | Y                |      | String |
| side    | order side                  | Y                |      | String |
| time    | completed time(millisecond) | Y                |      | String |
| tradeId |                             | Y                |      | String |

#### 6.position info

request uri：{requestUrl}/contract/positionInfo
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

#### 7.adjust margin

request uri：{requestUrl}/contract/marginEdit
request method：POST
request parameter infomationDescription：

| Field        | Description                              | Required(Y or N) | Mark | Type   |
| ------------ | ---------------------------------------- | ---------------- | ---- | ------ |
| symbol       |                                          | Y                |      | String |
| amountChange | amount change with positive and negative | Y                |      | String |

response description：when code="0" is success

#### 8.query asset account for contract

request uri：{requestUrl}/contract/assetInfo
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

#### 9.asset change detail for contract

request uri：{requestUrl}/contract/assetChangeDetail
request method：POST
request parameter infomation：

| Field     | Description      | Required(Y or N) | Mark | Type   |
| --------- | ---------------- | ---------------- | ---- | ------ |
| timeStart | query start time | N                |      | Long   |
| timeEnd   | query end time   | N                |      | Long   |
| coinId    | coin type        | N                |      | String |
| type      |                  | N                |      | String |
| page      |                  | Y                |      | Int    |
| count     |                  | Y                |      | Int    |

request description：

| Field    | Description | Required(Y or N) | Mark | Type   |
| -------- | ----------- | ---------------- | ---- | ------ |
| pageInfo |             | Y                |      | Object |
| records  |             | Y                |      | Array  |

pageInfo:（refer to orderList）

records：

| Field             | Description                 | Required(Y or N) | Mark                      | Type   |
| ----------------- | --------------------------- | ---------------- | ------------------------- | ------ |
| coinId            | coin type                   | Y                |                           | String |
| count             | change amount               | Y                |                           | String |
| fee               |                             | Y                |                           | String |
| frozenChangeCount | frozen amount change        | Y                |                           | String |
| label             |                             | Y                |                           | String |
| side              | order side                  | Y                | -1:out,0:transaction,1:in | String |
| state             | order state                 | Y                | 0:not, 1 completed        | String |
| time              | create time (millionsecond) | Y                |                           | String |
| type              |                             | Y                |                           | String |

#### 10.query open order list

request uri：{requestUrl}/contract/openOrderList
request method：POST
request parameter infomation：

| Field  | Description        | Required(Y or N) | Mark | Type   |
| ------ | ------------------ | ---------------- | ---- | ------ |
| page   | current page       | Y                |      | Int    |
| count  | current page count | Y                |      | Int    |
| symbol |                    | Y                |      | String |

response description:

| Field    | Description | Required(Y or N) | Mark | Type   |
| -------- | ----------- | ---------------- | ---- | ------ |
| pageInfo | page info   | Y                |      | Object |
| records  | record data | Y                |      | Array  |

pageInfo：

| Field       | Description        | Required(Y or N) | Mark | Type |
| ----------- | ------------------ | ---------------- | ---- | ---- |
| page        | current page       | Y                |      | Int  |
| count       | current page count | Y                |      | Int  |
| pageTotal   | total page         | Y                |      | Int  |
| recordTotal | total record count | Y                |      | Int  |

records：

| Field         | Description                           | Required(Y or N) | Mark | Type   |
| ------------- | ------------------------------------- | ---------------- | ---- | ------ |
| orderId       |                                       | Y                |      | String |
| symbol        |                                       | Y                |      | String |
| type          |                                       | Y                |      | String |
| side          |                                       | Y                |      | String |
| timeinforce   |                                       | Y                |      | String |
| amountDisplay |                                       | Y                |      | String |
| price         | order price，type = market is 0       | Y                |      | String |
| amountReal    | real quantity                         | Y                |      | String |
| amountFill    | completed quantity                    | Y                |      | String |
| orderValue    | order value（compute by marketPrice） | Y                |      | String |
| status        | order status                          | Y                |      | String |
| marginTotal   | order total margin                    | Y                |      | String |
| marginUsed    | used margin                           | Y                |      | String |
| startTime     | create time(millionsecond)            | Y                |      | String |
| postOnly      |                                       | Y                |      | String |
| reduceOnly    |                                       | Y                |      | String |
| property      |                                       | Y                |      | String |
| amountRemain  | remain amount                         | Y                |      | String |

#### 11.query history order list

request uri：{requestUrl}/contract/historyOrderList
request method：POST
request parameter infomation：

| Field | Description        | Required(Y or N) | Mark                                                         | Type   |
| ----- | ------------------ | ---------------- | ------------------------------------------------------------ | ------ |
| page  | current page       | Y                |                                                              | Int    |
| count | current page count | Y                |                                                              | Int    |

response description:

| Field    | Description | Required(Y or N) | Mark | Type   |
| -------- | ----------- | ---------------- | ---- | ------ |
| pageInfo | page info   | Y                |      | Object |
| records  | record data | Y                |      | Array  |
| symbol   |             | Y                |      | String |

pageInfo：

| Field       | Description        | Required(Y or N) | Mark | Type |
| ----------- | ------------------ | ---------------- | ---- | ---- |
| page        | current page       | Y                |      | Int  |
| count       | current page count | Y                |      | Int  |
| pageTotal   | total page         | Y                |      | Int  |
| recordTotal | total record count | Y                |      | Int  |

records：

| Field         | Description                           | Required(Y or N) | Mark | Type   |
| ------------- | ------------------------------------- | ---------------- | ---- | ------ |
| orderId       |                                       | Y                |      | String |
| symbol        |                                       | Y                |      | String |
| type          |                                       | Y                |      | String |
| side          |                                       | Y                |      | String |
| timeinforce   |                                       | Y                |      | String |
| amountDisplay |                                       | Y                |      | String |
| price         | order price，type = market is 0       | Y                |      | String |
| amountReal    | real quantity                         | Y                |      | String |
| amountFill    | completed quantity                    | Y                |      | String |
| orderValue    | order value（compute by marketPrice） | Y                |      | String |
| status        | order status                          | Y                |      | String |
| marginTotal   | order total margin                    | Y                |      | String |
| marginUsed    | used margin                           | Y                |      | String |
| startTime     | create time(millionsecond)            | Y                |      | String |
| postOnly      |                                       | Y                |      | String |
| reduceOnly    |                                       | Y                |      | String |
| property      |                                       | Y                |      | String |
| amountRemain  | remain amount                         | Y                |      | String |

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