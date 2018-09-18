# TradingView JS API集成教程(三)：第2部分-实时图表更新

在本篇中，我们将在图表上实现实时价格更新。请务必阅读简介，以及第1部分第
> **在本篇中，我们将在图表上实现实时价格更新。在此之前请务必阅读**[**简介**](https://www.jianshu.com/p/a96fabb9f830)**，以及**[**第1部分**](https://www.jianshu.com/p/603d671400da)
> 
为了让图表可以实时更新数据，我们将继续改造第1部分的示例。

**此示例将使用CryptoCompare的websocket来获取价格更新。**

> **你可以看到**[**示例**](https://tv-tut-part2.glitch.me/)**部署在**[**glitch**](https://medium.com/@glitch)**上，并查看第2部分的**[**代码**](https://github.com/jonchurch/tradingview-js-api-tutorial/tree/master/part2)

第1部分重点介绍如何设置TradingView图表库Widget，以及设置JS API以从我们自己的数据源获取历史K线数据。

**让我们可以实时更新图表的JS API方法是：**

-   `subscribeBars` - 订阅商品的实时数据
-   `unsubscribeBars` - 取消订阅商品的实时数据

基本上我们需要做的就是更新我们图表上最新的K线，无论是1分钟的K线，还是1天的K线，这个过程几乎是一样的。

我们必须保持图表上最后一条K线的变量记录，用最新价格数据更新它（该周期的开盘价，最高价，最低价，收盘价或成交量是否发生了变化？），如果时间进入了新的周期，则提供一个新的K线数据而不是更新最后一条K线数据。

> **注意：如果你提供1分钟的TradingView，并让它建立5分钟，15分钟等等，那么你实际只需更新1分钟。不用担心，TradingView将在调用**`subscribeBars` **时指定所需的分辨率！**

首先，让我们来看看我们将要使用的这些新的JS API方法。

#### subscribeBars

`resolveSymbol`  假设我们能够成功解析商品，则在之后，图表库将调用此方法。如果您不熟悉`resolveSymbol`请查看本指南的[第1部分](https://www.jianshu.com/p/603d671400da)。

```javascript
subscribeBars: (symbolInfo, resolution, onRealtimeCallback, subscribeUID, onResetCacheNeededCallback) => {  
    
},
```

**图表库将这些参数传递给**`subscribeBars`**：**

-   `symbolInfo`- `Object`symbolInfo对象
-   `resolution`- `String`K线周期
-   `onRealtimeCallback`- `Function`将我们更新的K线传递给此回调以更新图表
-   `subscribeUID`- `String`此交易对的唯一ID和表示订阅的分辨率，生成规则：ticker+'_'+周期
-   `onResetCacheNeededCallback`- `Function`调用次回调让图表再次请求历史K线数据

在实现方面，当图表商品或周期发生变化时，或者图表需要订阅新商品时，图表库会调用subscribeBars。

当订阅K线实时数据时，我们需要创建一个ws订阅记录，里面需要包含onRealtimeCallback函数（可以将onRealtimeCallback公开到window对象中），这样您就可以使用从实时数据源接收的新数据调用onRealtimeCallback函数。

JS API是一个传递给图表库的JS对象，它必须包含TradingView定义的函数。这些函数由图表库根据需要调用，您不能自己调用​​它们，只能调用它们的回调函数。

你应该做的是保持对订阅的引用，然后将更新的K线传递给subscribeBars函数的回调`realtimeUpdateCallback`。

----------

### **让我们开始行动！**

假设我们有两个文件，`api/index.js`JS API所在的文件，以及`api/stream.js`我们的实时更新代码：

```javascript
// api/index.js

import stream from './stream'

...

subscribeBars: (symbolInfo, resolution, onRealtimeCallback, subscribeUID, onResetCacheNeededCallback) => {

  stream.subscribeBars(symbolInfo, resolution, onRealtimeCallback, subscribeUID, onResetCacheNeededCallback)  
 },  
...
```

请记住，上面的很多内容涉及我正在使用的数据源的细节，CryptoCompare的Socket.io websocket频道。

我们需要了解图表库中没有提供的东西，我们需要知道图表上的`lastBar`。我们需要拥有一个对指定图表的onRealtimeUpdateCB引用。

TradingView特有的部分如下：

-   创建ws订阅记录，以便我们可以存储对`updateCB`的引用并将数据传递到右侧图表
-   `lastBar`通过我们数据源的更新或创建新K线
-   在TradingView调用`subscribeBars`时，将更新/创建的K线数据传递给`updateCB`

从TradingView图表库的角度来看，就这么简单。但是，使用您自己的数据源来实现这个流程可以是简单或复杂的。

### TV要求

通过更新图表上最新K线（当前的K线）来实现K线的实时更新。当实时价格数据进入时，您需要为图表库提供该K线数据的更新版本。

JS API提供了两个管理它的功能。`susbcribeBars`和`unsubscribeBars`。

**当图表加载商品，或者当前图表的周期发生变化时**，TV将通过使用想要订阅的商品和周期调用`subscribeBars`来请求订阅该K线的实时更新。

```javascript
subscribeBars: (symbolInfo, resolution, onRealtimeCallback, subscribeUID, onResetCacheNeededCallback) => {  
  console.log('=====subscribeBars runnning')  
}

unsubscribeBars
```

TV将调用`subscribeBars`函数以开启商品的实时订阅。为了将更新数据传递给图表，您需要调用`onRealtimeCallback`。

> **请注意，TV只调用一次`subscribeBars`，当您需要更新该订阅的图表时，您需要保留对**`onRealtimeCallback`**函数的引用。**

所以您要负责：

-   连接到实时数据源
-   管理订阅，TV会通过JS API `subscribeBars`和`unsubscribeBars`方法通知您。
-   持续更新图表上最新K线
-   将数据修改为TV格式

TV的K线格式是一个如下所示的对象：

```json
{   
time： 1528322340000，//K线时间必须以毫秒为单位  
open： 7678.85，  
high： 7702.55，  
low：  7656.98，  
close： 7658.25，//收盘价是开盘后最新的价格  
volume： 0.9834   
}
```

了解TV的需求与您自己的数据实现之间的区别非常重要。您可以使用任何数据，只要您以正确的格式将其提供给TV图表库即可。

下面是我自己实现[CryptoCompare的](https://www.cryptocompare.com/api/#-api-web-socket-) socket.io交易数据websocket 的例子。

**这只是完整JS API实现的一部分**，我们从这个文件中导出两个方法，这些方法将从我们传递给TV的JS API对象中调用。**有关JS API的更多信息，****请参阅****本教程的**[**第1部分**](https://www.jianshu.com/p/603d671400da)

```javascript
// api/stream.js
import historyProvider from './historyProvider.js';
// 我们使用Socket.io客户端连接cryptocompare的socket.io数据流
var io = require('socket.io-client');
var socket_url = 'wss://streamer.cryptocompare.com';
var socket = io(socket_url);
// 正在订阅的对象数组
var _subs = [];

export default {
  subscribeBars: function(symbolInfo, resolution, updateCb, uid, resetCache) {
    const channelString = createChannelString(symbolInfo);
    socket.emit('SubAdd', { subs: [channelString] });

    var newSub = {
      channelString,
      uid,
      resolution,
      symbolInfo,
      lastBar: historyProvider.history[symbolInfo.name].lastBar,
      listener: updateCb,
    };
    _subs.push(newSub);
  },
  unsubscribeBars: function(uid) {
    var subIndex = _subs.findIndex((e) => e.uid === uid);
    if (subIndex === -1) {
      //console.log("No subscription found for ",uid)
      return;
    }
    var sub = _subs[subIndex];
    socket.emit('SubRemove', { subs: [sub.channelString] });
    _subs.splice(subIndex, 1);
  },
};

socket.on('connect', () => {
  console.log('===Socket connected');
});
socket.on('disconnect', (e) => {
  console.log('===Socket disconnected:', e);
});
socket.on('error', (err) => {
  console.log('====socket error', err);
});
socket.on('m', (e) => {
  //这里我们得到CryptoCompare连接订阅的所有事件
  //我们需要将这些新数据发送到我们订阅的图表上
  const _data = e.split('~');
  if (_data[0] === '3') {
    // console.log('Websocket Snapshot load event complete')
    return;
  }
  const data = {
    sub_type: parseInt(_data[0], 10),
    exchange: _data[1],
    to_sym: _data[2],
    from_sym: _data[3],
    trade_id: _data[5],
    ts: parseInt(_data[6], 10),
    volume: parseFloat(_data[7]),
    price: parseFloat(_data[8]),
  };

  const channelString = `${data.sub_type}~${data.exchange}~${data.to_sym}~${data.from_sym}`;

  const sub = _subs.find((e) => e.channelString === channelString);

  if (sub) {
    // 如果是历史K线数据，则跳过
    if (data.ts < sub.lastBar.time / 1000) {
      return;
    }

    var _lastBar = updateBar(data, sub);

    // 将最新的K线发送给TV的realtimeUpdate回调
    sub.listener(_lastBar);
    // 更新我们自己的lastBar记录
    sub.lastBar = _lastBar;
  }
});

// 取得交易数据和订阅记录, 返回更新后的最新K线
function updateBar(data, sub) {
  var lastBar = sub.lastBar;
  let resolution = sub.resolution;
  if (resolution.includes('D')) {
    // 一天的分钟数 === 1440
    resolution = 1440;
  } else if (resolution.includes('W')) {
    // 一周的分钟数 === 10080
    resolution = 10080;
  }
  var coeff = resolution * 60;
  // console.log({coeff})
  var rounded = Math.floor(data.ts / coeff) * coeff;
  var lastBarSec = lastBar.time / 1000;
  var _lastBar;

  if (rounded > lastBarSec) {
    // 创建一个新的K线，使用最后的收盘价作为开盘价**个人选择**
    _lastBar = {
      time: rounded * 1000,
      open: lastBar.close,
      high: lastBar.close,
      low: lastBar.close,
      close: data.price,
      volume: data.volume,
    };
  } else {
    // 更新最新的K线
    if (data.price < lastBar.low) {
      lastBar.low = data.price;
    } else if (data.price > lastBar.high) {
      lastBar.high = data.price;
    }

    lastBar.volume += data.volume;
    lastBar.close = data.price;
    _lastBar = lastBar;
  }
  return _lastBar;
}

// 将symbolInfo对象作为输入参数，并创建要发送到CryptoCompare的订阅字符串
function createChannelString(symbolInfo) {
  var channel = symbolInfo.name.split(/[:/]/);
  const exchange = channel[0] === 'GDAX' ? 'Coinbase' : channel[0];
  const to = channel[2];
  const from = channel[1];
  // 订阅指定交易所和交易对的CryptoCompare交易频道
  return `0~${exchange}~${from}~${to}`;
}
```

不过，您的数据源实现可能与我的不同，除非您也使用CryptoCompare！

**一般步骤是：**

-   提供拥有subscribe/unsubscribeBars方法的JS API对象给TV
-   记录相应的订阅对象，保存TV传递给您`realtimeUpdateCallback` 、subscriberUID和resolution。您还需要图表上的最新K线，我的historyProvider负责提供最新K线。
-   订阅您的自己数据源的ws频道
-   获取来自数据源的数据，更新您的lastBar（或者如果最新K线自上次收到数据后相应周期已结束，则创建一个新K线）
-   将更新的K线数据传递给realtimeCallback。
-   处理取消订阅

**让我们分析一下我正在做些什么来实现CryptoCompare的特定实时数据来处理这个问题。** 这仅限于TV，因为我正在按照他们指定的格式，格式化我可用的数据。

-   为指定的交易对/交易所订阅交易级别的更新频道。Cryptocompare处理许多交易所，以及列出的所有交易对，这就是我将它们写到商品名称中的原因，例如，`Coinbase:BTC/USD`为我提供了使用CryptoCompare订阅正确频道所需的信息
-   在订阅频道时，收听从CryptoCompare发送的“快照”，快照数据是该频道最后几分钟的交易，这使我可以在价格发生变化时，更新历史数据。这些更新的发送方式与所有未来的更新一样，因此除了忽略比图表上的最新K线更旧的过时更新之外，我不必做任何事情。
-   我正在将每个更新的时间戳修改为我们使用的交易周期的相应时间戳，这是为了更新最新K线所必需的，并确定何时我们需要创建一个新的K线。
-   更新我们的订阅记录中的最新K线。记录新的高、低、开、交易量和收盘价。收盘价始终是您获得的最后一次价格更新。对于周期为结束的K线，“收盘价”为当前价格。
-   将更新后的K线发送到updateCB以进行正确的订阅处理。

### 目前为止就这么多了！

> **你可以看到部署在演示在**[Glitch](https://tv-tut-part2.glitch.me/)**，并查看第2部分的**[**代码**](https://github.com/jonchurch/tradingview-js-api-tutorial/tree/master/part2)

希望这有助于回答有关Trading View 图表库实时更新的一些问题！

这是我在图表库中系列中最受欢迎的部分，我打算随着时间的推移更新它，因此我简化了这个过程。

本系列的第3部分将介绍更多TradingView功能（有非常多！），并将很快推出。我计划创建一些更好的起始项目，以加快开发tradingview的过程。

----------

  
### 关于译者

我是一名现就职于[日本最大数字货币交易所bitbank](bitbank.cc)的全栈开发人员，住在东京。如果您需要加快将TradingView放入到您的网站或交易网页，我可以付费帮忙。

**您可以联系我：zlq4863947@gmail.com**

#### 您也可以加入tradingview开发交流qq群：313839516

### 关于本篇译文

#### 英文原文地址： [TradingView Charting Library JS API Setup for Crypto: Realtime Chart Updates](https://medium.com/@jonchurch/tv-part-2-b8c2d4211f90)