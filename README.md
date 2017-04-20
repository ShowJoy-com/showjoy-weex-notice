# showjoy-weex-notice
> 记录团队weex实践过程中需要特殊注意的点，对部分bugs的hack，以及版本升级后的功能更新介绍。



## 如何适配屏幕高度？

### 原因：

![iOS屏幕示例](http://cdn1.showjoy.com/images/2e/2ee591732c3c49fe8778d34f111c234e.png)



| 开发环境    | 单位     | 描述                                |
| ------- | ------ | --------------------------------- |
| Weex    | px     | Pixels（设备分辨率）                     |
| iOS     | pt     | Points（逻辑分辨率）                     |
| Android | dp或dip | Density-Independent pixel（设备独立像素） |

ppi：每英寸的长度中所具有的像素，屏幕像素密度
> 每个屏幕的ppi不同，最终导致屏幕分辨率不同

| 常用倍率 | 值      | 描述                      |
| ---- | ------ | ----------------------- |
| @1x  | 163ppi | iPhone3gs等              |
| @2x  | 326ppi | iPhone4、4s、5、5s、6、6s、7等 |
| @3x  | 401ppi | iPhone6P、6sP、7P等        |



### 解决方案：

```javascript
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
在进入页面时调用`setListHeight`方法就能完美的在所有屏下设定全屏高，其中的`(64 * deviceScale)`是iOS与Android的导航栏高度，需要减掉。在我司的产品中titlebar以及顶部系统区域的高是固定的。



## Weex列表加载功能抖动？

### 原因：

使用<list> <cell>和<loading>组件，调用@loading触发加载更多导致列表抖动。



### 解决方案：

**方案1：**（推荐）

> 使用<scroller><div>和<loading>组件，调用@loading触发加载更多。

```html
<!-- scroller结构示例 -->
<scroller class="scroller">
    <div class="cell" v-for="num in lists">
      <div class="panel">
        <text class="text">{{num}}</text>
      </div>
    </div>
    <loading class="loading" @loading="loadMoreItem" :display="showLoading">
      <!-- @loading加载更多 -->
    </loading>
  </scroller>
```



**方案2：**（列表数据全部加载后，无法再触发@loadmore事件）

> 使用<list><cell>组件，调用@loadmore触发加载更多。

```html
<!-- list结构示例 -->
<list class="list" @loadmore="loadMoreItem">
  <!-- @loadmore加载更多 -->
    <cell class="cell" v-for="num in lists">
      <div class="panel">
        <text class="text">{{num}}</text>
      </div>
    </cell>
  </list>
```