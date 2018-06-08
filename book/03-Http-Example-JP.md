
# 日本股票图表

本例子将显示日本东证一部股票数据。

* 视频程序链接：https://gitee.com/proficient-tradingvi/lesson03-udf-jpstock

### 1、数据准备

 - 东证一部股票一览
 api: https://hesonogoma.com/stocks/data/tosho-1st-stock-data.json

返回结构：
 
| 项目 | 表示例 | 项目 | 表示例 |
|--------|-----------|--------|-----------|
| 銘柄 | 6553 | 始値 | 2493 |
| 名称 | ソウルドアウト | 高値 | 2516 |
| 市場 | 東証マザ | 安値 | 2433 |
| 業種 | サービス | 出来高 | 178600 |
| 日時 | 9/25 15:00 | 売買代金(千円) | 442286 |
| 株価 | 2453 | 時価総額(百万円) | 24047 |
| 前日比 | +10 | 値幅下限 | 1943 |
| 前日比(％) | +0.41 | 値幅上限 | 2943 |
| 前日終値 | 2443 |
 
 - K线数据
api: https://finance.google.com/finance/getprices?q=6553&x=TYO&p=6M&i=86400&f=d,o,c,h,l,v

接口说明

| 参数 | 说明 | 表示例 |
|--------|--------|-----------|
| q | 商品代码 | 日本股票（eg：6553）、日经225（NI225）、美元兑人民币（USDCNY） |
| x | 商品的交易所 | TYO，汇率或特殊商品可不填 |
| p | 时间范围 | 6m：6个月 (d：天、m:月、年：y) |
| i | 时间周期 | 86400：日线(60x60x24) |
| f | 查询项目 | o:open, c:close, l:low, h:high, v:volume |

node模块
https://www.npmjs.com/package/ns-findata

### 2、待实现接口


- Datefeed配置

GET /config

返回值：

```
{
    supports_search: true,
    supports_group_request: false,
    supported_resolutions: ["1", "5", "15", "30", "60", "1D", "1W", "1M"],
    supports_marks: false,
    supports_time: true
}
```

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

返回值参考： https://b.aitrade.ga/books/tradingview/book/JS-Api.html#getbarssymbolinfo-resolution-from-to-onhistorycallback-onerrorcallback-firstdatarequest

- 服务器时间

GET /time

返回值：数字unix时间（秒）