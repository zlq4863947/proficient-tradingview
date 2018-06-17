
# OKCoinf交易所的WebSocket图表

本例子将使用WebSocket显示OKCoin的数据。

## 视频程序链接
https://github.com/zlq4863947/lesson06-ws-tradingview

## 视频链接

- 01

[精通TradingView开发(五) UDF实例之多交易所OKCoin图表-01
](https://v.youku.com/v_show/id_XMzY2ODkwMTA4OA==.html)

- 02

[精通TradingView开发(五) UDF实例之多交易所OKCoin图表-02
](http://v.youku.com/v_show/id_XMzY2OTA5MTg5Mg==.html)

- 03

[精通TradingView开发(五) UDF实例之多交易所OKCoin图表-03
](http://v.youku.com/v_show/id_XMzY2OTIwOTQ5Ng==.html)

- 04

[精通TradingView开发(五) UDF实例之多交易所OKCoin图表-04
](http://v.youku.com/v_show/id_XMzY2OTI2NDA1Mg==.html)

## TradingView请求区间

- 1
http://datafeed.aitrade.ga/history?symbol=HUOBI%3ABTC%2FUSDT&resolution=1&from=1528880202&to=1528966662

2018/06/13 17:56:42
2018/06/14 17:57:42
1天
60*24 = 1440

- 5
http://datafeed.aitrade.ga/history?symbol=HUOBI%3ABTC%2FUSDT&resolution=5&from=1528534652&to=1528966712

2018/06/09 17:57:32
2018/06/14 17:58:32
5天
60*24/5*5 = 1440

- 15
http://datafeed.aitrade.ga/history?symbol=HUOBI%3ABTC%2FUSDT&resolution=15&from=1527670763&to=1528966823

2018/05/30 17:59:23
2018/06/14 18:00:23
15天
60*24/15*15 = 1440

- 1D
http://datafeed.aitrade.ga/history?symbol=HUOBI%3ABTC%2FUSDT&resolution=D&from=1497862502&to=1528966562

2017/06/19 17:55:02
2018/06/14 17:56:02
60*24/60*24*365 = 365