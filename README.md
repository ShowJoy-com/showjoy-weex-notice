# showjoy-weex-notice
> 记录团队weex实践过程中需要特殊注意的点，对部分bugs的hack，以及版本升级后的功能更新介绍。



## （一）如何适配/获取屏幕高度？

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

我们在开发页面的时候经常会使用scroller、list进行；列表渲染，而列表又需要指定高度，我们可以通过一定方法获取到不同设备的可绘制区域的高度，方便布局。

### 解决方案：

```javascript
/**
    * 
    * @description 设置list或者scroller的全屏高度
    * @param {Number} [height] - 需要减掉的高度 「default: 0」
    * @param {Boolean} [isAbsHeight] - 是否使用绝对高度
    *                                    任意屏都显示相同的高度 「default: false」
    * @return {Number}
    */
    getListHeight: function (height, isAbsHeight) {

      let deviceHeight = parseInt(weex.config.env.deviceHeight);
      let rate = weex.config.env.deviceWidth / 750;
      let deviceScale = weex.config.env.scale;

      if (weex.config.env.platform.toLowerCase() === 'web') {

        if (height && typeof height === 'number') {

          if (isAbsHeight && typeof isAbsHeight === 'boolean') {
            return (deviceHeight / rate) - ((height / 2) / deviceScale);
          } else {
            return (deviceHeight / rate) - height;
          }

        } else {
          return (deviceHeight / rate);
        }

      } else {

        if (height && typeof height === 'number') {

          if (isAbsHeight && typeof isAbsHeight === 'boolean') {
            return (deviceHeight - ((64 + (height / 2)) * deviceScale)) / rate;
          } else {
            return (deviceHeight - (64 * deviceScale)) / rate - height;
          }

        } else {

          return (deviceHeight - (64 * deviceScale)) / rate;
        }
      }
    },
```
在进入页面时调用`getListHeight`方法就能完美的在所有屏下设定全屏高，其中的`(64 * deviceScale)`是iOS与Android的导航栏高度，需要减掉。在我司的产品中titlebar以及顶部系统区域的高是固定64的。


## （二）class动态绑定

根据vue[官方文档](https://cn.vuejs.org/v2/guide/class-and-style.html#对象语法)，动态绑定class是支持对象形式的，但是weex内不支持

```
:class={'header': true}
```

### 解决方法 

我们要动态改变一个元素的class时需要使用较繁琐的数组形式。

```
:class="[true ? 'header' : '']"
```

## （三）animation动画在ios8及以下的H5页面失效

使用官方提供的`animation.transition`方法，在低版本ios系统中部分动画无法生效，原因在于`0.10.4`版本的H5Sdk（weex-vue-render）中没有对webkit做兼容。

### 解决方法

对于webkit不兼容的css样式(transform)进行兼容

```
animation.transition(cWrap, {
    styles: {
        transform: 'scale(1, 1)',
        '-webkit-transform': 'scale(1, 1)', // 兼容H5的ios7、8
    },
    duration: 300,
    timingFunction: 'ease-in-out',
    delay: 0,
});
```

## （四）scroller横向滚动时Ios设备元素无法横向排列

> ios sdk 0.13.0

当给scroller设置`scroll-direction="horizontal"`属性时，安卓与H5设备scroller内的元素都能横排显示，ios无法横排显示，且无法左右滑动也无法上下滑动。

### 解决方法

需要给scroller设置样式 `flex-deriction: row`，这样可以确保三端显示一致。

## （五）native环境官方提供的部分组件的属性设置无效

> 老版的weex-loader打包

当我们使用官方提供的组件时，有自定义的属性涉及到连接符时可能会使用不成功。比如scroller组件的`scroll-direction`属性，或者slider组件的`auto-play`属性，如果使用老版的weex-loader进行打包，这两个属性都会设置失败，需要改成驼峰写法。`scrollDirection`和`autoPlay`。新版的weex-loader没有这样的问题是因为在打包过程中，工具将这些属性全部自动换成了驼峰形式。

### 解决方法（任选）

1、升级weex-loader打包工具
2、连接符形式的属性写成驼峰形式

## （六）$parent在native和H5中取到的层级不一样

在编写复合组件时可能会需要在子组件去取当前父组件，native和H5取到的$parent实例是不同的。

### 解决方法

```
const parentVm = weex.config.env.platform.toLowerCase() === 'web' ?
      self.$parent.$parent.$parent : self.$parent;
```

## （七）native中getComponentRect方法不准确

如果有需求在页面加载时需要获取某个dom元素的位置及空间信息，mounted生命周期执行时`getComponentRect`方法并不能100%获取真实数据。

关于`getComponentRect`[文档](http://weex-project.io/cn/references/modules/dom.html)

### 解决方法
目前使用该api的场景为dom元素已经100%渲染完毕，调用此方法可以正确获取数据。


## （八）shopBase.close() 兼容

因为 H5 并不支持 `close()` 方法，因此在使用该接口方法时，需要做 `web` 的判断

```js
if (!self.isWeb) {
    shopBase.close();
} else {
    shopBase.openUrl(`/weexCommon/after-sale-schedule-weex.html`);
}
```

## （九）globalEvent 使用

由于 H5 没有 `globalEvent` 模块，因此使用时要先进行判断，更推荐的方案是使用 `self.addEventListener` 方法

```js
import { event } from 'components/weex-mixins/weex-mixins';

mixins: [event]

self.addEventListener('resume', () => {
   if (self.isSubmitSuccess) {
       shopBase.close();
       shopBase.fireGlobalEvent('close-after-sale-option', { }, () => { });
   }
});
```

