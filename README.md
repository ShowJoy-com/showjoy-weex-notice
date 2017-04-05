# showjoy-weex-notice
> 记录团队weex实践过程中需要特殊注意的点，对部分bugs的hack，以及版本升级后的功能更新介绍。

## 如何适配屏幕高

```
setListHeight: function () {
  let self = this;
  let deviceHeight = parseInt(weex.config.env.deviceHeight);
  let rate = weex.config.env.deviceWidth / 750;
  let deviceScale = weex.config.env.scale;

 if (weex.config.env.platform.toLowerCase() === 'web') {
    self.listHeight = deviceHeight / rate;
  } else {
    self.listHeight = (deviceHeight - (64 * deviceScale)) / rate;
  }
}
```
在进入页面时调用`setListHeight`方法就能完美的在所有屏下设定全屏高，其中的`(64 * deviceScale)`是iOS与Android的titlebar以及顶部系统区域的高，需要减掉。在我司的产品中titlebar以及顶部系统区域的高是固定的。
