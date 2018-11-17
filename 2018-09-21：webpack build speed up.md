## webpack build speed up
 - ref:[Speeding up webpack](https://medium.com/onfido-tech/speed-up-webpack-ff53c494b89c)
 - ref:[0–100 in two seconds — speed up webpack](https://medium.com/ottofellercom/0-100-in-two-seconds-speed-up-webpack-465de691ed4a)

## 事前的 npm run build
 - 時間：Time: 120343ms => 120 秒～ 2分鐘

## 第一招 => UglifyJsPlugin
```javascript
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

module.exports = {
  //...
  optimization: {
    minimizer: [new UglifyJsPlugin({ parallel: true })]
  }
};
```
多 CPU 平行處理，這台筆電為 2 core，能加速多少呢？

 - 時間：Time: 108231ms => 108 秒 ，省下 12秒

## 第二招 => HardSourceWebpackPlugin
```javascript
// webpack.config.js
var HardSourceWebpackPlugin = require('hard-source-webpack-plugin');

module.exports = {
  context: // ...
  entry: // ...
  output: // ...
  plugins: [
    new HardSourceWebpackPlugin()
  ]
}
```

要跑兩次才能比較時間，第一次時間會一樣，第二次才有效果。
 - 時間1：Time: 104586ms
 - 時間2：Time: 67133ms => 67 秒，真的快～

document 有說到其他 config 檔可能會抓不到變動的議題，但應該很難遇到。
 
>  The default configHash may not detect all of your options to plugins or other configuration files like `.babelrc` or `postcss.config.js`. In those cases a custom `configHash` is **needed hashing the webpack config and those other values that it cannot normally reach**.

## 第三招 IgnorePlugin 
上面東西太少了，在另外找一些。momentJS 的 locale file 太大了，忽略它吧。
```javascript
const webpack = require('webpack');
module.exports = {
  //...
  plugins: [
    // Ignore all locale files of moment.js
    new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/),
  ],
};
```

時間更長了... 沒有幫助？  
跑第二次真的比較久  
怪，拿掉後又變成 2 分鐘...，先放回去好了。

先用這幾招試試看