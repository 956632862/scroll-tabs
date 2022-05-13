# Scroll-Tabs

uniapp支持点击滚动到对应的tab的选项卡组件，也支持滚动到对应组件时切换相应的tab选项。
下面其中的**scrollTop为必传项，下面的表格不知道为什么展示不全部，需要在父页面中调用onPageScroll获取到滚动位置之后传入**

## 组件参数说明

### ScrollTabs

#### 参数配置

参数|说明|类型|默认值|必传
:-:|:--:|:-:|:-:|---
scrollTop|**由于组件内部无法获取滚动条位置，需要在父级组件调用`onPageScroll`获取**|Number|0|是
current|选中的tab下标|Number|0|否
tabs|选项卡列表，**详情见下面说明**|Array|[ ]|是
tabOptions|选项卡组件的配置 **详情见下面说明**|String|0rpx|否
sticky|是否设置选项卡为固定定位|Boolean|true|否
itemOffsetTop|滚动相隔间距，到达会切换tab|Number|60|否
stickyTop|固定定位的top高度，传入时需要携带单位（如 0rpx）|Object|{}|否
scrollTab|开启滚动时切换到指定的tab|Boolean|true|否
clickScroll|开启点击时滚动到相应位置|Boolean|true|否
offsetTop|滚动偏离值，添加之后，每次滚动都会减去这个值，数值单位px|Number|0|否



#### 事件

|  事件名  |      说明       |                      传参                      |
| :------: | :-------------: | :--------------------------------------------: |
| onChange | tab切换事件监听 | （tab,index）选中的tab信息,选中的选项下标index |



### Tab列表传参说明

插件实现是原理是使用了uniapp的 `createSelectorQuery`api，所以在需要进行相应滚动的操作的时候，需要在对象中的`scroll_id`中传入对应元素的`class`名，否则无法进行滚动操作，配置示例：

```vue
<template>
    <view class="content">
        <scrollTabs
          :tabs="tabs"
        >
            <view class="item _a">我第一个</view>
            <view class="item _b">我第2个</view>
            <view class="item _c">我第3个</view>
        </scrollTabs>
    </view>
</template>

<script>
import scrollTabs from "./scroll-tabs.vue"
export default {
  data(){
    return {
        tabs:[
          {label:'文件',scroll_id:'_a'},
          {label:'编辑',scroll_id:'_b'},
          {label:'视图',scroll_id:'_c'},
          {label:'导航',scroll_id:'_d'},
          {label:'代码',scroll_id:'_e'},
        ]
    }
  }
}
</script>
```



### TabOptions配置说明

选项卡组件使用的别的组件，api查看这个文档 https://github.com/956632862/easy-tabs
