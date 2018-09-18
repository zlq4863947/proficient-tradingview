
# TradingView JS API集成教程(二)：第1部分

![](https://cdn-images-1.medium.com/max/2000/1*c-dfKdC4CexERoS6t33eRA.png)

查看本教程系列的[简介](https://www.jianshu.com/p/a96fabb9f830)（如果您还没有）。设置TradingView图表可能是一个复杂的过程，所以请事先查看免责声明和备注。

> **免责声明：TradingView图表库是Github上的免费私有项目，您必须**[**申请**](https://www.tradingview.com/HTML5-stock-forex-bitcoin-charting-library/)**访问。我认为许可协议禁止我将其分发给您，因此要完全完成本指南，您需要申请下载图表库的权限。**

要在本地运行本教程的这一部分（假设您可以访问图表库）克隆下面提供的repo，然后将图表库文件夹复制到part1文件夹中的`/public/`目录下。运行`npm install`然后`npm start`启动开发服务器。

> **教程资源：**  [https://github.com/jonchurch/tradingview-js-api-tutorial.git](https://github.com/jonchurch/tradingview-js-api-tutorial.git)
> **部署预览：**  [https://tv-tut-part1.glitch.me/](https://tv-tut-part1.glitch.me/)

----------

TradingView允许您在自己的网站上使用自己的图表库，并拥有自己的数据源。

有两种方法可以将您的数据导入TradingView，UDF API和JS API。JS API使您可以最大程度地控制数据，在我看来，它更加灵活。而且它是Javascript！

您可以随意实现数据连接，但实际的实现细节*非常模糊不清*。本教程的目的是向您展示使用您自己的数据源和TradingView图表创建一个基本的静态图表的工作示例。

> **注意：TradingView不会为您提供此数据源，并假设您已实现自己的来源。为方便起见，本教程依赖于CryptoCompare的历史价格API作为数据源。**

本指南以[此处](https://github.com/tradingview/charting-library-examples)提供的React Javascript TradingView示例为基础

### 概述

首次加载图表widget时，它将使用默认交易对的`商品名称`调用JS API方法`resolveSymbol` 。在我们的示例中`Coinbase:BTC/USD`是默认的交易对。您应该将[symbolInfo](https://zlq4863947.gitbooks.io/tradingview/book/Symbology.html)对象通过图表库JSAPI传递给`resolveSymbol`函数的回调`onSymbolResolvedCallback`。

**整个集成由几部分组成：**

-   图表库Widget构造函数 - 获取widget配置对象，传入datafeed，显示默认交易对，用户选项，图表加载/保存选项
-   Datafeed - JS API和后端之间的接口
-   JS API - 图表库显示数据所需的接口
-   History Provider - 提供OHLCV的K线数据
-   Realtime Provider - 提供实时更新或增加最新的K线数据
-   Symbol Store - 提供可用的商品列表

_第1部分介绍图表库Widget构造函数，Datafeed，JS API和History Provider，用于创建有硬编码交易对的静态图表。_

### Widget构造函数

Widget构造选项可配置TradingView图表，并影响图表首次加载时启用的功能，以及用户可以设置的选项。

在这里，我们设置选项，如用户ID，样式设置，语言，要加载的交易对，图表库的公共资源路径，以及传入我们的JS API Datafeed实现。

> Widget构造选项的文档都[可以在这里找到](https://zlq4863947.gitbooks.io/tradingview/book/Widget-Constructor.html)

**这是我们开始的构造函数选项：**

```javascript
const widgetOptions = {  
   debug: false,  
   symbol: 'Coinbase:BTC/USD',  
   datafeed: Datafeed, // our datafeed object  
   interval: '15',  
   container_id: 'tv_chart_container',  
   library_path: '/charting_library/',  
   locale: getLanguageFromURL() || 'en',  
   disabled_features: ['use_localstorage_for_settings'],  
   enabled_features: [],  
   client_id: 'test',  
   user_id: 'public_user_id',  
   fullscreen: false,  
   autosize: true,  
   overrides: {  
    "paneProperties.background": "#131722",  
    "paneProperties.vertGridProperties.color": "#363c4e",  
    "paneProperties.horzGridProperties.color": "#363c4e",  
    "symbolWatermarkProperties.transparency": 90,  
    "scalesProperties.textColor" : "#AAA",  
    "mainSeriesProperties.candleStyle.wickUpColor": '#336854',  
    "mainSeriesProperties.candleStyle.wickDownColor": '#7f323f',  
   }  
  };
```

这会将Widget配置为向我们显示`Coinbase:BTC/USD`15分钟周期的K线数据，并设置一些其他自定义（禁用的功能集，要覆盖的默认设置，要使用的语言等）。

加载后，您不需要更改任何选项，[Widget会公开方法](https://zlq4863947.gitbooks.io/tradingview/book/Widget-Methods.html)可用于动态更改某些设置。（更改商品可以通过我们将在本教程的第3部分中实现的商品搜索来完成）

我通过设置功能集将图表默认为黑色模式`overrides."painProperties.background": “#131722”` 。

### JS API Datafeed集成

现在我们已经配置了Widget并设置了我们喜欢的样式，让我们看看我们如何将图表数据连接到TradingView 图表库的JS API上。

**JS API实际上是您提供给TradingView Widget的对象，它公开了TradingView将调用的函数，并且在大多数情况下，您需要将数据传递给这些函数中的回调函数，以使您的数据与TradingView一起使用。**

例如，我们使用[CryptoCompare的历史图表数据](https://min-api.cryptocompare.com/)，在第2部分中，使用websocket API来获得实时价格更新。

**TradingView将**根据需要**调用您提供的方法**，以使数据填充当前图表，以及必须执行的其他生命周期方法。

**下面是TradingView希望您传递给Widget的整个JS API对象。**有些方法是可选的，[请参阅文档以获取更多信息](https://zlq4863947.gitbooks.io/tradingview/book/JS-Api.html)

```javascript
{  
/* 实时图表的必须方法 */  
onReady: cb => {},

// 只在检索功能开启时才需要searchSymbols方法
searchSymbols:(userInput, exchange, symbolType, onResultReadyCallback) => {},

resolveSymbol: (symbolName, onSymbolResolvedCallback, onResolveErrorCallback) => {},

getBars: (symbolInfo, resolution, from, to, onHistoryCallback, onErrorCallback, firstDataRequest) => {},

subscribeBars: (symbolInfo, resolution, onRealtimeCallback, subscribeUID, onResetCacheNeededCallback) => {},

unsubscribeBars: subscriberUID => {},

/* 可选方法 */

getServerTime: cb => {},

calculateHistoryDepth: (resolution, resolutionBack, intervalBack) => {},

getMarks: (symbolInfo, startDate, endDate, onDataCallback, resolution) => {},

getTimeScaleMarks: (symbolInfo, startDate, endDate, onDataCallback, resolution) => {}  
}
```

**首次加载图表时，JS API流程如下所示：**

**1.**  `onReady`被调用，传递[datafeed配置选项](https://zlq4863947.gitbooks.io/tradingview/book/JS-Api.html#onreadycallback)给`cb`

**2.**  `resolveSymbol`被调用，传递[symbolInfo对象]https://zlq4863947.gitbooks.io/tradingview/book/Symbology.html)给`onSymbolResolvedCallback`

**3.**  `getBars`被调用，传递K线对象数组(时间为以毫秒为单位的UTC时间戳)个体`onHistoryCallback`

让我们看看我们的每个实现

#### onReady

```javascript
const config = {  
  supported_resolutions: ["1", "3", "5", "15", "30", "60", "120",   "240", "D"]  
}

onReady: cb => {  
  console.log('=====onReady running')   
  setTimeout(() => cb(config), 0)  
}
```

`onReady`在图表Widget初始化之后立即调用，我们必须将datafeed配置选项传递给该`onReady` 方法的 `cb`回调函数。图表库希望它以异步方式执行，并建议在延迟为0秒的setTimeout中包装以强制执行此行为。

**现在我们只指定了**[**一个可能的选项**](https://zlq4863947.gitbooks.io/tradingview/book/JS-Api.html#onreadycallback)**，**`supported_resolutions`它告诉图表库我们的数据源支持哪些K线周期。这些将显示给用户，并且可以在稍后的每个交易对的`resolveSymbol` 方法中被覆盖。我们提供的列表转换为：

`1分钟, 3分钟, 15分钟, 30分钟, 1小时, 2小时, 4小时, 1天`

在本教程的后面，我们将为Datafeed配置添加选项，因为我们实现了搜索和实时数据图表。

#### resolveSymbol

配置数据源后，图表库将通过Widget初始化对象中的`symbol`属性调用`resolveSymbol`  。我们只给出一个字符串值，并且必须返回表示相应商品的[symbolInfo对象](https://zlq4863947.gitbooks.io/tradingview/book/Symbology.html)。

```javascript
resolveSymbol: (symbolName, onSymbolResolvedCallback, onResolveErrorCallback) => {  
  var split_data = symbolName.split(/[:/]/)  
    
  var symbol_stub = {  
   name: symbolName,  
   description: '',  
   type: 'crypto',  
   session: '24x7',  
   timezone: 'America/New_York',  
   ticker: symbolName,  
   minmov: 1,  
   pricescale: 100000000,  
   has_intraday: true,  
   intraday_multipliers: ['1', '60'],  
   supported_resolution:  ["1", "3", "5", "15", "30", "60", "120",   "240", "D"],  
   volume_precision: 8,  
   data_status: 'streaming',  
  }

  if (split_data[2].match(/USD|EUR|JPY|AUD|GBP|KRW|CNY/)) {  
   symbol_stub.pricescale = 100  
  }  
    
  setTimeout(function() {  
   **onSymbolResolvedCallback(symbol_stub)**  
  }, 0)  
}
```

您可以在此配置单个商品，设置要显示的小数位数，每个刻度移动多少（对于数字货币它几乎总是1），以及**非常重要的，容易搞错的**的`intraday_multipliers` ！因为数字货币是不间断交易的，所以我们将交易时间设置为`24x7` 。时区应该是这个商品的交易所时区。

symbolInfo的所有文档都在[这里](https://zlq4863947.gitbooks.io/tradingview/book/Symbology.html)，请务必熟悉它。

`intraday_multipliers`和`has_intraday`控制显示低于1天的K线周期。现在我在这里犯了很多错误：TradingView可以为你建立一些K线。
例如，让我们假设我们的历史数据API只能以1分钟的时间周期给我们数据，这意味着如果我们请求过去24小时的数据，我们将获得1440个K线数据，即24小时内的分钟数。

但是如果我们想要显示15分钟的K线呢？
您可以告诉TradingView我们的`intraday_multiplier`是`‘1’`并且只传递1分钟K线。图表库将为您构建15分钟的K线，并将其显示在图表上。

我们正在为小时K线做同样的事情，告诉TradingView我们可以提供60分钟的K线，它应该从我们的60分钟K线建立我们的2小时和4小时的K线。

**Ticker**也非常重要。如果设置，那么图表库将在内部使用Ticker来作为唯一ID（`ticker`值将被发送到`resolveSymbol`而不是`name`字段）。`name`字段将显示给用户。我将`name`和`ticker`都设置了相同的值以使我的工作更轻松，因为我使用的`name`包括识别商品所需的所有信息：交易所，交易对（例如Coinbase:BTC/USD）

**Pricescale**有点有趣，因为不同的交易对可以有不同的小数精度。例如，BTC/USD将其测量为小数点后两位，`pricescale = 100`但是对于TRX/BTC（写入时为0.00000771 BTC），我们将其测量为satoshi的8位小数。因此对于TRX/BTC `pricescale = 100000000`但对于TRX/USD（写作时为0.059432 USD），我们将其设置为6位小数`pricescale = 1000000`。

**了解symbolInfo如何影响您的图表非常重要，因此请查看**[**文档**](https://zlq4863947.gitbooks.io/tradingview/book/Symbology.html)**。**

#### getBars

现在开始进入有趣的部分！
从我们的API源获取图表数据并将其交给TradingView。

```javascript
getBars: function(symbolInfo, resolution, from, to, onHistoryCallback, onErrorCallback, firstDataRequest) {  
  
  historyProvider.getBars(symbolInfo, resolution, from, to, firstDataRequest)  
  .then(bars => {  
   if (bars.length) {  
    onHistoryCallback(bars, {noData: false})
   } else {  
    onHistoryCallback(bars, {noData: true})  
   }  
  }).catch(err => {  
   console.log({err})  
   onErrorCallback(err)  
  })  
}

...

/* historyProvider.js */  
var rp = require('request-promise').defaults({json: true})

    getBars: function(symbolInfo, resolution, from, to, first, limit) {  
  var split_symbol = symbolInfo.name.split(/[:/]/) 
     
  const url = resolution === 'D' ? '/data/histoday' : resolution >= 60 ? '/data/histohour' : '/data/histominute'

   const qs = {  
     e: split_symbol[0], // Coinbase  
     fsym: split_symbol[1], // BTC  
     tsym: split_symbol[2], // USD  
     toTs:  to ? to : '',  
     limit: 2000,   
    }

return rp({  
                url: `${api_root}${url}`,  
                qs,  
            })  
            .then(data => {  
    if (data.Response && data.Response === 'Error') {  
     console.log('CryptoCompare API error:',data.Message)  
     return []  
    }  
    if (data.Data.length) {  
     var bars = data.Data.map(el => {  
      return {  
       time: el.time * 1000, //TradingView requires bar time in ms  
       low: el.low,  
       high: el.high,  
       open: el.open,  
       close: el.close,  
       volume: el.volumefrom   
      }  
     })  
     return bars  
    } else {  
     return []  
    }  
   })  
}
```

**好吧，让我们打破所有代码吧！**

Tradingview调用`getBars`并传递symbolInfo对象，此symbolInfo对象是我们传递给`resolveSymbol`回调传递的，周期（我们需要1分钟K线？60分钟K线？1天？），to和from时间戳，以及是否第一次请求这个商品数据的布尔类型标记。

从那里开始，我们调用的`historyProvider.getBars`是我们编写的代码，用于从[Cryptocompare的历史价格API中](https://min-api.cryptocompare.com/)检索历史性的ohlcv数据。我们必须将一个K线数据数组传递给getBar的 `onHistoryCallback` ，该数组在1分钟K线数据中看起来像这样：

```json
[  
...{  
time: 1528322340000, //bar time must be in milliseconds  
open: 7678.85,  
high: 7702.55,  
low: 7656.98,  
close: 7658.25,  
volume: 0.9834  
},  
...  
]
```

因此，我们的historyProvider文件负责实际向CryptoCompare发出请求以获取适当的数据。要使用CrytoCompare发出请求，我们需要知道商品，商品和我们想要数据的指定交易所。

**因为我们选择将所有相关信息放入商品名称（Coinbase:BTC/USD），所以我们能够从****字符串中****提取这些参数**`symbolInfo.name`**。**

TradingView还传递`resolution`给getBars，它将告知我们从CryptoCompare请求的API端点，分钟，小时或日历史数据端点。

由于CryptoCompare API的限制（我们一次只能获得2000条记录），我们可能会传递一套不完整的TradingView请求的数据。别担心！将再次调用getBars，使用new和from 时间戳，直到获得填充图表可见部分所需的所有数据。

----------

### 万岁静态图表！

我希望这对你有帮助。这个过程一开始让我不知所措，这就是我试图与你分享我的学习的原因，这是一个令人困惑的过程。

你可能会想“好的，但静态图表对我帮助不大”。在本教程系列的[第2部分中](https://medium.com/@jonchurch/tv-part-2-b8c2d4211f90)，我们实现了对图表的实时更新。首先要掌握这里概述的概念，熟悉TradingView的文档非常重要。

> **以下是第1部分的部署预览**：[https://tv-tut-part1.glitch.me/](https://tv-tut-part1.glitch.me/)

我相信很多人仍然感到困惑，或者发现我可能犯过的错误。您可以在此处发表评论，或通过以下电子邮件与我联系👇👇

----------

### 关于译者

我是一名现就职于[日本最大数字货币交易所bitbank](bitbank.cc)的全栈开发人员，住在东京。如果您需要加快将TradingView放入到您的网站或交易网页，我可以付费帮忙。

**您可以联系我：zlq4863947@gmail.com**

#### 您也可以加入tradingview开发交流qq群：313839516

### 关于本篇译文

#### 英文原文地址： [TradingView Charting Library JS API Setup for Crypto: Part 1](https://medium.com/@jonchurch/tradingview-charting-library-js-api-setup-for-crypto-part-1-57e37f5b3d5a)