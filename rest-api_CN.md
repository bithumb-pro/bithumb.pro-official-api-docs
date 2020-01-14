目录

[TOC]

## [REST API简介]

- rest api的基本请求路径base-endpoint：**https://global-openapi.bithumb.pro/openapi/v1**
- 所有api的响应都是JSON格式
- GET方法的api, 参数必须在query string中发送
- POST方法的api，参数必须在request body中发送(headers里需添加content-type:application/json)
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
```json
{
"num":5,  //数据总数量
"list":[{"":""}...]
}
```

## [数据签名] (重要)

- 用户计算签名的基于哈希的协议，此处使用 HmacSHA256(签名参数, secretKey(用户申请页面获取))

  String signature = HmacSHA256.encode(signString, secretKey)

- 待签名字符串(signString)：签名数据由客户端的所有请求参数(不为空)按照key的字典顺序来组成，具体示例如下

- 签名示例:
```json
  {
	"apiKey" : "XXXXXXXXXXXX", 	//(用户申请页面获取)
	"msgNo":"1234567890", //参考[请求参数说明]
	"timestamp":15348923323343,
  }
```
  待签名字符串(signString)：apiKey=XXXXXXX&msgNo=123456789&timestamp=1455323333332

  在获得signString后，通过HmacSHA256 使用secretKey生成signature,并将signature加到请求参数中.

## [请求与应答] (重要)

- 对于POST请求，请求headers里需要有Content-Type:application/json

- 请求参数事例:
```json
  {
  	"timestamp":"1455323333332",
	"apiKey":"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX", //(用户申请页面获取)
	"signature":"1EF13F23D123A3123GXXXXXXXXXXXXXXXXXXXXXXXXX"	//参考[数据签名]
  }
```

- 应答事例:

```json
 {
    "code": "0", //响应码，参考[api响应码]
    "msg": "success",
    "data": ""
 }
```
## [接口说明]

### [普通接口]

#### 1. 系统时间

请求路径：{base-endpoint}/serverTime

请求方式：GET

请求参数说明：无

响应示例：

```json
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

```json
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
```json
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
```

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
```json
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
```

### [现货认证接口]


#### 1. 币币下单 （**需要交易权限**）

请求路径：{base-endpoint}/spot/placeOrder

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
```json
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
```

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
```json
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
```

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

#### 9. 币币订单批量撤销（**需要交易权限**）

请求路径：{base-endpoint}/spot/cancelOrder/batch

请求方式：POST

请求参数说明:

| 字段   | 说明         | 必填(是/否/可选) | 备注                                                         | 类型   |
| ------ | ------------ | ---------------- | ------------------------------------------------------------ | ------ |
| ids    | 订单编号列表 | 否               | 使用","拼接的订单id列表（单个批次最多100单），当ids为空字符串或null，或者没有该字段，则撤销该用户在此币对上的所有订单 | String |
| symbol | 币种类型     | 是               |                                                              | String |

请求示例:

```json
{
"symbol":"ETH-USDT",
"ids":"74740189190115328,74740189190115329,74740189190115330"
}
```

#### 10. 批量下单（**需要交易权限**）

请求路径：{base-endpoint}/spot/placeOrders

请求方式：POST

请求参数说明:

| 字段        | 说明         | 必填(是/否/可选) | 备注                                          | 类型   |
| ----------- | ------------ | ---------------- | --------------------------------------------- | ------ |
| multiParams | 批量下单参数 | 是               | 单个下单参数的JSON数组字符串，0 < 订单数<= 10 | String |

请求示例:

```json
{
"multiParams":"[{\"symbol\":\"ETH-BTC\",\"type\":\"limit\",\"side\":\"buy\",\"price\":\"1.1234\",\"quantity\":\"10\",\"timestamp\":1576229333470,\"msgNo\":\"test1576229333470\"},{\"symbol\":\"ETH-BTC\",\"type\":\"limit\",\"side\":\"buy\",\"price\":\"1.1234\",\"quantity\":\"10\",\"timestamp\":1576229333470,\"msgNo\":\"test1576229333470\"}]
"
}
```

响应示例：

```json
{ 
	"msg":"success",
	"code":"0",
	"data":[
		{
			"data":
				{
					"symbol":"ETH-BTC",
					"orderId":"134556412346773504"
				},
			"code":"0",
			"msg":"success",
			"timestamp":1576229851850
		},
		{
			"data":
				{
					"symbol":"ETH-BTC",
					"orderId":"134556412699095040"
				},
			"code":"0",
			"msg":"success",
			"timestamp":1576229851934
		}
		],
	"timestamp":1576229851934
}
```

#### 

### [杠杆资产认证接口]

#### 1. 杠杆资产列表

请求路径：{base-endpoint}/lever/asset/list

请求方式： POST

请求参数说明：

| 字段       | 类型   | 是否必填 | 说明                                                       |
| ---------- | ------ | -------- | ---------------------------------------------------------- |
| coin | 字符串 | 否       | 不传默认查询当前柜台全部杠杆配置币种情况 |

返回结果说明：

| 字段     | 类型 | 是否必填 | 说明           |
| -------- | ---- | -------- | -------------- |
|  coin | String | 是 | 币种 |
| coinFull | String | 是       | 全称 |
| total | String | 是       | 总资产 |
| available | String | 是 | 可用资产 |
| borrowed | String | 是 | 借入资产 |
| interest | String | 是 | 应付利息 |
| rate | String | 是 | 日利率 |
| maxTransferAvailable | String | 是 | 最大可转出 |
| markPrice | String | 是 | 查询时的mp |
| frozen | String | 是 | 冻结 |
| btcAvailable | String | 是 | 可用资产的BTC价值 |

响应示例：
```json
{
    "code":"0",
    "msg":"success",
    "info":[
        {
            "coin":"USDT",
            "coinFull":"XXX XXX XXX",
            "total":"12311231",
            "available":"312",
            "borrowed":"23.12312",
            "interest":"31.23131",
            "rate":"0.05",
            "maxTransferAvailable":"100.231",
            "frozen":"0",
            "btcAvailable":"0"
        },
        {
            "coin":"BTC",
            "coinFull":"XXX XXX XXX",
            "total":"12311231",
            "available":"312",
            "borrowed":"23.12312",
            "interest":"31.23131",
            "rate":"0.05",
            "maxTransferAvailable":"100.231",
            "frozen":"3.1415",
            "btcAvailable":"12"
        }
    ],
    "params":[

    ]
}
```

#### 2. 杠杆账户总览
请求路径：{base-endpoint}/lever/asset/info

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
| interest | 字符串 | 是 | 应付利息 |
| borrowed | 字符串 | 是 | 借贷资产 |


响应示例：
```json
{
    "code":"0",
    "msg":"success",
    "info":{
        "totalAmount":"123.456",
        "netAmount":"121.123",
        "fullMargin":"32",
        "maintainMargin":"32",
        "leverage":"2.1",
        "cushionRate":"1.3",
        "interest":"123",
        "borrowed":"0"
    },
    "params":[

    ]
}
```

#### 3. 借贷资产

请求路径：{base-endpoint}/lever/asset/borrowed

请求方式： POST

请求参数说明：

返回结果说明：

| 字段     | 类型 | 是否必填 | 说明           |
| -------- | ---- | -------- | -------------- |
| coinId | 字符串 | 是       | 币种ID |
| availableAmount  | 字符串 | 是       | 资产余额 |
| borrowedAmount | 字符串 | 是 | 已借贷资产 |


响应示例：
```json
{
    "code":"0",
    "msg":"success",
    "info":[
        {
            "coinId": "BTC",
            "availableAmount": "0.5000000000000000",
            "borrowedAmount": "0.0000000000000000"
        },
        {
            "coinId": "USDT",
            "availableAmount": "0.0000000000000000",
            "borrowedAmount": "0.0000000000000000"
        }
    ],
    "params":[

    ]
}
```

#### 4. 杠杆账户资产汇总

请求路径：{base-endpoint}/lever/asset/collect

请求方式： POST

请求参数说明：

返回结果说明：

| 字段  | 类型   | 是否必填 | 说明       |
| ----- | ------ | -------- | ---------- |
| total | 字符串 | 是       | 总资产折合 |
| debt  | 字符串 | 是       | 负债折合   |
| net   | 字符串 | 是       | 净资产折合 |


响应示例：
```json
{
    "code":"0",
    "msg":"success",
    "info":{
        "total":"123123",
        "debt":"123123",
        "net":"3321"
    },
    "params":[

    ]
}
```

#### 5. 杠杆资产变更记录

请求路径：{base-endpoint}/lever/asset/change/list

请求方式： POST

请求参数说明：

| 字段  | 类型   | 是否必填 | 说明       |
| ---- | ---- | ---- | ---- |
| type | 整数型 | 否 | 类型 |
| coin | 字符串 | 否 | 币种 |
| timeStart | 整数型 | 否 | 查询时间限制起始时间戳 |
| timeEnd | 整数型 | 否 | 查询时间限制结束时间戳 |
| count | 整数型 | 否 | 每页数 |
| page | 整数型 | 否 | 当前页 |
| label | 字符串 | 是 | 类型，all为查询全部，transaction为查询交易变动 |

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

| 字段  | 类型   | 是否必填 | 说明       |
| ---- | ---- | ---- | ---- |
| time | 整数型 | 是 | 变更时间的毫秒时间戳 |
| type | 整数型 | 是 | 变更类型枚举(TODO: 待完善枚举) |
| coin | 字符串 | 是 | 币种 |
| amount | 字符串 | 是 | 变更数量 |
| interestRemain | 字符串 | 是 | 剩余利息 |
| borrowedRemain | 字符串 | 是 | 剩余借贷 |
| status | 字符串 | 是 | 状态,枚举值Completed,Processing |

响应示例：
```json
{
    "code":"0",
    "msg":"success",
    "info":{
        "pageInfo": {
            "totalCount": 1,
            "totalPage": 10,
            "pageIndex": 1,
            "pageSize": 2
        },
        "records": [
            {
                "time": 1576029158937,
                "type": 1,
                "coin": "BTC",
                "amount": "0.5000000000000000",
                "interestRemain": "0.0006349415030000",
                "borrowedRemain": "2.4498050100000000",
                "status": "Completed"
            },
            {
                "time": 1575963173224,
                "type": 1,
                "coin": "BTC",
                "amount": "1.9498050100000000",
                "interestRemain": "0.0001949805010000",
                "borrowedRemain": "1.9498050100000000",
                "status": "Completed"
            }
        ]
    },
    "params":[
    ]
}
```

### [杠杆交易认证接口]

#### 1. 杠杆下单 （**需要交易权限**）

请求路径：{base-endpoint}/lever/order/create

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

请求路径：{base-endpoint}/lever/order/cancel

请求方式：POST

请求参数说明:

| 字段    | 说明     | 必填(是/否/可选) | 备注       | 类型   |
| ------- | -------- | ---------------- | ---------- | ------ |
| orderId | 订单号   | 是               | 由平台返回 | String |
| symbol  | 币种类型 | 是               | 币对       | String |

#### 3. 查询未成交杠杆订单列表

请求路径：{base-endpoint}/lever/order/list/open

请求方式：POST

请求参数说明:

| 字段   | 说明             | 必填(是/否/可选) | 备注     | 类型    |
| ------ | ---------------- | ---------------- | -------- | ------- |
| symbol | 币对             | 是               |          | String  |
| page   | 页码             | 否               | 缺省值1  | Integer |
| count  | 每一页的显示条数 | 否               | 缺省值10 | Integer |

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

请求路径：{base-endpoint}/lever/order/list

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

#### 5. 查询成交记录

请求路径：{base-endpoint}/lever/trades/list

请求方式：POST

请求参数说明:

| 字段            | 说明       | 必填(是/否/可选) | 备注       | 类型    |
| --------------- | ---------- | ---------------- | ---------- | ------- |
| symbol          | 币对       | 是               |            | String  |
| limit           | 显示条数   | 是               |            | Integer |
| orderId         | 订单ID     | 否               |            | String  |
| page            | 分页页数   | 否               |            | Integer |
| count           | 分页没页数 | 否               |            | Integer |
| exchangeOrderId | 交易流水ID | 否               |            | String  |
| startTime       | 开始时间   | 否               | yyyy-mm-dd | String  |
| endTime         | 结束时间   | 否               | yyyy-mm-dd | String  |

返回结果说明:

| 字段       | 说明                       | 备注 | 类型       |
| ---------- | -------------------------- | ---- | ---------- |
| createTime |                            |      | String     |
| price      | 成交价格                   |      | BigDecimal |
| quantity   | 成交量                     |      | BigDecimal |
| side       | 成交类型（buy or sell）    |      | String     |
| direction  | 成交角色（taker or maker） |      | String     |
| time       | 成交时间                   |      | Date       |
| total      | 总量                       |      | BigDecimal |
| fee        | 手续费                     |      | BigDecimal |
| feeUnit    | 手续费币                   |      | String     |

响应示例：

```
"data":{
            "num":1,
            list:[{
                "createTime":"1574048908593",
                "price":"10000",
                "quantity":"1",
                "total":"10000",
                "fee":"0.001",
                "feeUnit":"BTC",
                "side":"buy",
                "symbol":"BTC-USDT",
                "direction":"maker"
            }]
        },
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

#### 6. 下单可用查询

请求路径：{base-endpoint}/lever/available

请求方式：POST

请求参数说明:

| 字段   | 说明        | 必填(是/否/可选) | 备注 | 类型   |
| ------ | ----------- | ---------------- | ---- | ------ |
| symbol | 币对        | 是               |      | String |
| side   | buy or sell | 是               |      | String |


返回结果说明:

| 字段           | 说明           | 备注 | 类型       |
| -------------- | -------------- | ---- | ---------- |
| available      | 可用余额       |      | BigDecimal |
| availableLever | 可用额度       |      | BigDecimal |
| borrowed       | 当前已借贷     |      | BigDecimal |
| lever          | 当前币杠杆倍数 |      | String     |
| coin           | 当前使用币     |      | String     |


响应示例：

```json
"data":{
        "available": "0.0000000000000000",
        "availableLever": "0",
        "borrowed": "0.0000000000000000",
        "lever": "5",
        "coin": "USDT"
    },
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

#### 7. 批量取消订单

请求路径：{base-endpoint}/lever/order/cancel/batch

请求方式：POST

请求参数说明:

| 字段   | 说明   | 必填(是/否/可选) | 备注                                          | 类型   |
| ------ | ------ | ---------------- | --------------------------------------------- | ------ |
| symbol | 币对   | 是               |                                               | String |
| ids    | 指定id | 否               | 如果需要取消指定id的订单，用,拼接成一条字符串 | String |

响应示例：

```json
"data":{},
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```
#### 8. 签名

请求路径：{base-endpoint}/lever/sign

请求方式：POST

请求参数说明:
无

响应示例：

```json
"data":{},
"code": "0",
"msg": "success",
"timestamp": 1551346473238,
"params": []
}
```

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
| 9004  | access denied          | 请求不允许       |   非ip白名单，无API权限，账户已冻结       |
| 9005  | key expired            | apiKey过期       |          |
| 9006  | no server              | 无服务           |          |
| 9999  | system error           | 系统异常         |          |
|       |                        |                  |          |
|       |                        |                  |          |
| 20003 | asset not enough       | 资产不足                 |          |
| 20012 | cancel faild,order status changed | 该订单不允许撤销，请联系客服 | |
| 20043 | price accuracy is wrong for placing order | 挂单价格精度错误 |          |
| 20044 | quantity accuracy is wrong for placing order| 挂单数量精度错误 |          |
| 20053 | need sign protocol in website|  需要在网页签署协议         |          |
| 20054 | order price out of range| 挂单价格超出范围                 |  参考config接口的percentPrice属性        |
| 20056 | order quantity out of range| 挂单数量超出范围              |          |
|       |                        |                  |          |
