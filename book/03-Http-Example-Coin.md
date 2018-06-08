
# 支持多数字货币交易所的图表

本例子将显示火币与OKCoin的数据。

* 视频程序链接：https://gitee.com/proficient-tradingvi/lesson04-multi-coin-exchange

## 1、数据准备

### 火币

- api文档地址：https://github.com/huobiapi/API_Docs/wiki/REST_api_reference

- 币种信息
api：https://api.huobipro.com/v1/common/symbols

```
{
    "status": "ok",
    "data": [
        {
            "base-currency": "btc",
            "quote-currency": "usdt",
            "price-precision": 2,
            "amount-precision": 4,
            "symbol-partition": "main"
        },
        {
            "base-currency": "bch",
            "quote-currency": "usdt",
            "price-precision": 2,
            "amount-precision": 4,
            "symbol-partition": "main"
        }
        ...
    ]
}
```

data结构：
 
| 项目 | 表示例 | 说明 |
|--------|-----------|-----------|
| base-currency | btc | 基础币种 |
| quote-currency | usdt | 计价币种 |
| price-precision | 2 | 价格精度位数（0为个位） |
| amount-precision | 4 | 数量精度位数（0为个位） |
| symbol-partition | main | 交易区，main主区，innovation创新区，bifurcation分叉区 |
 
 - K线数据
api: https://api.huobipro.com/market/history/kline?period=1day&size=1000&symbol=btcusdt

接口说明

| 参数 | 说明 | 表示例 |
|--------|--------|-----------|
| symbol | 交易对 | btcusdt, bchbtc, rcneth ... |
| period | K线周期 | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |
| size | 获取数量 | 1 ~ 1000 |



### OKCoin

- api文档地址：https://support.okcoin.com/hc/zh-cn/articles/360000697832

- 币种信息
https://www.okcoin.com/marketList
 
| 交易对 | 价格精度 |
|--------|-----------|
| btc_usd | 2 |
| ltc_usd | 2 |
| eth_usd | 2 |
| etc_usd | 2 |
| bch_usd | 2 |
 
 - K线数据
api: https://www.okcoin.com/api/v1/kline.do?symbol=btc_usd&type=1min

接口说明

| 参数 | 说明 | 表示例 |
|--------|--------|-----------|
| symbol | 交易对 | btc_usd：比特币    ltc_usd：莱特币    eth_usd :以太坊     etc_usd :以太经典    bch_usd :比特现金 |
| type | K线周期 | 1min, 3min , 5min, 15min, 30min, 1hour, 2hour, 4hour, 6hour, 12hour, 1day, 3day, 1week |
| size | 获取数量 | 指定获取数据的条数(不填为全部获取) |
| since | 开始时间 | 时间戳，返回该时间戳以后的数据(例如1417536000000) (不填为全部获取) |


```
[
    [
        1526278980000, 时间戳
        "8556.66", 开
        "8556.66", 高
        "8556.66", 低
        "8556.66", 收
        "0"        成交量
    ],
    [
        1526279040000,
        "8556.66",
        "8556.66",
        "8556.66",
        "8556.66",
        "0"
    ],
    ...
]
```

### datafeed数据服务模块
https://www.npmjs.com/package/datafeed-server

## 2、待实现接口

- 商品解析

```
GET /symbols?symbol=<symbol>
```

返回值： JSON包含的对象与SymbolInfo完全一样

- 商品检索

```
GET /search?query=<query>&type=<type>&exchange=<exchange>&limit=<limit>
```

| 参数 | 类型 | 说明 |
|--------|--------|-----------|
| query | string | 用户在商品搜索编辑框中输入的文本 |
| type | string | 商品类型 |
| exchange | string | 交易所名称 |
| limit | integer | 响应最大项目数 |

返回值参考： 
https://b.aitrade.ga/books/tradingview/book/JS-Api.html#searchsymbolsuserinput-exchange-symboltype-onresultreadycallbackhttps://b.aitrade.


- 商品历史查询

```
GET /history?symbol=<ticker_name>&from=<unix_timestamp>&to=<unix_timestamp>&resolution=<resolution>
```

https://b.aitrade.ga/books/tradingview/book/UDF.html#k%E7%BA%BF%E6%9F%B1