<template>
<view class="easy-tabs-container" :class="customClass ? customClass : ''">
  <view id="easy-tabs-scroll-container" >
    <scroll-view scroll-x class="easy-tabs-scroll" :scrollLeft="scrollLeft" scroll-with-animation>
      <view class="easy-tabs-box" :class="{'easy-tabs-box-flex-space-between': flexBetween}">
        <!--列表-->
        <view
            class="easy-tabs-item"
            :id="'easy-tab-item-'+index"
            :style="[tabItemStyle(index)]"
            v-for="(item,index) in list"
            :key="index"
            @tap="handleSelectItem(index)"
        >
          {{item[label]}}
        </view>
        <!--滚动条-->
        <view class="easy-tab-bar" :style="[tabBarStyle]"></view>
      </view>
    </scroll-view>
  </view>
</view>
</template>

<script>
/**
 * @description 参数说明
 * @param [array] list  tab列表
 * @param [number] current 当前选中项 支持.sync
 * @param [string] label label字段名
 * @param [string] activeColor 选中颜色
 * @param [string] inactiveColor   默认颜色
 * @param [number] duration  过渡时间 (s)
 * @param [number] barHeight tabBar的高度
 * @param [string | number] barWidth  tabBar的宽 设置为auto的时候，会根据tab的宽度自动变化
 * @param [boolean] flexBetween 是否开启均匀分布
 * @param [object] itemStyle  tab-item的内联样式
 * @param [string] customClass 最外层自定义class
 */
export default {
  name: "easy-tabs",
  props:{
    list:{
      type:Array,
      default:() => []
    },
    current:{
      type:Number,
      default:0
    },
    label:{
      type:String,
      default:'label'
    },
    activeColor:{
      type:String,
      default:"#62C085",
    },
    inactiveColor:{
      type:String,
      default:"#666666",
    },
    duration:{
      type: [String, Number],
      default: 0.5
    },
    barHeight:{
      type:Number,
      default:6
    },
    barWidth:{
      type:[String,Number],
      default:50
    },
    flexBetween:{
      type:Boolean,
      default:false
    },
    itemStyle:{
      type:Object,
      default:() => ({})
    },
    customClass:{
      type:String,
      default:null
    }
  },
  data(){
    return {
      active:0,
      privateBarWidth:0,
      parentInfo:{},
      scrollLeft:0,     // scroll-view 距离左边的距离
      tabItemsInfo:[],  // 所有的tab的信息
      scrollBarLeft: 0, // 移动bar需要通过translateX()移动的距离
    }
  },
  watch:{
    current:{
      immediate: true,
      handler:function (nVal) {
        this.$nextTick(() => {
          this.active = nVal
          this.scrollTabBar()
        })
      }
    }
  },
  computed:{
    tabBarStyle(){
      let style = {
        'transition-duration':this.duration + 's',
        height:this.barHeight + 'rpx',
        background: this.activeColor,
        transform: `translate(${this.scrollBarLeft}px, -100%)`,
        width: this.privateBarWidth + 'rpx',
      }
      return style
    },
    tabItemStyle(){
      return (index) => {
        const style = {}
        style.color = index === this.active ?  this.activeColor :  this.inactiveColor
        Object.assign(style,this.itemStyle)
        return style
      }
    }
  },
  mounted() {
    this.init();
  },
  methods:{
    // 初始化
    async init(){
      this.privateBarWidth = this.barWidth
      this.parentInfo = await this.createSelectorQuery('#easy-tabs-scroll-container')
      this.getTabInfo()
    },
    // 获取所有节点的信息
    getTabInfo(){
      const length = this.list.length
      let ids = ''
      for (let i = 0; i < length; i++) {
        ids += i !== length - 1 ? `#easy-tab-item-${i},` : `#easy-tab-item-${i}`
      }
      this.createSelectorQuery(ids,true).then(data => {
        this.tabItemsInfo = data
        this.scrollTabBar()
      })
    },
    // 移动滚动条并设置可选项
    scrollTabBar(){
      // 当前项
      let tabInfo = this.tabItemsInfo[this.active]
      if (!tabInfo) return;
      let parentLeft = this.parentInfo.left
      // 实际距离，因为uni的api获取都是可视范围内的距离
      let offsetLeft  = tabInfo.left - parentLeft

      // 如果是适应宽度，直接将bar移动到当前项的左边界就行
      if (this.barWidth === 'auto'){
        this.privateBarWidth = tabInfo.width * 2
        this.scrollBarLeft = offsetLeft
      }else{
        // 当前活动item的中点点到左边的距离减去滑块宽度的一半，即可得到滑块所需的移动距离 注意px 与 rpx
        this.scrollBarLeft = offsetLeft + tabInfo.width/2 - uni.upx2px(this.privateBarWidth)/2
      }

      // 同时滑动scroll-view
      let scrollLeft = offsetLeft - (this.parentInfo.width - tabInfo.width) / 2;
      this.scrollLeft = scrollLeft < 0 ? 0 : scrollLeft;
    },
    // 选中tab
    handleSelectItem(index){
      if (index === this.active) return
      this.active = index
      this.scrollTabBar()
      const tab = this.list[index]
      this.$emit('update:current',index)
      this.$emit('change',{index,tab})
    },
    createSelectorQuery(selector,all){
      return new Promise(resolve => {
        uni.createSelectorQuery().
        in(this)[all ? 'selectAll' : 'select'](selector).boundingClientRect(rect => {
          if (all && Array.isArray(rect) && rect.length) {resolve(rect)}
          if (!all && rect) {resolve(rect)}
        }).exec()
      })
    },
  }
}
</script>

<style scoped lang="scss">
.easy-tabs-container{
  .easy-tabs-scroll{
    width: 100%;
    white-space: nowrap;
    position: relative;
  }
  .easy-tabs-box{
    position: relative;
  }
  .easy-tabs-box-flex-space-between{
    display: flex;
    justify-content: space-between;
  }
  .easy-tabs-item{
    display: inline-block;
    text-align: center;
    transition-property: background-color, color;
    padding: 15rpx 30rpx 20rpx 30rpx;
  }
  .easy-tab-bar{
    position: absolute;
    bottom: 0;
  }
}

</style>
