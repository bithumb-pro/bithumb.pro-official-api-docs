目录

[TOC]

## [REST API简介]

- 本篇列出REST接口的base-endpoint：[**https://global-openapi.bithumb.pro/openapi/v1**][base-endpoint]
- 所有接口的响应都是JSON格式
- GET方法的接口, 参数必须在query string中发送
- POST方法的接口，参数必须在request body中发送(headers里需添加content-type:application/json)
- 对参数的顺序不做要求。

| 版本号 | 说明                               | 时间        | 备注 |
| ------ | ---------------------------------- | ----------- | ---- |
| v1.0.0 | 币币交易、合约交易，参考业务编号表 | 2019年 02月 | 可用 |
|        |                                    |             |      |

## [请求参数说明] (重要)

### 公共请求参数

| 字段      | 说明              | 必填(是/否/可选) | 备注                                                         | 类型   |
| --------- | ----------------- | ---------------- | ------------------------------------------------------------ | ------ |
| apiKey    | 用户账号(api专用) | 否               | 需要认证的请求（必须），在用户申请界面申请                                           | String |
| timestamp | 时间戳毫秒        | 否               | 需要认证的请求（必须），会校验用户的请求时间戳，和当前时间戳相隔太长的请求会被拒绝 | Long   |
| signature | 签名数据          | 否               | 需要认证的请求（必须），对应业务数据签名数据,参考[数据认证签名]                      | String |
| msgNo     | 请求流水          | 否               | 由用户生成的请求串（50个字符长度以内的字符串），在某些请求下会响应给用户（下单接口） | String |

### 返回结果说明

对于分页查询接口，返回的数据格式如下：

​	{

​           "num":5  // 数据总数量

​	    "list":[{"":""}……...]   查询数据 集合

​	}

## [数据签名] (重要)

- 用户计算签名的基于哈希的协议，此处使用 HmacSHA256(签名参数, secretKey(用户申请页面获取))

  String signature = HmacSHA256.encode(signString, secretKey)

- 待签名字符串(signString)：签名数据由客户端的所有请求参数(不为空)按照key的字典顺序来组成，具体示例如下

- 签名示例:

  {

  ​	"apiKey" : "XXXXXXXXXXXX", 	// (用户申请页面获取)

  ​	"msgNo":"1234567890", 参考[请求参数说明]

  ​	"timestamp":15348923323343,

  }   
  待签名字符串(signString)：apiKey=XXXXXXX&msgNo=123456789&timestamp=1455323333332
  
  在获得signString后，通过HmacSHA256 使用secretKey生成signature,并将signature加到请求参数中.

## [请求与应答] (重要)

- 对于POST请求，请求headers里需要有Content-Type:application/json

- 请求参数事例:

  {
  ​	"timestamp":"1455323333332",
  ​	"apiKey":"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX", //(用户申请页面获取)
  ​	"signature":"1EF13F23D123A3123GXXXXXXXXXXXXXXXXXXXXXXXXX"	//参考[数据签名]
  }

- 应答事例:

 {
    "code": "0", //响应码，参考[api响应码]
    "msg": "success",
    "data": ""
  }

## [接口说明]

### [普通接口]

#### 1. 系统时间

请求路径：[url][base-endpoint]/serverTime

请求方式：GET

请求参数说明：无

响应示例：

```
{
"data": 1557134972000,
"success": true,
"msg": "",
"code": "0"
}
```

#### 2. 配置详情

请求路径：{base-endpoint}/spot/config

请求方式：GET

请求参数说明：无

返回结果说明:

| 字段           | 说明     | 备注 | 类型   |
| -------------- | -------- | ---- | ------ |
| spotConfig     | 现货配置 |      | Object |
| coinConfig     | 币种配置 |      | Object |
| contractConfig | 合约配置 |      | Object |

coinConfig对象：

| 字段           | 说明                        | 备注 | 类型   |
| -------------- | --------------------------- | ---- | ------ |
| name           | 币种名称                    |      | String |
| fullName       | 全称                        |      | String |
| depositStatus  | 是否可充值                  |      | String |
| withdrawStatus | 是否可提                    |      | String |
| minWithdraw    | 最小提币数量                |      | String |
| withdrawFee    | 提币手续费                  |      | String |
| makerFeeRate   | 交易时作为maker，交易手续费 |      | String |
| takerFeeRate   | 交易时作为taker，交易手续费 |      | String |
| minTxAmt       | 最小交易量                  |      | String |

contractConfig对象：

| 字段         | 说明                        | 备注 | 类型   |
| ------------ | --------------------------- | ---- | ------ |
| symbol       | 交易对                      |      | String |
| makerFeeRate | 交易时作为maker，交易手续费 |      | String |
| takerFeeRate | 交易时作为taker，交易手续费 |      | String |

spotConfig对象：

| 字段     | 说明   | 备注                   | 类型                                          |
| -------- | ------ | ---------------------- | --------------------------------------------- |
| symbol   | 交易对 |                        | String                                        |
| accuracy | 精度   | 包含价格精度和数量精度 | String[2](第一个为价格精度，第二个为数量精度) |
| openTime | 开盘时间   |  | Long |
| openPrice | 开盘价   |  | Long |
| percentPrice | 价格振幅系数  | 表示为一个瞬时的涨跌停限制，不允许价格瞬间剧烈浮动，以当前成交价格为基准，第一笔交易，以开盘价为基准 | Object |

响应示例：

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
                ],
		"openTime":1565766000000,
		"percentPrice":
		{
		  "multiplierDown":"0.8",
		  "multiplierUp":"1.2"
		},
		"openPrice":"0"
            },
            ...
    ]
},
"code": "0",
"msg": "success",
"timestamp": 1557200664263
}
```

### [普通认证接口]

#### 1. 提币（**需要提币权限**）

请求路径：{base-endpoint}/withdraw

请求方式：POST

请求参数说明：

| 字段        | 说明        | 必填(是/否/可选) | 备注                 | 类型   |
| ----------- | ----------- | ---------------- | -------------------- | ------ |
| coinType    | 币种类型    | 是               | 例如:BTC             | String |
| address     | 提币地址    | 是               | 外部的虚拟币钱包地址 | String |
| extendParam | memo或者tag | 否               |                      | String |
| quantity    | 数量        | 是               |                      | String |
| mark        | 备注        | 是               | 最多支持250个字符    | String |

#### 2. 内部账户资产划转（**需要提币权限**）

请求路径：{base-endpoint}/transfer

请求方式：POST

请求参数说明：

| 字段     | 说明                                                         | 必填(是/否/可选) | 备注     | 类型   |
| -------- | ------------------------------------------------------------ | ---------------- | -------- | ------ |
| coinType | 币种类型                                                     | 是               | 例如:BTC | String |
| quantity | 数量                                                         | 是               |          | String |
| from     | 来源类型（SPOT=币币账户，LEVER=杠杆账户） | 是               |          | String |
| to       | 目标类型（SPOT=币币账户，LEVER=杠杆账户） | 是               |          | String |

### [现货普通接口]

#### 1. 行情

请求路径：{base-endpoint}/spot/ticker

请求方式：GET

请求参数说明：

| 字段   | 说明                         | 必填(是/否/可选) | 备注                                                         | 类型   |
| ------ | ---------------------------- | ---------------- | ------------------------------------------------------------ | ------ |
| symbol | 币对符号，格式是 coin-market | 是               | 前面交易币种,后面是交易市场     (BTC-USDT)，或者为ALL时，取所有交易对。 | String |

返回结果说明:

| 字段 | 说明             | 备注 | 类型   |
| ---- | ---------------- | ---- | ------ |
| c    | 24小时最新成交价 |      | String |
| h    | 24小时最高价     |      | String |
| l    | 24小时最低价     |      | String |
| p    | 24小时涨跌幅     |      | String |
| v    | 24小时成交量     |      | String |
| s    | 交易对           |      | String |

响应示例：

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

#### 2. 订单薄

请求路径：{base-endpoint}/spot/orderBook

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |

返回结果说明:

| 字段 | 说明                 | 备注         | 类型                                    |
| ---- | -------------------- | ------------ | --------------------------------------- |
| b    | 买方向订单薄（bids） |              | String[2]（第一个为价格，第二个为数量） |
| s    | 卖方向订单薄（asks） |              | String[2]（第一个为价格，第二个为数量） |
| ver  | 版本号               | 防止消息回溯 | String                                  |

响应示例：

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
	    ]，
	    "ver":"1"
	},
	"success": true,
	"msg": "",
	"code": "0",
	"params": []
	}

#### 3. 交易记录（最新100条）

请求路径：{base-endpoint}/spot/trades

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |

返回结果说明:

| 字段 | 说明     | 备注 | 类型   |
| ---- | -------- | ---- | ------ |
| p    | 成交价格 |      | String |
| s    | 交易类型 |      | String |
| v    | 成交量   |      | String |
| t    | 时间戳   |      | String |
| ver  | 版本号   |      | String |

响应示例：
	
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

#### 4. k线

请求路径：{base-endpoint}/spot/kline

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                                         | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT)                   | String |
| type   | k线类型                       | 是               | m1,m3,m5,m15,m30,h1,h2,h4,h6,h8,h12,d1,d3,w1,M1(m为分钟，h为小时，d为天，w为周，M为月) | String |
| start  | 开始时间                      | 是               | 单位：秒                                                     | Long   |
| end    | 结束时间                      | 是               | 单位：秒                                                     | Long   |

返回结果说明:

| 字段 | 说明         | 备注 | 类型   |
| ---- | ------------ | ---- | ------ |
| c    | 收盘价       |      | String |
| h    | 最高价       |      | String |
| l    | 最低价       |      | String |
| o    | 开盘价       |      | String |
| s    | 累计成交额   |      | String |
| t    | 累计成交笔数 |      | String |
| time | 时间戳       |      | String |
| v    | 累计成交量   |      | String |

响应示例：
	
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

### [现货认证接口]


#### 1. 币币下单 （**需要交易权限**）

请求路径：{base-endpoint}/spot//placeOrder

请求方式：POST

请求参数说明：

| 字段      | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| --------- | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol    | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |
| type      | 交易类型 (limit or market)    | 是               | limit=限价，market=市价                    | String |
| side      | 交易方向 (buy or sell)        | 是               | buy=买入，sell=卖出                        | String |
| price     | 价格                          | 是               | type为市价(market)交易时，固定值 "-1"（**该字段类型为String**）      | String |
| quantity  | 数量                          | 是               | 交易数量（**该字段类型为String**），当type=market，side=buy时，该字段为计价货币数量，其余均为token,并且数量必须大于等于token的最小交易量                                   | String |
| timestamp | 时间戳                        | 是               |                                            | String |

返回结果说明:

| 字段    | 说明           | 备注                        | 类型   |
| ------- | -------------- | --------------------------- | ------ |
| orderId | 返回平台订单号 | 供撤单查询使用 不支持 msgNo | String |
| symbol  | 返回币对符号   |                             | String |

响应示例：

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

#### 2. 币币订单撤销（**需要交易权限**）

请求路径：{base-endpoint}/spot/cancelOrder

请求方式：POST

请求参数说明:

| 字段    | 说明     | 必填(是/否/可选) | 备注       | 类型   |
| ------- | -------- | ---------------- | ---------- | ------ |
| orderId | 订单号   | 是               | 由平台返回 | String |
| symbol  | 币种类型 | 是               |            | String |

#### 3. 币币交易账户资产查询

请求路径：{base-endpoint}/spot/assetList

请求方式：POST

请求参数说明:

| 字段      | 说明                                          | 必填(是/否/可选) | 备注                 | 类型   |
| --------- | --------------------------------------------- | ---------------- | -------------------- | ------ |
| coinType  | 币种类型                                      | 可选             | 如 BTC（不传查所有） | String |
| assetType | 资产类型(spot=现货虚拟资产, wallet=钱包资产 ) | 是               |                      |        |

返回结果说明:

| 字段        | 说明     | 备注             | 类型   |
| ----------- | -------- | ---------------- | ------ |
| coinType    | 币种类型 | 不传查所有       | String |
| count       | 可用数量 |                  | String |
| frozen      | 冻结数量 |                  | String |
| btcQuantity | btc估值  |                  | String |
| type        | 类型     | 1=虚拟币，2=法币 |        |

响应信息：

	{
	"data": [{
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

请求路径：{base-endpoint}/spot/orderDetail

请求方式：POST

请求参数说明:

| 字段    | 说明     | 必填(是/否/可选) | 备注                  | 类型   |
| ------- | -------- | ---------------- | --------------------- | ------ |
| orderId | 订单id   | 是               |                        | String |
| symbol  |          | 是               |                       | String |
| page    | 页码     | 可选             | 不选默认1             | String |
| count   | 显示条数 | 可选             | 不选默认10            | String |

返回结果说明:

| 字段 | 说明         | 备注 | 类型 |
| ---- | ------------ | ---- | ---- |
| num  | 总条数       |      | Long |
| list | 交易明细列表 |      | List |

返回结果说明 (List):

| 字段          | 说明             | 备注                   | 类型   |
| ------------- | ---------------- | ---------------------- | ------ |
| orderId       | 订单号           |                        | String |
| orderSign     | 当前订单当前状态 | 当前make还是taker 成交 | String |
| getCount      | 获得数量         |                        | String |
| getCountUnit  | 获得数量单位     | 币种                   | String |
| loseCount     | 失去数量         |                        | String |
| loseCountUnit | 失去数量         | 币种                   | String |
| price         | 成交价格         |                        | String |
| priceUnit     | 成交价格单位     | 币种                   | String |
| fee           | 手续费           | 手续费                 | String |
| feeUnit       | 手续费单位       | 币种                   | String |
| time          | 时间             |                        | Long   |
| fsymbol       | 交易币对         | BTC-USDT               | String |
| side          | 方向             | 买 or 卖               | String |

响应示例：
	
	{
	"data":{
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


#### 5. 查询历史订单列表

请求路径：{base-endpoint}/spot/orderList

请求方式：POST

请求参数说明:

| 字段       | 说明                                                  | 必填(是/否/可选) | 备注 | 类型    |
| ---------- | ----------------------------------------------------- | ---------------- | ---- | ------- |
| side       | 订单买卖类型（buy，sell）                             | 是               |      | String  |
| symbol     |                                                       | 是               |      | String  |
| status     | 订单类型（traded=历史单据）                           | 是               |      | String  |
| queryRange | 查询订单的范围（thisweek=本周，thisweekago=本周以前） | 是               |      | String  |
| page       | 页码                                                  | 否               |      | Integer |
| count      | 每一页的显示条数                                      | 否               |      | Integer |

返回结果说明:

| 字段 | 说明     | 备注 | 类型 |
| ---- | -------- | ---- | ---- |
| num  | 总条数   |      | Long |
| list | 订单列表 |      | List |

list说明：

| 字段       | 说明         | 备注                           | 类型    |
| ---------- | ------------ | ------------------------------ | ------- |
| orderId    | 订单id       |                                | String  |
| symbol     |              |                                | String  |
| price      | 挂单价格     |                                | Decimal |
| tradedNum  | 已成交数量   |                                | Decimal |
| quantity   | 挂单数量     |                                | Decimal |
| avgPrice   | 平均成交价格 |                                | Decimal |
| status     | 订单状态     | send，pending，success，cancel | String  |
| type       | 订单类型     | market，limit                  | String  |
| side       | 订单方向     | buy，sell                      | String  |
| createTime | 挂单时间     |                                | Date    |
| tradeTotal | 委托总额     |                                | Decimal |

响应示例：

```
"data":{
    "num":"10",
    "list":[{
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

#### 6. 单个订单查询

请求路径：{base-endpoint}/spot/singleOrder

请求方式：POST

请求参数说明:

| 字段    | 说明         | 必填(是/否/可选) | 备注 | 类型   |
| ------- | ------------ | ---------------- | ---- | ------ |
| orderId | 订单唯一编号 | 是               |      | String |
| symbol  |              | 是               |      | String |

返回结果说明：

| 字段       | 说明         | 备注                           | 类型    |
| ---------- | ------------ | ------------------------------ | ------- |
| orderId    | 订单id       |                                | String  |
| symbol     |              |                                | String  |
| price      | 挂单价格     |                                | Decimal |
| tradedNum  | 已成交数量   |                                | Decimal |
| quantity   | 挂单数量     |                                | Decimal |
| avgPrice   | 平均成交价格 |                                | Decimal |
| status     | 订单状态     | send，pending，success，cancel | String  |
| type       | 订单类型     | market，limit                  | String  |
| side       | 订单方向     | buy，sell                      | String  |
| createTime | 挂单时间     |                                | Date    |
| tradeTotal | 委托总额     |                                | Decimal |

响应示例：

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

#### 7. 查询未成交订单列表

请求路径：{base-endpoint}/spot/openOrders

请求方式：POST

请求参数说明:

| 字段   | 说明             | 必填(是/否/可选) | 备注 | 类型    |
| ------ | ---------------- | ---------------- | ---- | ------- |
| symbol |                  | 是               |      | String  |
| page   | 页码             | 否               |      | Integer |
| count  | 每一页的显示条数 | 否               |      | Integer |

返回结果说明:

| 字段 | 说明     | 备注 | 类型 |
| ---- | -------- | ---- | ---- |
| num  | 总条数   |      | Long |
| list | 订单列表 |      | List |

list说明：

| 字段       | 说明         | 备注                           | 类型    |
| ---------- | ------------ | ------------------------------ | ------- |
| orderId    | 订单id       |                                | String  |
| symbol     | 市场类型     |                                | String  |
| price      | 挂单价格     |                                | Decimal |
| tradedNum  | 已成交数量   |                                | Decimal |
| quantity   | 挂单数量     |                                | Decimal |
| avgPrice   | 平均成交价格 |                                | Decimal |
| status     | 订单状态     | send，pending，success，cancel | String  |
| type       | 订单类型     | market，limit                  | String  |
| side       | 订单方向     | buy，sell                      | String  |
| createTime | 挂单时间     |                                | Date    |
| tradeTotal | 委托总额     |                                | Decimal |

响应示例：

```
"data":{
    "num":"10",
    "list":[{
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

#### 8. 查询我的成交记录

请求路径：{base-endpoint}/spot/myTrades

请求方式：POST

请求参数说明:

| 字段      | 说明             | 必填(是/否/可选) | 备注 | 类型    |
| --------- | ---------------- | ---------------- | ---- | ------- |
| symbol    |                  | 是               |      | String  |
| startTime | 指定成交起始时间 | 是               | 毫秒 | Long    |
| limit     | 显示条数         | 是               |      | Integer |

返回结果说明:

| 字段      | 说明                       | 备注 | 类型       |
| --------- | -------------------------- | ---- | ---------- |
| id        |                            |      | Long       |
| price     | 成交价格                   |      | Decimal |
| amount    | 成交量                     |      | Decimal |
| side      | 成交类型（buy or sell）    |      | String     |
| direction | 成交角色（taker or maker） |      | String     |
| time      | 成交时间                   |      | Date       |

响应示例：

```
"data":[
			"id":"12300993210",
			"price":"100.01",
			"amount":"0.1",
			"side":"buy",
			"direction":"taker",
			"time": 1557047375000
			},
			...
		],
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```



### [杠杆资产认证接口]

#### 1. 杠杆资产列表

请求路径：{base-endpoint}/lever/assetList

请求方式： POST

请求参数说明：

| 字段       | 类型   | 是否必填 | 说明                                                       |
| ---------- | ------ | -------- | ---------------------------------------------------------- |
| page       | 整数型 | 否       | 分页页数，默认1                                            |
| count      | 整数型 | 否       | 分页每页记录数，默认10                                     |
| coinIdLike | 字符串 | 否       | 币种模糊查询字符，不传默认查询当前柜台全部杠杆配置币种情况 |

返回结果说明：

| 字段     | 类型 | 是否必填 | 说明           |
| -------- | ---- | -------- | -------------- |
|  pandect     | 对象 | 是 | 账户总览 |
| pageInfo | 对象 | 是       | 返回的分页信息 |
| records  | 数组 | 是       | 返回的记录数组 |

**pandect**

| 字段         | 类型   | 是否必填 | 说明          |
| ------------ | ------ | -------- | ------------- |
| assetsBTC    | 字符串 | 是       | 总资产折合BTC |
| liabilityBTC | 字符串 | 是       | 负债折合BTC   |
| netBTC       | 字符串 | 是       | 净资产折合BTC |

**pageInfo**

| 字段        | 类型   | 是否必填 | 说明       |
| ----------- | ------ | -------- | ---------- |
| page        | 数字型 | 是       | 分页当前页 |
| count       | 数字型 | 是       | 每页记录数 |
| pageTotal   | 数字型 | 是       | 总页数     |
| recordTotal | 数字型 | 是       | 总记录数   |

**records数组中元素参数**

| 字段              | 类型 | 是否必填 | 说明 |
| ----------------- | ---- | -------- | ---- |
| coinId | 字符串 | 是 | 币种ID |
| totalAmount | 字符串 | 是 | 账户资产 |
| availableAmount | 字符串 | 是 | 可用资产 |
| borrowedAmount | 字符串 | 是 | 借入资产 |
| interestAmount | 字符串 | 是 | 应付利息 |
| inetrestRate | 字符串 | 是 | 日利率(返回为百分后的数字，即返回1代表1%) |
| maxAmountBeTransfer | 字符串 | 是 | 最大可转出额度 |
| totalBTC | 字符串 | 是 | 账户资产的BTC估值 |

响应示例：
```json
{
    "code":"0",
    "msg":"success",
    "info":{
        "pandect":{
            "assetsBTC":"1.213",
            "liabilityBTC":"2131",
            "netBTC":"1211"
        },
        "pageInfo":{
            "page":1,
            "count":10,
            "pageTotal":1,
            "recordTotal":2
        },
        "records":[
            {
                "coinId":"BTC",
                "totalAmount":"123",
                "availableAmount":"0",
                "borrowedAmount":"0",
                "interestAmount":"123",
                "inetrestRate":"10",
                "maxAmountBeTransfer":"1231",
                "totalBTC":"123"
            },
            {
                "coinId":"TBTC",
                "totalAmount":"101.99274098",
                "availableAmount":"0",
                "borrowedAmount":"0",
                "interestAmount":"123",
                "inetrestRate":"10",
                "maxAmountBeTransfer":"1231",
                "totalBTC":"0"
            }
        ]
    },
    "params":[

    ]
}
```

#### 2. 杠杆账户总览
请求路径：{base-endpoint}/lever/assetInfo

请求方式： POST

请求参数说明：

| 字段       | 类型   | 是否必填 | 说明                                                       |
| ---------- | ------ | -------- | ---------------------------------------------------------- |
| symbol	    | 字符串 | 是 | 币对符号 |

返回结果说明：

| 字段     | 类型 | 是否必填 | 说明           |
| -------- | ---- | -------- | -------------- |
| totalAmount    | 字符串 | 是       | 总资产(杠杆账户内所有币种市场价值折合成USDT的总和)           |
| netAmount      | 字符串 | 是       | 净资产(总资产-借贷资产-应付利息)                             |
| fullMargin     | 字符串 | 是       | 全额保证金(用户需要借贷额外资产时杠杆账户里所必须持有的保证金) |
| maintainMargin | 字符串 | 是       | 维持保证金(临界强制平仓对应的保证金)                         |
| leverage       | 字符串 | 是       | 当前币对的杠杆值                                             |
| cushionRate    | 字符串 | 是       | 维持担保比例=缓冲率(缓冲率 = 净资产/最低保证金，区间为0 - 10000%。缓冲率到达120%发送爆仓预警邮件，到达100%平台可随时强制平仓) |


响应示例：
```json
{
    "code":"0",
    "msg":"success",
    "info":{
        "totalAmount":"123",
        "netAmount":"123",
        "fullMargin":"123",
        "maintainMargin":"231",
        "leverage":"2.0",
        "cushionRate":"10"
    },
    "params":[

    ]
}
```


#### 3. 杠杆账户资产变更记录

请求路径：{base-endpoint}/lever/assetHistory

请求方式： POST

请求参数说明：

| 字段       | 类型   | 是否必填 | 说明                                                       |
| ---------- | ------ | -------- | ---------------------------------------------------------- |
| page       | 整数型 | 否       | 分页页数，默认1                                            |
| count      | 整数型 | 否       | 分页每页记录数，默认10                                     |
| type | 字符串 | 否 | 变更类型 |
| coinIdLike | 字符串 | 否       | 币种模糊查询字符，不传默认查询当前柜台全部杠杆配置币种情况 |

返回结果说明：

| 字段     | 类型 | 是否必填 | 说明           |
| -------- | ---- | -------- | -------------- |
| pageInfo | 对象 | 是       | 返回的分页信息 |
| records  | 数组 | 是       | 返回的记录数组 |

**pageInfo**

| 字段        | 类型   | 是否必填 | 说明       |
| ----------- | ------ | -------- | ---------- |
| page        | 数字型 | 是       | 分页当前页 |
| count       | 数字型 | 是       | 每页记录数 |
| pageTotal   | 数字型 | 是       | 总页数     |
| recordTotal | 数字型 | 是       | 总记录数   |

**records数组中元素参数**

| 字段              | 类型 | 是否必填 | 说明 |
| ----------------- | ---- | -------- | ---- |
| coinId | 字符串 | 是 | 币种ID |
| totalAmountChange | 字符串 | 是 | 账户资产变化 |
| availableAmountChange | 字符串 | 是 | 可用资产变化 |
| borrowedAmountChange | 字符串 | 是 | 借入资产变化 |
| interestAmountChange | 字符串 | 是 | 应付利息变化 |
| maxAmountBeBorrowedChange | 字符串 | 是 | 最大可借数量变化 |
| maxAmountTransferChange | 字符串 | 是 | 最大可转出数量变化 |
| type | 字符串 | 是 | 变更类型 |
| time | 整数型 | 是 | 变更时间的时间戳 |
| state | 字符串 | 是 | 状态 |

响应示例：
```json
{
    "code":"0",
    "msg":"success",
    "info":{
        "pageInfo":{
            "page":1,
            "count":10,
            "pageTotal":1,
            "recordTotal":2
        },
        "records":[
            {
                "coinId":"BTC",
                "totalAmountChange":"123",
                "availableAmountChange":"0",
                "borrowedAmountChange":"0",
                "interestAmountChange":"123",
                "maxAmountBeBorrowedChange":"10",
                "maxAmountTransferChange":"1231",
                "type":"123",
                "time":31231312312313,
                "state":"1"
            },
            {
                "coinId":"BTC",
                "totalAmountChange":"123",
                "availableAmountChange":"0",
                "borrowedAmountChange":"0",
                "interestAmountChange":"123",
                "maxAmountBeBorrowedChange":"10",
                "maxAmountTransferChange":"1231",
                "type":"123",
                "time":31231312312313,
                "state":"1"
            }
        ]
    },
    "params":[

    ]
}
```

#### 4. 杠杆账户借贷列表

请求路径：{base-endpoint}/lever/assetBorrowing

请求方式： POST

请求参数说明：

| 字段       | 类型   | 是否必填 | 说明                                                       |
| ---------- | ------ | -------- | ---------------------------------------------------------- |
| page       | 整数型 | 否       | 分页页数，默认1                                            |
| count      | 整数型 | 否       | 分页每页记录数，默认10                                     |
| coinIdLike | 字符串 | 否       | 币种模糊查询字符，不传默认查询当前柜台全部杠杆配置币种情况 |

返回结果说明：

| 字段     | 类型 | 是否必填 | 说明           |
| -------- | ---- | -------- | -------------- |
| pageInfo | 对象 | 是       | 返回的分页信息 |
| records  | 数组 | 是       | 返回的记录数组 |

**pageInfo**

| 字段        | 类型   | 是否必填 | 说明       |
| ----------- | ------ | -------- | ---------- |
| page        | 数字型 | 是       | 分页当前页 |
| count       | 数字型 | 是       | 每页记录数 |
| pageTotal   | 数字型 | 是       | 总页数     |
| recordTotal | 数字型 | 是       | 总记录数   |

**records数组中元素参数**

| 字段              | 类型 | 是否必填 | 说明 |
| ----------------- | ---- | -------- | ---- |
| coinId | 字符串 | 是 | 币种ID |
| totalAmount | 字符串 | 是 | 总资产 |
| borrowedAmount | 字符串 | 是 | 已借贷资产 |

响应示例：
```json
{
    "code":"0",
    "msg":"success",
    "info":{
        "pageInfo":{
            "page":1,
            "count":10,
            "pageTotal":1,
            "recordTotal":2
        },
        "records":[
            {
                "coinId":"BTC",
                "totalAmount":"123",
                "borrowedAmount":"0"
            },
            {
                "coinId":"BTC",
                "totalAmount":"123",
                "borrowedAmount":"0"
            }
        ]
    },
    "params":[

    ]
}
```

### [杠杆交易认证接口]

#### 1. 杠杆下单 （**需要交易权限**）

请求路径：{base-endpoint}/lever/placeOrder

请求方式：POST

请求参数说明：

| 字段      | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| --------- | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol    | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |
| type      | 交易类型 (limit or market)    | 是               | limit=限价，market=市价                    | String |
| side      | 交易方向 (buy or sell)        | 是               | buy=买入，sell=卖出                        | String |
| price     | 价格                          | 是               | type为市价(market)交易时，固定值 "-1"（**该字段类型为String**）      | String |
| quantity  | 数量                          | 是               | 交易数量（**该字段类型为String**），当type=market，side=buy时，该字段为计价货币数量，其余均为token,并且数量必须大于等于token的最小交易量                                   | String |
| timestamp | 时间戳                        | 是               |                                            | String |

返回结果说明:

| 字段    | 说明           | 备注                        | 类型   |
| ------- | -------------- | --------------------------- | ------ |
| orderId | 返回平台订单号 | 供撤单查询使用 不支持 msgNo | String |
| symbol  | 返回币对符号   |                             | String |

响应示例：

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

#### 2. 杠杆订单撤销（**需要交易权限**）

请求路径：{base-endpoint}/lever/cancelOrder

请求方式：POST

请求参数说明:

| 字段    | 说明     | 必填(是/否/可选) | 备注       | 类型   |
| ------- | -------- | ---------------- | ---------- | ------ |
| orderId | 订单号   | 是               | 由平台返回 | String |
| symbol  | 币种类型 | 是               |            | String |

#### 3. 查询未成交杠杆订单列表

请求路径：{base-endpoint}/lever/openOrders

请求方式：POST

请求参数说明:

| 字段   | 说明             | 必填(是/否/可选) | 备注 | 类型    |
| ------ | ---------------- | ---------------- | ---- | ------- |
| symbol |                  | 是               |      | String  |
| page   | 页码             | 否               |      | Integer |
| count  | 每一页的显示条数 | 否               |      | Integer |

返回结果说明:

| 字段 | 说明     | 备注 | 类型 |
| ---- | -------- | ---- | ---- |
| num  | 总条数   |      | Long |
| list | 订单列表 |      | List |

list说明：

| 字段       | 说明         | 备注                           | 类型    |
| ---------- | ------------ | ------------------------------ | ------- |
| orderId    | 订单id       |                                | String  |
| symbol     | 市场类型     |                                | String  |
| price      | 挂单价格     |                                | decimal |
| tradedNum  | 已成交数量   |                                | Decimal |
| quantity   | 挂单数量     |                                | Decimal |
| avgPrice   | 平均成交价格 |                                | Decimal |
| status     | 订单状态     | send，pending，success，cancel | String  |
| type       | 订单类型     | market，limit                  | String  |
| side       | 订单方向     | buy，sell                      | String  |
| createTime | 挂单时间     |                                | Date    |
| tradeTotal | 委托总额     |                                | Decimal |

响应示例：

```
"data":{
    "num":"10",
    "list":[{
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

#### 4. 查询历史订单列表

请求路径：{base-endpoint}/lever/orderList

请求方式：POST

请求参数说明:

| 字段       | 说明                                                  | 必填(是/否/可选) | 备注 | 类型    |
| ---------- | ----------------------------------------------------- | ---------------- | ---- | ------- |
| side       | 订单买卖类型（buy，sell）                             | 是               |      | String  |
| symbol     |                                                       | 是               |      | String  |
| status     | 订单类型（traded=历史单据）                           | 是               |      | String  |
| queryRange | 查询订单的范围（thisweek=本周，thisweekago=本周以前） | 是               |      | String  |
| page       | 页码                                                  | 否               |      | Integer |
| count      | 每一页的显示条数                                      | 否               |      | Integer |

返回结果说明:

| 字段 | 说明     | 备注 | 类型 |
| ---- | -------- | ---- | ---- |
| num  | 总条数   |      | Long |
| list | 订单列表 |      | List |

list说明：

| 字段       | 说明         | 备注                           | 类型    |
| ---------- | ------------ | ------------------------------ | ------- |
| orderId    | 订单id       |                                | String  |
| symbol     |              |                                | String  |
| price      | 挂单价格     |                                | Decimal |
| tradedNum  | 已成交数量   |                                | Decimal |
| quantity   | 挂单数量     |                                | Decimal |
| avgPrice   | 平均成交价格 |                                | Decimal |
| status     | 订单状态     | send，pending，success，cancel | String  |
| type       | 订单类型     | market，limit                  | String  |
| side       | 订单方向     | buy，sell                      | String  |
| createTime | 挂单时间     |                                | Date    |
| tradeTotal | 委托总额     |                                | Decimal |

响应示例：

```
"data":{
    "num":"10",
    "list":[{
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

#### 5. 订单交易明细

请求路径：{base-endpoint}/lever/orderDetail

请求方式：POST

请求参数说明:

| 字段    | 说明     | 必填(是/否/可选) | 备注                  | 类型   |
| ------- | -------- | ---------------- | --------------------- | ------ |
| orderId | 订单id   | 是               |                        | String |
| symbol  |          | 是               |                       | String |
| page    | 页码     | 可选             | 不选默认1             | String |
| count   | 显示条数 | 可选             | 不选默认10            | String |

返回结果说明:

| 字段 | 说明         | 备注 | 类型 |
| ---- | ------------ | ---- | ---- |
| num  | 总条数       |      | Long |
| list | 交易明细列表 |      | List |

返回结果说明 (List):

| 字段          | 说明             | 备注                   | 类型   |
| ------------- | ---------------- | ---------------------- | ------ |
| orderId       | 订单号           |                        | String |
| orderSign     | 当前订单当前状态 | 当前make还是taker 成交 | String |
| getCount      | 获得数量         |                        | String |
| getCountUnit  | 获得数量单位     | 币种                   | String |
| loseCount     | 失去数量         |                        | String |
| loseCountUnit | 失去数量         | 币种                   | String |
| price         | 成交价格         |                        | String |
| priceUnit     | 成交价格单位     | 币种                   | String |
| fee           | 手续费           | 手续费                 | String |
| feeUnit       | 手续费单位       | 币种                   | String |
| time          | 时间             |                        | Long   |
| fsymbol       | 交易币对         | BTC-USDT               | String |
| side          | 方向             | 买 or 卖               | String |

响应示例：
	
	{
	"data":{
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

#### 6. 单个订单查询

请求路径：{base-endpoint}/lever/singleOrder

请求方式：POST

请求参数说明:

| 字段    | 说明         | 必填(是/否/可选) | 备注 | 类型   |
| ------- | ------------ | ---------------- | ---- | ------ |
| orderId | 订单唯一编号 | 是               |      | String |
| symbol  |              | 是               |      | String |

返回结果说明：

| 字段       | 说明         | 备注                           | 类型    |
| ---------- | ------------ | ------------------------------ | ------- |
| orderId    | 订单id       |                                | String  |
| symbol     |              |                                | String  |
| price      | 挂单价格     |                                | Decimal |
| tradedNum  | 已成交数量   |                                | Decimal |
| quantity   | 挂单数量     |                                | Decimal |
| avgPrice   | 平均成交价格 |                                | Decimal |
| status     | 订单状态     | send，pending，success，cancel | String  |
| type       | 订单类型     | market，limit                  | String  |
| side       | 订单方向     | buy，sell                      | String  |
| createTime | 挂单时间     |                                | Date    |
| tradeTotal | 委托总额     |                                | Decimal |

响应示例：

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

#### 7. 查询我的成交记录

请求路径：{base-endpoint}/lever/myTrades

请求方式：POST

请求参数说明:

| 字段      | 说明             | 必填(是/否/可选) | 备注 | 类型    |
| --------- | ---------------- | ---------------- | ---- | ------- |
| symbol    |                  | 是               |      | String  |
| startTime | 指定成交起始时间 | 是               | 毫秒 | Long    |
| limit     | 显示条数         | 是               |      | Integer |

返回结果说明:

| 字段      | 说明                       | 备注 | 类型       |
| --------- | -------------------------- | ---- | ---------- |
| id        |                            |      | Long       |
| price     | 成交价格                   |      | BigDecimal |
| amount    | 成交量                     |      | BigDecimal |
| side      | 成交类型（buy or sell）    |      | String     |
| direction | 成交角色（taker or maker） |      | String     |
| time      | 成交时间                   |      | Date       |

响应示例：

```
"data":[
			"id":"12300993210",
			"price":"100.01",
			"amount":"0.1",
			"side":"buy",
			"direction":"taker",
			"time": 1557047375000
			},
			...
		],
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

### [合约普通接口] (已废弃)

#### 1. 订单薄

请求路径：{base-endpoint}/contract/orderBook

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |

返回结果说明:

| 字段 | 说明               | 备注                                    | 类型   |
| ---- | ------------------ | --------------------------------------- | ------ |
| b    | 买方向订单薄(bids) | （以:分割，第一个为价格，第二个为数量） | String |
| s    | 卖方向订单薄(asks) | （以:分割，第一个为价格，第二个为数量） | String |
| type | 类型               |                                         |        |

响应示例：

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

#### 2. 行情

请求路径：{base-endpoint}/contract/ticker

请求方式：GET

请求参数说明：

| 字段   | 说明                          | 必填(是/否/可选) | 备注                                       | 类型   |
| ------ | ----------------------------- | ---------------- | ------------------------------------------ | ------ |
| symbol | 币对符号，格式是 token-market | 是               | 前面交易币种 后面是交易市场     (BTC-USDT) | String |

返回结果说明:

| 字段         | 说明                             | 备注 | 类型   |
| ------------ | -------------------------------- | ---- | ------ |
| symbol       | 合约符号                         |      | String |
| type         | 类型                             |      | String |
| lastPrice    | 最新成交价                       |      | String |
| high         | 24小时最高                       |      | String |
| low          | 24小时最低                       |      | String |
| volume       | 24小时成交量                     |      | String |
| change       | 需要*100才是百分数               |      | String |
| openValue    | 未平仓数量                       |      | String |
| fundRate0    | 下次合约费率交换值               |      | String |
| fundTime0    | 下次费率交换的时间(时间戳到毫秒) |      | String |
| adlRanker    | ADL排位区间                      |      | String |
| ver          | 版本号                           |      | String |
| openInterest |                                  |      | String |
| turnover     |                                  |      | String |

响应示例：

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

### [合约认证接口] （已废弃）

#### 1. 合约下单

请求路径：{base-endpoint}/contract/order/create

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
| timeInForce      | 订单时效类型（默认GTC）         | 是               | 'GTC'（一直有效直到取消），'FOK'（全部成交或取消）， 'IOC'（立即成交或取消，可以是部分成交） | String |
| leverage         | 杠杆值                          | 否               |                                                              | String |
| triggerPrice     | 触发价格                        | 否               | trigger单需填写（用于触发单）                                | String |
| benchmarkPrice   | 触发订单字段。                  | 否               | 下单时候的价格，用于判断是价格是向上还是向下穿越触发         | String |
| triggerPriceType | 触发价格的类型                  | 否               | mark（mark price类型）/index（正常的指数价格）/last（最新成交价） | String |

返回结果说明:

| 字段    | 说明           | 备注                        | 类型   |
| ------- | -------------- | --------------------------- | ------ |
| orderId | 返回平台订单号 | 供撤单查询使用 不支持 msgNo | String |

响应示例：

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

#### 2. 合约取消订单

请求路径：{base-endpoint}/contract/order/cancel

请求方式：POST

请求参数说明：

| 字段    | 说明   | 必填(是/否/可选) | 备注 | 类型   |
| ------- | ------ | ---------------- | ---- | ------ |
| orderId | 订单号 | 是               |      | String |

返回参数说明：code = "0"为成功

#### 3. 修改杠杆

请求路径：{base-endpoint}/contract/leverage/update

请求方式：POST

请求参数说明：

| 字段     | 说明                 | 必填(是/否/可选) | 备注 | 类型   |
| -------- | -------------------- | ---------------- | ---- | ------ |
| symbol   | 合约符号             | 是               |      | String |
| leverage | 请求修改的合约杠杆值 | 是               |      | String |

返回参数说明：code="0"为成功

#### 4. 仓位信息

请求路径：{base-endpoint}/contract/position

请求方式：POST

请求参数说明：

| 字段   | 说明     | 必填(是/否/可选) | 备注 | 类型   |
| ------ | -------- | ---------------- | ---- | ------ |
| symbol | 合约符号 | 是               |      | String |

返回参数说明：

| 字段             | 说明             | 备注 | 类型   |
| ---------------- | ---------------- | ---- | ------ |
| positionId       | 仓位ID           |      | String |
| symbol           | 合约符号         |      | String |
| amount           | 仓位数量         |      | String |
| margin           | 仓位保证金       |      | String |
| positionValue    | 仓位价值         |      | String |
| leverage         | 杠杆             |      | String |
| status           | 仓位状态         |      | String |
| openPositionTime | 开仓时间的时间戳 |      | String |
| flatPositionTime | 平仓时间的时间戳 |      | String |
| realProfit       | 已实现盈亏       |      | String |
| liquidation      | 强平价           |      | String |
| side             | 仓位方向         |      | String |
| frozen           | 仓位冻结金       |      | String |

#### 5. 调整保证金

请求路径：{base-endpoint}/contract/margin/update

请求方式：POST

请求参数说明：

| 字段         | 说明                         | 必填(是/否/可选) | 备注 | 类型   |
| ------------ | ---------------------------- | ---------------- | ---- | ------ |
| symbol       | 合约符号                     | 是               |      | String |
| changeAmount | 变更数量（带正负号表示方向） | 是               |      | String |

返回参数说明：code="0"为成功

#### 6. 合约资产查询

请求路径：{base-endpoint}/contract/asset/info

请求方式：POST

请求参数说明：

| 字段       | 说明                                           | 必填(是/否/可选) | 备注 | 类型   |
| ---------- | ---------------------------------------------- | ---------------- | ---- | ------ |
| page       | 分页当前页                                     | 是               |      | int    |
| count      | 分页每页记录数                                 | 是               |      | int    |
| coinIdLike | 币种类型，前端已经要求将模糊查询改成精确查询。 | 否               |      | String |

返回参数说明：

| 字段     | 说明           | 备注 | 类型 |
| -------- | -------------- | ---- | ---- |
| pageInfo | 返回的分页信息 |      | 对象 |
| records  | 返回的记录数组 |      | 数组 |

pageInfo:通用分页参数（见orderList）

records数组中的元素参数：

| 字段     | 说明         | 备注 | 类型   |
| -------- | ------------ | ---- | ------ |
| btcValue | BTC估值      |      | String |
| coinId   | 币种类型     |      | String |
| count    | 资产数量     |      | String |
| frozen   | 冻结资产数量 |      | String |

#### 7. 查询用户私有合约信息 

请求路径：{base-endpoint}/contract/info

请求方式：POST

请求参数说明：

| 字段   | 说明     | 必填(是/否/可选) | 备注 | 类型   |
| ------ | -------- | ---------------- | ---- | ------ |
| symbol | 合约符号 | 是               |      | String |

返回参数说明：

| 字段      | 说明               | 备注 | 类型   |
| --------- | ------------------ | ---- | ------ |
| symbol    | 合约符号           |      | String |
| leverage  | 杠杆值的字符串形式 |      | String |
| fundRate0 | 资金费用           |      | String |
| riskLimit | 风险限额           |      | String |

#### 8. 查询用户合约账户信息 

请求路径：{base-endpoint}/contract/account/info

请求方式：POST

请求参数说明：

| 字段 | 说明     | 必填(是/否/可选) | 备注 | 类型   |
| ---- | -------- | ---------------- | ---- | ------ |
| coin | 币种符号 | 是               |      | String |

返回参数说明：

| 字段                 | 说明               | 备注 | 类型   |
| -------------------- | ------------------ | ---- | ------ |
| coin                 | 币种符号           |      | String |
| totalAmount          | 总资产             |      | String |
| remainMargin         | 剩余保证金         |      | String |
| openPositionMargin   | 仓位保证金         |      | String |
| openOrderMarginTotal | 活动委托订单保证金 |      | String |
| availableAmount      | 可用数量           |      | String |

#### 9. 订单列表查询（活动委托和历史委托）

请求路径：{base-endpoint}/contract/orders

请求方式：POST

请求参数说明：

| 字段   | 说明                        | 必填(是/否/可选) | 备注   | 类型    |
| ------ | --------------------------- | ---------------- | ------ | ------- |
| symbol | 合约符号                    | 是               |        | String  |
| type   | 订单类型（open or history） | 是               |        | String  |
| page   |                             | 否               | 默认1  | Integer |
| count  |                             | 否               | 默认10 | Integer |

返回参数说明:

| 字段     | 说明     | 备注 | 类型 |
| -------- | -------- | ---- | ---- |
| pageInfo | 分页信息 |      | 对象 |
| records  | 记录数组 |      | 数组 |

pageInfo：

| 字段        | 说明         | 备注 | 类型 |
| ----------- | ------------ | ---- | ---- |
| page        | 分页当前页数 |      | int  |
| count       | 每页记录数   |      | int  |
| pageTotal   | 总页数       |      | int  |
| recordTotal | 总记录数     |      | int  |

records数组中的元素参数：

| 字段       | 说明              | 备注                                      | 类型   |
| ---------- | ----------------- | ----------------------------------------- | ------ |
| orderId    | 订单ID            |                                           | String |
| symbol     | 合约符号          |                                           | String |
| type       | 下单类型          | limit or market                           | String |
| side       | 下单方向          | buy or sell                               | String |
| price      | 价格，market单为0 |                                           | String |
| amountReal | 真实数量          |                                           | String |
| amountFill | 成交数量          |                                           | String |
| status     | 订单状态          | open、cancel、filled、rejected、untrigger | String |
| avgPrice   | 平均成交价        |                                           | String |
| time       | 下单的时间戳      |                                           | Long   |

#### 10. 查询用户合约的交易记录 

请求路径：{base-endpoint}/contract/trades

请求方式：POST

请求参数说明：

| 字段   | 说明     | 必填(是/否/可选) | 备注       | 类型    |
| ------ | -------- | ---------------- | ---------- | ------- |
| symbol | 合约符号 | 是               |            | String  |
| page   |          | 否               | default=1  | Integer |
| count  |          | 否               | default=10 | Integer |

返回参数说明：

| 字段     | 说明           | 备注 | 类型 |
| -------- | -------------- | ---- | ---- |
| pageInfo | 返回的分页信息 |      | 对象 |
| records  | 返回的记录数组 |      | 数组 |

pageInfo:通用分页参数（见orderList）

records返回参数说明：

| 字段    | 说明                    | 备注 | 类型    |
| ------- | ----------------------- | ---- | ------- |
| symbol  | 合约符号                |      | String  |
| orderId | 订单id                  |      | String  |
| isTaker | 是否为taker             |      | Boolean |
| side    | 订单方向（buy or sell） |      | String  |
| fee     | 手续费                  |      | String  |
| price   | 成交价格                |      | String  |
| amount  | 成交数量                |      | String  |
| time    | 成交时间                |      | String  |
| version | 版本号                  |      | Long    |

#### 11. 查询用户合约单个订单接口

请求路径：{base-endpoint}/contract/queryOrder

请求方式：POST

请求参数说明：

| 字段    | 说明 | 必填(是/否/可选) | 备注       | 类型   |
| ------- | ---- | ---------------- | ---------- | ------ |
| orderId |      | 否               | 内部订单号 | String |
| msgNo   |      | 否               | 外部订单号 | String |

返回参数说明：

| 字段       | 说明                    | 备注                    | 类型   |
| ---------- | ----------------------- | ----------------------- | ------ |
| symbol     | 合约符号                |                         | String |
| orderId    | 订单id                  |                         | String |
| type       | 订单类型                |                         | String |
| status     | 订单状态                |                         | String |
| side       | 订单方向（buy or sell） |                         | String |
| avgPrice   | 平均成交价              |                         | String |
| price      | 成交价格                |                         | String |
| amountFill | 成交数量                |                         | String |
| amountReal | 委托数量                |                         | String |
| time       | 成交时间                |                         | Long   |
| msgNo      | 外部订单号              | 通过API发送保存的订单号 | String |
| version    | 版本号                  |                         | Long   |

### [C2C接口]
**1、获取订单列表**

- 请求类型为application/json

**请求URL：** 

- ` http://{base-endpoint}/c2c/api/order/getlist`

**请求方式：**

- GET 

**请求参数** 

| 参数名 | 必选 | 类型   | 说明   |
| :----- | :--- | :----- | ------ |
|memberId|是|String|会员id|
|pageNum|是|String|当前页|
|pageSize|是|String|页面总数|
|isAppeal|是|String|是否申诉|
|status|是|String|订单状态(1:处理中2:待付款3:待收款4:已完成5:已取消)|

 **请求示例**

```json
{
  "memberId": "1230",
  "isAppeal": "0",
  "status": "1,2,3",
  "pageNum": "1",
  "pageSize": "10"
}
```

**返回参数** 

| 参数名 | 必选 | 类型   | 说明                            |
| :----- | :--- | :----- | ------------------------------- |
|amount|是|String|数量|
|serviceCharge|是|String|手续费|
|coinId|是|String|币种类型|
|name|是|String|对方姓名|
|fphone|是|String|电话|
|isAppeal|是|String|是否申诉|
|orderNum|是|String|订单号|
|price|是|String|单价|
|priceUnit|是|String|价格单位|
|status|是|String|订单状态|
|totalMoney|是|String|总额|
|tradeType|是|String|交易类型|
|createdTime|是|String|创建时间|
|dealTime|是|String|处理时间|
**返回示例**

```json
{
  "amount": "10",
  "serviceCharge": "0.1",
  "coinId": "string",
  "createdTime": "2019-06-24T07:15:12.319Z",
  "dealTime": "2019-06-24T07:15:12.319Z",
  "name": "string",
  "fphone": "string",
  "isAppeal": true,
  "orderNum": "string",
  "price": "0",
  "priceUnit": "string",
  "status": 0,
  "totalMoney": "0",
  "tradeType": 0
}
```

**2、根据orderNum获取订单**

- 请求类型为application/json

**请求URL：** 

- ` http://{base-endpoint}/c2c/api/order/getorderdetail`

**请求方式：**

- GET 

**请求参数** 

| 参数名 | 必选 | 类型   | 说明   |
| :----- | :--- | :----- | ------ |
|memberId|是|String|会员id|
|orderNum|是|String|原订单号|

 **请求示例**

```json
{
  "memberId": "1230",
  "orderNum": "string"
}
```

**返回参数** 

| 参数名 | 必选 | 类型   | 说明                            |
| :----- | :--- | :----- | ------------------------------- |
|advertisementId|是|String|广告id|
|amount|是|String|数量|
|serviceCharge|是|String|手续费|
|coinId|是|String|币种类型|
|name|是|String|对方姓名|
|fphone|是|String|对方手机|
|isAppeal|是|String|是否申诉|
|orderNum|是|String|订单号|
|payType|是|String|支付类型(1银行卡2支付宝3微信)|
|price|是|String|单价|
|priceUnit|是|String|价格单位|
|status|是|String|订单状态|
|totalMoney|是|String|总额|
|tradeType|是|String|买卖类型(1:买2:卖)|
|userId|是|String|用户id|
|nickName|是|String|付款昵称|
|enclosure1|是|String|附件1|
|enclosure2|是|String|附件2|
|enclosure3|是|String|附件3|
|enclosure1url|是|String|附件1访问地址|
|enclosure2url|是|String|附件2访问地址|
|enclosure3url|是|String|附件3访问地址|
|advertisementUserId|是|String|广告方id|
**返回示例**

```json
{
  "advertisementId": "string",
  "amount": "10",
  "serviceCharge": "0.1",
  "coinId": "string",
  "createdTime": "2019-06-24T07:23:48.708Z",
  "name": "string",
  "fphone": "string",
  "isAppeal": true,
  "orderNum": "string",
  "payType": "string",
  "price": "1",
  "priceUnit": "string",
  "status": 0,
  "totalMoney": "10",
  "tradeType": 0,
  "userId": "string",
  "nickName": "string",
  "enclosure1": "string",
  "enclosure2": "string",
  "enclosure3": "string",
  "enclosure1url": "string",
  "enclosure2url": "string",
  "enclosure3url": "string",
  "advertisementUserId": "string",
  "orderPaytypeSnapshot": [{
            "id": null,
            "orderNum": null,
            "userId": null,
            "name": "姓名",
            "idNumber": "身份证号",
            "account": "银行卡(支付宝，微信)账户",
            "branchOrNickname": ”支行或者昵称“,
            "wayKey": “二维码地址”,
            "url": “访问地址”,
            "type": "支付类型",
            "status": null,
            "createdTime": null
        }]
}
```
**3、确认付款**

- 请求类型为application/json

**请求URL：** 

- ` http://{base-endpoint}/c2c/order/payment`

**请求方式：**

- POST 

**请求参数** 

| 参数名 | 必选 | 类型   | 说明   |
| :----- | :--- | :----- | ------ |
|orderNum|是|String|订单号|
|type|是|String|支付类型(1银行卡2支付宝3微信)|
|nickName|否|String|昵称|
|enclosure1|否|String|附件1|
|enclosure2|否|String|附件2|
|enclosure3|否|String|附件3|

 **请求示例**

```json
{
  "orderNum": "string",
  "type": "string",
  "nickName": "string",
  "enclosure1": "string",
  "enclosure2": "string",
  "enclosure3": "string"
}
```
**返回参数** 

| 参数名 | 必选 | 类型   | 说明                            |
| :----- | :--- | :----- | ------------------------------- |
|info|是|String|订单编号|

**返回示例**

```json
{
  "code": "0",
  "info": "123456",
  "msg": "success",
  "params": []
}
```

**4、新增订单**

- 请求类型为application/json

**请求URL：** 

- ` http://{base-endpoint}/c2c/api/order/savebymatchcondition`

**请求方式：**

- POST 

**请求参数** 

| 参数名 | 必选 | 类型   | 说明   |
| :----- | :--- | :----- | ------ |
|coinId|是|String|购买币种(USDT、BTC)|
|totalMoney|是|String|交易金额|
|payType|是|String|支付方式(1银行卡2支付宝3微信)|
|memberId|是|String|会员id|
|callBack|是|String|交易成功后回调接口地址(请求方式为post请求，并且会带订单编号orderNum和会员idmemberId作为参数)|

 **请求示例**

```json
{
  "coinId": "USDT",
  "totalMoney": "10000",
  "payType": "1",
  "memberId": "123",
  "phone": "12345678",
  "callBack": "https://www.baidu.com"
}
```
**返回参数** 

| 参数名 | 必选 | 类型   | 说明                            |
| :----- | :--- | :----- | ------------------------------- |
|info|是|String|订单编号|

**返回示例**

```json
{
  "code": "0",
  "data": "12345",
  "msg": "success",
  "params": []
}
```

**5、保存kyc**

- 请求类型为application/json

**请求URL：** 

- ` http://{base-endpoint}/saveKyc`

**请求方式：**

- POST 

**请求参数** 

| 参数名 | 必选 | 类型   | 说明   |
| :----- | :--- | :----- | ------ |
|nationality|是|String|国家（alpha3Code）|
|papersType|是|String|证件类型：ID/TRANSPORT(身份证、护照)|
|papersOne|是|String|证件正面（需base64编码），图片本身字节流大小需小于2M|
|papersTwo|是|String|证件反面（需base64编码），图片本身字节流大小需小于2M|
|papersThird|是|String|手持证件照（需base64编码），图片本身字节流大小需小于2M|
|papersId|是|String|证件id（身份证号或护照号）|
|gender|是|String|性别：0：女  1：男|
|birthday|是|String|生日日期: 格式2019-08-01|
|name|是|String|名字(身份证真实名字)|
|memberId|是|String|第三方唯一用户ID|
|phone|是|String|第三方用户手机号|
|areaCode|是|String|绑定国家区号(和手机绑定的)|

 **请求示例**

```json
{
  "nationality": "CHN",
  "papersType": "ID",
  "papersOne": "string",
  "papersTwo": "string",
  "papersThird": "string",
  "papersId": "1111",
  "gender": "1",
  "birthday": "2019-08-01",
  "name": "张三",
  "memberId": "11111",
  "phone": "string",
  "areaCode": "string"
}
```
**返回参数** 

| 参数名 | 必选 | 类型   | 说明                            |
| :----- | :--- | :----- | ------------------------------- |
|memberId|是|String|openApi third userId|

**返回示例**

```json
{
  "code": "0",
  "data": {"memberId":"123456"},
  "msg": "success",
  "params": [
  ]
}
```
**6、获取相关国家信息**

**请求URL：** 

- ` http://{base-endpoint}/countryInfo`

**请求方式：**

- GET

**请求参数** 
无

 **请求示例**
无

**返回参数** 

| 参数名 | 必选 | 类型   | 说明                            |
| :----- | :--- | :----- | ------------------------------- |
|alpha3Code|是|String|国家码|
|areaCode|是|String|国家区号（和手机号连用）|

**返回示例**

```json
{
  "code": "0",
  "data": [{"alpha3Code":"CHN","areaCode":"86"},...],
  "msg": "success",
  "params": [
  ]
}
```

## [应答码对照表]

| code  | msg                    | 说明             | 备注     |
| ----- | ---------------------- | ---------------- | -------- |
| 0     | success                | 成功             | 请求成功 |
| 9000  | missing parameter      | 缺少参数         |          |
| 9001  | version not matched    | 版本号不对       |          |
| 9002  | verifySignature failed | 验证签名失败     |          |
| 9004  | access denied          | 请求不允许       |          |
| 9005  | key expired            | apiKey过期       |          |
| 9006  | no server              | 无服务           |          |
| 9999  | system error           | 系统异常         |          |
|       |                        |                  |          |
|       |                        |                  |          |
| 20043 |                        | 挂单价格精度错误 |          |
| 20044 |                        | 挂单数量精度错误 |          |
|       |                        |                  |          |
|       |                        |                  |          |
|       |                        |                  |          |
|       |                        |                  |          |

[base-endpoint]: [**https://global-openapi.bithumb.pro/openapi/v1**]
