
## 图表配置


本节主要讲解图表配置，主要内容包括图表全局样式、绘图区、图表事件、等相关内容。

### 一、图表使用
目前（2018年5月1日），图表库的版本为：
* 稳定版: v 1.12
* 不稳定版: v 1.13

#### 1、引入必须文件

图表库的使用前需要引用3个js文件：
```
<script type="text/javascript" src="charting_library/charting_library.min.js"></script>
<script type="text/javascript" src="datafeeds/udf/dist/polyfills.js"></script>
<script type="text/javascript" src="datafeeds/udf/dist/bundle.js"></script>
```

* charting_library.min.js
	图表库本体js文件（从1.8版本）
* polyfills.js
	ES5浏览器提供ES6特性支持的文件,比如Promise等。
* bundle.js
	UDF经过Rollup模块打包后的问题件

#### 2、使用构造函数

```
TradingView.onready(function() {
    var widget = window.tvWidget = new TradingView.widget({
        // debug: true, // 取消注释可以在控制台中查看的库图表错误和警告
        fullscreen: true,
        symbol: 'AAPL',
        interval: 'D',
        container_id: "tv_chart_container",
        // 注意： URL结尾不可以有斜线
        datafeed: new Datafeeds.UDFCompatibleDatafeed("https://demo_feed.tradingview.com"),
        library_path: "charting_library/",
        locale: getParameterByName('lang') || "en",
        // Regression Trend-related functionality is not implemented yet, so it's hidden for a while
        drawings_access: { type: 'black', tools: [ { name: "Regression Trend" } ] },
        disabled_features: ["use_localstorage_for_settings"],
        enabled_features: ["study_templates"],
        charts_storage_url: 'http://saveload.tradingview.com',
        charts_storage_api_version: "1.1",
        client_id: 'tradingview.com',
        user_id: 'public_user_id'
    });
});

```