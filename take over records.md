**说明**

> 查询爆仓接管记录

**接口地址**

`{requestUrl}/lever/takeOverList`


**API地址**
`{base-endpoint}/lever/take-over/list`

**请求方式**

*HTTP POST*

**请求参数**

| 字段      | 类型   | 是否必填 | 说明                                                       |
| --------- | ------ | -------- | ---------------------------------------------------------- |
| page      | 整数型 | 否       | 分页页数，默认1                                            |
| count     | 整数型 | 否       | 分页每页记录数，默认10                                     |
| coin      | 字符串 | 否       | 币种，不传默认查询当前柜台全部杠杆配置币种情况 |
| timeStart | 整数型 | 否       | 查询时间起始限制，default 0                                |
| timeEnd   | 整数型 | 否       | 查询时间结束限制，default now time                         |
| userId    | 字符串 | 否       | 用户ID, default null                                       |
| counterId | 字符串 | 否       | 所在柜台ID，default 当前柜台ID                             |
| priceMax  | 字符串 | 否       | 价格最大限制，default null                                 |
| priceMin  | 字符串 | 否       | 价格最小限制，default 0                                    |
| amountMax | 字符串 | 否       | 数量最大限制，default null                                 |
| amountMin | 字符串 | 否       | 数量最小限制，default 0                                    |


**返回参数**

| 字段     | 类型 | 是否必填 | 说明           |
| -------- | ---- | -------- | -------------- |
|  id   | 对象 | 是 | 记录ID |
| userId | 字符串 | 是       | 返回的分页信息 |
| counterId | 字符串 | 是       | 返回的记录数组 |
| coinId | 字符串 | 是 | 币种 |
| amount | 字符串 | 是 | 数量 |
| price | 字符串 | 是 | 价格 |
| time | 整数型 | 是 | 时间 |

**返回示例**

```json
{
    "code": "0",
    "msg": "成功",
    "info": {
        "total": 2,
        "list": [
            {
                "id": 2,
                "userId": "321",
                "counterId": "2",
                "coinId": "IEN",
                "amount": "134959029",
                "price": "123",
                "time": "1569469899722"
            },
            {
                "id": 1,
                "userId": "123",
                "counterId": "2",
                "coinId": "BYC",
                "amount": "100",
                "price": "200",
                "time": "1569469899500"
            }
        ],
        "pageNum": 1,
        "pageSize": 100,
        "size": 2,
        "startRow": 1,
        "endRow": 2,
        "pages": 1,
        "prePage": 0,
        "nextPage": 0,
        "isFirstPage": true,
        "isLastPage": true,
        "hasPreviousPage": false,
        "hasNextPage": false,
        "navigatePages": 8,
        "navigatepageNums": [
            1
        ],
        "navigateFirstPage": 1,
        "navigateLastPage": 1
    },
    "params": []
}
```

