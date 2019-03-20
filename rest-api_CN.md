目录

[TOC]

## [REST API简介]

- 本篇列出REST接口的baseurl **https://global-openapi.dcex.world/**
- 所有接口的响应都是JSON格式
- GET`方法的接口, 参数必须在`query string`中发送
- POST方法的接口，参数必须在request body中发送(application/json)
- 对参数的顺序不做要求。

| 版本号 | 说明                               | 时间        | 备注 |
| ------ | ---------------------------------- | ----------- | ---- |
| v1.0.0 | 币币交易、合约交易，参考业务编号表 | 2019年 02月 | 可用 |
|        |                                    |             |      |

## [请求参数说明] (重要)

### 公共请求参数

| 字段      | 说明              | 必填(是/否/可选) | 备注                                    | 类型   |
| --------- | ----------------- | ---------------- | --------------------------------------- | ------ |
| apiKey    | 用户账号(api专用) | 是               | 在用户申请界面申请                      | String |
| timestamp | 时间戳毫秒        | 是               | 毫秒时间戳                              | Long   |
| signature | 签名数据          | 是               | 对应业务数据签名数据,参考[数据认证签名] | String |
| version   | 版本号            | 是               | 当前调用Api 版本号 [REST API简介]       | String |
| msgNo     | 请求流水          | 是               | 用户生成 (50位以内显示长度 唯一字符串)  | String |

### 返回结果说明

如果 查询有分页条件(page, count)接口 返回数据格式

​	{

​           "num":5  // 数据总数量

​	    "list":[{"":""}……...]   查询数据 集合

​	}



## [数据签名] (重要)

- 用户计算签名的基于哈希的协议，此处使用 HmacSHA256  (签名参数 , secretKey(用户申请页面获取))

  String signature = HmacSHA256 . encode ( jsonStringData , secretKey );

- 签名数据(signature) :签名数据由 公共请求参数中 除signature 字段 按照key的字典顺序来排序(升序)组成jsonString 进行 签名

- 签名事例:

  {

  ​	"apiKey" : "XXXXXXXXXXXX", 	// (用户申请页面获取)

  ​	"bizCode" :"PLACE_ORDER_COIN", 

  ​	"msgNo":"1234567890", 参考[请求参数说明]

  ​	"timestamp":15348923323343,

  ​	"version":"V1.0.0"	 // 参考  [REST API简介] ]

  }   字典顺序排序(升序)    生成 源串:

  apiKey=XXXXXXX&bizCode=PLACE_ORDER_SPOT&msgNo=123456789&timestamp=1455323333332&version=v1.0.0

  然后通过HmacSHA256 使用secretKey生成 signature,并将signature加到请求参数中.

## [请求与应答] (重要)

- POST 请求 Content-Type:application/json

- 请求参数事例:

  {
  ​	"timestamp":"1455323333332",
  ​	"apiKey":"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX", //(用户申请页面获取)
  ​	"bizCode":"PLACE_ORDER_COIN", 
  ​        "version":"v1.0.0"，
  ​	"signature":"1EF13F23D123A3123GXXXXXXXXXXXXXXXXXXXXXXXXX"	//参考[数据签名]
  }

- 应答事例:

{
    "code": "0", //响应码，参考[api响应码]
    "msg": "success",
    "data": ""
}

## [接口说明]

### [现货普通接口]

#### 1.行情

请求路径：{requestUrl}/spot/ticker

请求方式：GET

请求参数说明：

| 字段   | 说明                         | 必填(是/否/可选) | 备注                                                         | 类型   |
| ------ | ---------------------------- | ---------------- | ------------------------------------------------------------ | ------ |
| symbol | 币对符号，格式是 coin-market | 是               | 前面交易币种,后面是交易市场     (BTC-USDT)，或者为ALL时，取所有交易对。 | String |

返回结果说明:

| 字段 | 说明             | 必填(是/否/可选) | 备注 | 类型   |
| ---- | ---------------- | ---------------- | ---- | ------ |
| c    | 24小时最新成交价 | 是               |      | String |
| h    | 24小时最高价     | 是               |      | String |
| l    | 24小时最低价     | 是               |      | String |
| p    | 24小时涨跌幅     | 是               |      | String |
| v    | 24小时成交量     | 是               |      | String |
| s    | 交易对           | 是               |      | String |

响应示例：

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

#### 2.订单薄

请求路径：{requestUrl}/spot/orderBook

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |

返回结果说明:

| 字段 | 说明                 | 必填(是/否/可选) | 备注 | 类型                                    |
| ---- | -------------------- | ---------------- | ---- | --------------------------------------- |
| b    | 买方向订单薄（bids） | 是               |      | String[2]（第一个为价格，第二个为数量） |
| s    | 卖方向订单薄（asks） | 是               |      | String[2]（第一个为价格，第二个为数量） |

响应示例：

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

#### 3.交易记录（最新100条）

请求路径：{requestUrl}/spot/trades

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |

返回结果说明:

| 字段 | 说明     | 必填(是/否/可选) | 备注 | 类型   |
| ---- | -------- | ---------------- | ---- | ------ |
| p    | 成交价格 | 是               |      | String |
| s    | 交易类型 | 是               |      | String |
| v    | 成交量   | 是               |      | String |
| t    | 时间戳   | 是               |      | String |

响应示例：
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

#### 4.k线

请求路径：{requestUrl}/spot/kline

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                                         | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT)                   | String |
| type   | k线类型                       | 是               | m1,m3,m5,m15,m30,h1,h3,h4,h6,h8,h12,d1,d3,w1,M1(m为分钟，h为小时，d为天，w为周，M为月) | String |
| start  | 开始时间                      | 是               | 11位到秒                                                     | Long   |
| end    | 结束时间                      | 是               | 11位到秒                                                     | Long   |

返回结果说明:

| 字段 | 说明         | 必填(是/否/可选) | 备注 | 类型   |
| ---- | ------------ | ---------------- | ---- | ------ |
| c    | 收盘价       | 是               |      | String |
| h    | 最高价       | 是               |      | String |
| l    | 最低价       | 是               |      | String |
| o    | 开盘价       | 是               |      | String |
| s    | 累计成交额   | 是               |      | String |
| t    | 累计成交笔数 | 是               |      | String |
| time | 时间戳       | 是               |      | String |
| v    | 累计成交量   | 是               |      | String |

响应示例：
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

#### 5.交易对配置

请求路径：{requestUrl}/spot/config

请求方式：GET

请求参数说明：无

返回结果说明:

| 字段         | 说明           | 必填(是/否/可选) | 备注 | 类型 |
| ------------ | -------------- | ---------------- | ---- | ---- |
| symbolConfig | 交易对精度配置 | 是               |      | 对象 |

symbolConfig对象说明：

| 字段     | 说明   | 必填(是/否/可选) | 备注                   | 类型                                          |
| -------- | ------ | ---------------- | ---------------------- | --------------------------------------------- |
| symbol   | 交易对 | 是               |                        | String                                        |
| accuracy | 精度   | 是               | 包含价格精度和数量精度 | String[2](第一个为价格精度，第二个为数量精度) |

响应示例：

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

### [现货认证接口]

#### 1. 币币下单

请求路径：{requestUrl}/spot/placeOrder

请求方式：POST

请求参数说明：

| 字段      | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| --------- | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol    | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |
| type      | 交易类型 (limit or market)    | 是               | limit 限价  market 市价                    | String |
| side      | 交易方向 (buy or sell)        | 是               | buy 买入  sell 卖出                        | String |
| price     | 价格                          | 是               | type为 市价(limit)交易时 固定值 "-1"       | String |
| quantity  | 数量                          | 是               | 交易数量                                   | String |
| timestamp | 时间戳                        | 是               |                                            | String |

返回结果说明:

| 字段    | 说明           | 必填(是/否/可选) | 备注                        | 类型   |
| ------- | -------------- | ---------------- | --------------------------- | ------ |
| orderId | 返回平台订单号 | 是               | 供撤单查询使用 不支持 msgNo | String |
| symbol  | 返回币对符号   | 是               |                             | String |

响应示例：

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

#### 2. 币币订单撤销

请求路径：{requestUrl}/spot/cancelOrder

请求方式：POST

请求参数说明:

| 字段    | 说明   | 必填(是/否/可选) | 备注       | 类型   |
| ------- | ------ | ---------------- | ---------- | ------ |
| orderId | 订单号 | 是               | 由平台返回 | String |

#### 3. 币币交易账户资产查询

请求路径：{requestUrl}/spot/assetList

请求方式：POST

请求参数说明:

| 字段     | 说明     | 必填(是/否/可选) | 备注                 | 类型   |
| -------- | -------- | ---------------- | -------------------- | ------ |
| coinType | 币种类型 | 可选             | 如 BTC（不传查所有） | String |

返回结果说明:

| 字段        | 说明     | 必填(是/否/可选) | 备注             | 类型   |
| ----------- | -------- | ---------------- | ---------------- | ------ |
| coinType    | 币种类型 | 可选             | 不传查所有       | String |
| count       | 可用数量 | 是               |                  | String |
| frozen      | 冻结数量 | 是               |                  | String |
| btcQuantity | btc估值  | 是               |                  | String |
| type        | 类型     | 是               | 1=虚拟币，2=法币 |        |

响应信息：

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

#### 4. 订单交易明细

请求路径：{requestUrl}/spot/orderDetail

请求方式：POST

请求参数说明:

| 字段    | 说明     | 必填(是/否/可选) | 备注                  | 类型   |
| ------- | -------- | ---------------- | --------------------- | ------ |
| orderId | 订单id   | 是               | 平台返回 不选查询所有 | String |
| page    | 页码     | 可选             | 不选默认1             | String |
| count   | 显示条数 | 可选             | 不选默认10            | String |

返回结果说明:

| 字段 | 说明         | 必填(是/否/可选) | 备注 | 类型 |
| ---- | ------------ | ---------------- | ---- | ---- |
| num  | 总条数       | 是               |      | Long |
| list | 交易明细列表 | 是               |      | List |

返回结果说明 (List):

| 字段          | 说明             | 必填(是/否/可选) | 备注                   | 类型   |
| ------------- | ---------------- | ---------------- | ---------------------- | ------ |
| orderId       | 订单号           | 是               |                        | String |
| orderSign     | 当前订单当前状态 |                  | 当前make还是taker 成交 | String |
| getCount      | 获得数量         | 是               |                        | String |
| getCountUnit  | 获得数量单位     | 是               | 币种                   | String |
| loseCount     | 失去数量         | 是               |                        | String |
| loseCountUnit | 失去数量         | 是               | 币种                   | String |
| price         | 成交价格         | 是               |                        | String |
| priceUnit     | 成交价格单位     | 是               | 币种                   | String |
| fee           | 手续费           | 是               | 手续费                 | String |
| feeUnit       | 手续费单位       | 是               | 币种                   | String |
| time          | 时间             | 是               |                        | Long   |
| fsymbol       | 交易币对         | 是               | BTC-USDT               | String |
| side          | 方向             | 是               | 买 or 卖               | String |

响应示例：
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


#### 5. 订单列表

请求路径：{requestUrl}/orderList

请求方式：POST

请求参数说明:

| 字段       | 说明                                                  | 必填(是/否/可选) | 备注 | 类型   |
| ---------- | ----------------------------------------------------- | ---------------- | ---- | ------ |
| side       | 订单买卖类型（buy，sell）                             | 是               |      | String |
| coinType   | 币种类型                                              | 是               |      | String |
| marketType | 市场类型                                              | 是               |      | String |
| status     | 订单类型（traded=历史单据，trading=委托单据）         | 是               |      | String |
| queryRange | 查询订单的范围（thisweek=本周，thisweekago=本周以前） | 是               |      | String |
| page       | 页码                                                  | 否               |      | String |
| count      | 每一页的显示条数                                      | 否               |      | String |

返回结果说明:

| 字段       | 说明             | 必填(是/否/可选) | 备注 | 类型 |
| ---------- | ---------------- | ---------------- | ---- | ---- |
| num        | 总条数           | 是               |      | Long |
| tradingNum | 进行中的订单数量 | 是               |      | Long |
| tradedNum  | 历史订单数量     | 是               |      | Long |
| list       | 订单列表         | 是               |      | List |

list说明：

| 字段       | 说明         | 必填(是/否/可选) | 备注                           | 类型    |
| ---------- | ------------ | ---------------- | ------------------------------ | ------- |
| orderId    | 订单id       | 是               |                                | String  |
| marketType | 市场类型     | 是               |                                | String  |
| coinType   | 币种类型     | 是               |                                | String  |
| price      | 挂单价格     | 是               |                                | decimal |
| tradedNum  | 已成交数量   | 是               |                                | Decimal |
| quantity   | 挂单数量     | 是               |                                | Decimal |
| avgPrice   | 平均成交价格 | 是               |                                | Decimal |
| status     | 订单状态     | 是               | send，pending，success，cancel | String  |
| type       | 订单类型     | 是               | market，limit                  | String  |
| side       | 订单方向     | 是               | buy，sell                      | String  |
| createTime | 挂单时间     | 是               |                                | Date    |
| tradeTotal | 委托总额     | 是               |                                | Decimal |

响应示例：

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

#### 6. 单个订单查询

请求路径：{requestUrl}/singleOrder

请求方式：POST

请求参数说明:

| 字段    | 说明         | 必填(是/否/可选) | 备注 | 类型   |
| ------- | ------------ | ---------------- | ---- | ------ |
| orderId | 订单唯一编号 | 是               |      | String |

返回结果说明：

| 字段       | 说明         | 必填(是/否/可选) | 备注                           | 类型    |
| ---------- | ------------ | ---------------- | ------------------------------ | ------- |
| orderId    | 订单id       | 是               |                                | String  |
| marketType | 市场类型     | 是               |                                | String  |
| coinType   | 币种类型     | 是               |                                | String  |
| price      | 挂单价格     | 是               |                                | decimal |
| tradedNum  | 已成交数量   | 是               |                                | Decimal |
| quantity   | 挂单数量     | 是               |                                | Decimal |
| avgPrice   | 平均成交价格 | 是               |                                | Decimal |
| status     | 订单状态     | 是               | send，pending，success，cancel | String  |
| type       | 订单类型     | 是               | market，limit                  | String  |
| side       | 订单方向     | 是               | buy，sell                      | String  |
| createTime | 挂单时间     | 是               |                                | Date    |
| tradeTotal | 委托总额     | 是               |                                | Decimal |

响应示例：

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

### [合约普通接口]

#### 1.订单薄

请求路径：{requestUrl}/contract/orderBook

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |

返回结果说明:

| 字段 | 说明               | 必填(是/否/可选) | 备注                                    | 类型   |
| ---- | ------------------ | ---------------- | --------------------------------------- | ------ |
| b    | 买方向订单薄(bids) | 是               | （以:分割，第一个为价格，第二个为数量） | String |
| s    | 卖方向订单薄(asks) | 是               | （以:分割，第一个为价格，第二个为数量） | String |
| type | 类型               |                  |                                         |        |

响应示例：

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

#### 2.行情

请求路径：{requestUrl}/contract/ticker

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |

返回结果说明:

| 字段      | 说明                             | 必填(是/否/可选) | 备注 | 类型   |
| --------- | -------------------------------- | ---------------- | ---- | ------ |
| symbol    | 合约符号                         | 是               |      | String |
| type      | 类型                             | 是               |      | String |
| lastPrice | 最新成交价                       | 是               |      | String |
| high      | 24小时最高                       | 是               |      | String |
| low       | 24小时最低                       | 是               |      | String |
| volume    | 24小时成交量                     | 是               |      | String |
| change    | 需要*100才是百分数               | 是               |      | String |
| openValue | 未平仓数量                       | 是               |      | String |
| fundRate0 | 下次合约费率交换值               | 是               |      | String |
| fundTime0 | 下次费率交换的时间(时间戳到毫秒) | 是               |      | String |
| adlRanker | ADL排位区间                      | 是               |      | String |
| ver       | 版本号                           | 是               |      | String |

响应示例：

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

### [合约认证接口]

#### 1.合约下单

请求路径：{requestUrl}/contract/order

请求方式：POST

请求参数说明：

| 字段             | 说明                            | 必填(是/否/可选) | 备注                                                         | 类型   |
| ---------------- | ------------------------------- | ---------------- | ------------------------------------------------------------ | ------ |
| property         | 合约类型，目前仅支持normal单    | 是               |                                                              | String |
| symbol           | 合约符号，如BTCUSD              | 是               |                                                              | String |
| type             | 订单类型                        | 是               | market或者limit                                              | String |
| amount           | 数量                            | 是               | （只能是整数）                                               | String |
| amountDisplay    | 显示数量（用于冰山单）          | 是               |                                                              | String |
| price            | 价格                            | 否               | market不需要（必须是tickerPrice的整数倍）                    | String |
| side             | 订单方向                        | 是               | buy或者sell                                                  | String |
| postOnly         | 是否只下post单（是否只做maker） | 是               | false或者true                                                | String |
| reduceOnly       | 是否只下reduce单（是否只减仓）  | 是               | false或true                                                  | String |
| timeinforce      | 订单时效类型（默认GTC）         | 是               | 'GTC'（一直有效直到取消），'FOK'（全部成交或取消）， 'IOC'（立即成交或取消，可以是部分成交） | String |
| leverage         | 杠杆值                          | 否               |                                                              | String |
| triggerPrice     | 触发价格                        | 否               | trigger单需填写（用于触发单）                                | String |
| benchmarkPrice   | 触发订单字段。                  | 否               | 下单时候的价格，用于判断是价格是向上还是向下穿越触发         | String |
| triggerPriceType | 触发价格的类型                  | 否               | mark（mark price类型）/index（正常的指数价格）/last（最新成交价） | String |

返回结果说明:

| 字段    | 说明           | 必填(是/否/可选) | 备注                        | 类型   |
| ------- | -------------- | ---------------- | --------------------------- | ------ |
| orderId | 返回平台订单号 | 是               | 供撤单查询使用 不支持 msgNo | String |

响应示例：

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

#### 2.合约取消订单

请求路径：{requestUrl}/contract/orderCancel

请求方式：POST

请求参数说明：

| 字段    | 说明   | 必填(是/否/可选) | 备注 | 类型   |
| ------- | ------ | ---------------- | ---- | ------ |
| orderId | 订单号 | 是               |      | String |

返回参数说明：code = "0"为成功

#### 3.查询杠杆信息

请求路径：{requestUrl}/contract/leverageInfo

请求方式：POST

请求参数说明：

| 字段   | 说明     | 必填(是/否/可选) | 备注 | 类型   |
| ------ | -------- | ---------------- | ---- | ------ |
| symbol | 合约符号 | 是               |      | String |

返回参数说明：

| 字段     | 说明               | 必填(是/否/可选) | 备注 | 类型   |
| -------- | ------------------ | ---------------- | ---- | ------ |
| symbol   | 合约符号           | 是               |      | String |
| leverage | 杠杆值的字符串形式 | 是               |      | String |

#### 4.修改杠杆

请求路径：{requestUrl}/contract/leverageEdit

请求方式：POST

请求参数说明：

| 字段     | 说明                 | 必填(是/否/可选) | 备注 | 类型   |
| -------- | -------------------- | ---------------- | ---- | ------ |
| symbol   | 合约符号             | 是               |      | String |
| leverage | 请求修改的合约杠杆值 | 是               |      | String |

返回参数说明：code="0"为成功

#### 5.查询订单列表

请求路径：{requestUrl}/contract/orderList

请求方式：POST

请求参数说明：

| 字段  | 说明       | 必填(是/否/可选) | 备注                                                         | 类型   |
| ----- | ---------- | ---------------- | ------------------------------------------------------------ | ------ |
| tab   | 类别       | 是               | normal(new-新订单，open-待完成，canceling-取消中), all（new-新订单，open-待完成，canceling-取消中，canceled-取消，filled-盈满，rejected-拒绝） | String |
| page  | 分页当前页 | 是               |                                                              | int    |
| count | 每页记录数 | 是               |                                                              | int    |

返回参数说明:

| 字段     | 说明     | 必填(是/否/可选) | 备注 | 类型 |
| -------- | -------- | ---------------- | ---- | ---- |
| pageInfo | 分页信息 | 是               |      | 对象 |
| records  | 记录数组 | 是               |      | 数组 |

pageInfo：

| 字段        | 说明         | 必填(是/否/可选) | 备注 | 类型 |
| ----------- | ------------ | ---------------- | ---- | ---- |
| page        | 分页当前页数 | 是               |      | int  |
| count       | 每页记录数   | 是               |      | int  |
| pageTotal   | 总页数       | 是               |      | int  |
| recordTotal | 总记录数     | 是               |      | int  |

records数组中的元素参数：

| 字段          | 说明                                    | 必填(是/否/可选) | 备注 | 类型   |
| ------------- | --------------------------------------- | ---------------- | ---- | ------ |
| orderId       | 订单ID                                  | 是               |      | String |
| symbol        | 合约符号                                | 是               |      | String |
| type          | 下单类型                                | 是               |      | String |
| side          | 下单方向                                | 是               |      | String |
| timeinforce   | 时效模式                                | 是               |      | String |
| amountDisplay | 显示数量                                | 是               |      | String |
| price         | 价格，market单为0                       | 是               |      | String |
| amountReal    | 真实数量                                | 是               |      | String |
| amountFill    | 成交数量                                | 是               |      | String |
| orderValue    | 订单价值（当前marketPrice，估值多少钱） | 是               |      | String |
| status        | 订单状态                                | 是               |      | String |
| marginTotal   | 订单总保证金                            | 是               |      | String |
| marginUsed    | 订单已使用保证金                        | 是               |      | String |
| startTime     | 下单时间的时间戳                        | 是               |      | String |
| postOnly      | 是否post only 单                        | 是               |      | String |
| reduceOnly    | 是否 reduce only 单                     | 是               |      | String |
| property      | 合约类型                                | 是               |      | String |
| amountRemain  | 剩余数量                                | 是               |      | String |

#### 6.查询订单详情

请求路径：{requestUrl}/contract/orderDetail

请求方式：POST

请求参数说明：

| 字段   | 说明       | 必填(是/否/可选) | 备注 | 类型   |
| ------ | ---------- | ---------------- | ---- | ------ |
| symbol | 合约符号   | 是               |      | String |
| page   | 分页当前页 | 是               |      | int    |
| count  | 每页记录数 | 是               |      | int    |

返回参数说明：

| 字段     | 说明           | 必填(是/否/可选) | 备注 | 类型 |
| -------- | -------------- | ---------------- | ---- | ---- |
| pageInfo | 返回的分页信息 | 是               |      | 对象 |
| fills    | 返回的记录数组 | 是               |      | 数组 |

pageInfo：通用分页对象（见orderList）

fills数组中元素参数：

| 字段    | 说明             | 必填(是/否/可选) | 备注 | 类型   |
| ------- | ---------------- | ---------------- | ---- | ------ |
| orderId | 订单ID           | 是               |      | String |
| symbol  | 合约符号         | 是               |      | String |
| amount  | 成交数量         | 是               |      | String |
| price   | 成交价格         | 是               |      | String |
| side    | 方向             | 是               |      | String |
| time    | 成交时间的时间戳 | 是               |      | String |
| tradeId | 交易ID           | 是               |      | String |

#### 7.仓位信息

请求路径：{requestUrl}/contract/positionInfo

请求方式：POST

请求参数说明：

| 字段   | 说明     | 必填(是/否/可选) | 备注 | 类型   |
| ------ | -------- | ---------------- | ---- | ------ |
| symbol | 合约符号 | 是               |      | String |

返回参数说明：

| 字段             | 说明             | 必填(是/否/可选) | 备注 | 类型   |
| ---------------- | ---------------- | ---------------- | ---- | ------ |
| positionId       | 仓位ID           | 是               |      | String |
| symbol           | 合约符号         | 是               |      | String |
| amount           | 仓位数量         | 是               |      | String |
| margin           | 仓位保证金       | 是               |      | String |
| positionValue    | 仓位价值         | 是               |      | String |
| leverage         | 杠杆             | 是               |      | String |
| status           | 仓位状态         | 是               |      | String |
| openPositionTime | 开仓时间的时间戳 | 是               |      | String |
| flatPositionTime | 平仓时间的时间戳 | 是               |      | String |
| realProfit       | 已实现盈亏       | 是               |      | String |
| liquidation      | 强平价           | 否               |      | String |
| side             | 仓位方向         | 是               |      | String |
| frozen           | 仓位冻结金       | 是               |      | String |

#### 8.调整保证金

请求路径：{requestUrl}/contract/marginEdit

请求方式：POST

请求参数说明：

| 字段         | 说明                         | 必填(是/否/可选) | 备注 | 类型   |
| ------------ | ---------------------------- | ---------------- | ---- | ------ |
| symbol       | 合约符号                     | 是               |      | String |
| amountChange | 变更数量（带正负号表示方向） | 是               |      | String |

返回参数说明：code="0"为成功

#### 9.合约资产查询

请求路径：{requestUrl}/contract/assetInfo

请求方式：POST

请求参数说明：

| 字段       | 说明                                           | 必填(是/否/可选) | 备注 | 类型   |
| ---------- | ---------------------------------------------- | ---------------- | ---- | ------ |
| page       | 分页当前页                                     | 是               |      | int    |
| count      | 分页每页记录数                                 | 是               |      | int    |
| coinIdLike | 币种类型，前端已经要求将模糊查询改成精确查询。 | 否               |      | String |

返回参数说明：

| 字段     | 说明           | 必填(是/否/可选) | 备注 | 类型 |
| -------- | -------------- | ---------------- | ---- | ---- |
| pageInfo | 返回的分页信息 | 是               |      | 对象 |
| records  | 返回的记录数组 | 是               |      | 数组 |

pageInfo:通用分页参数（见orderList）

records数组中的元素参数：

| 字段     | 说明         | 必填(是/否/可选) | 备注 | 类型   |
| -------- | ------------ | ---------------- | ---- | ------ |
| btcValue | BTC估值      | 是               |      | String |
| coinId   | 币种类型     | 是               |      | String |
| count    | 资产数量     | 是               |      | String |
| frozen   | 冻结资产数量 | 是               |      | String |

#### 10.合约资产变更详情

请求路径：{requestUrl}/contract/assetChangeDetail

请求方式：POST

请求参数说明：

| 字段      | 说明             | 必填(是/否/可选) | 备注 | 类型   |
| --------- | ---------------- | ---------------- | ---- | ------ |
| timeStart | 查询限制起始时间 | 否               |      | Long   |
| timeEnd   | 查询限制结束时间 | 否               |      | Long   |
| coinId    | 币种类型         | 否               |      | String |
| type      | 变更类型         | 否               |      | String |
| page      | 分页当前页数     | 是               |      | int    |
| count     | 分页每页记录数   | 是               |      | int    |

返回参数说明：

| 字段     | 说明           | 必填(是/否/可选) | 备注 | 类型 |
| -------- | -------------- | ---------------- | ---- | ---- |
| pageInfo | 返回的分页信息 | 是               |      | 对象 |
| records  | 返回的记录数组 | 是               |      | 数组 |

pageInfo:通用分页参数（见orderList）

records数组中元素参数：

| 字段              | 说明                             | 必填(是/否/可选) | 备注 | 类型   |
| ----------------- | -------------------------------- | ---------------- | ---- | ------ |
| coinId            | 币种类型                         | 是               |      | String |
| count             | 变更数量                         | 是               |      | String |
| fee               | 手续费                           | 是               |      | String |
| frozenChangeCount | 冻结金额变化                     | 是               |      | String |
| label             | 标签                             | 是               |      | String |
| side              | 方向，0为交易产生，1转入，-1转出 | 是               |      | String |
| state             | 状态，已完成1，未完成0           | 是               |      | String |
| time              | 产生记录的时间的时间戳           | 是               |      | String |
| type              | 类型                             | 是               |      | String |

## [应答码对照表]

| code | msg                    | 说明         | 备注     |
| ---- | ---------------------- | ------------ | -------- |
| 0    | success                | 成功         | 请求成功 |
| 9000 | missing parameter      | 缺少参数     |          |
| 9001 | version not matched    | 版本号不对   |          |
| 9002 | verifySignature failed | 验证签名失败 |          |
| 9003 | msgNo is exist         |              |          |
| 9004 | access denied          | 请求不允许   |          |
| 9005 | key expired            | apiKey过期   |          |
| 9006 | no server              | 无服务       |          |
| 9999 | system error           | 系统异常     |          |
|      |                        |              |          |
|      |                        |              |          |
|      |                        |              |          |
|      |                        |              |          |
|      |                        |              |          |
|      |                        |              |          |
|      |                        |              |          |
|      |                        |              |          |