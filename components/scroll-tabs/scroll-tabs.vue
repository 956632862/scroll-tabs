<template>
  <view class="scroll-tabs-container" >
    <view :class="{'scroll-tabs-sticky':sticky}" :style="{top:stickyTop || localStickyTop}">
      <easyTabs
          v-if="tabs.length"
          id="tabScrollTop"
          ref="tabScrollTop"
          :list='tabs'
          :current="active"
          :label="reTabOptions.label"
          :activeColor="reTabOptions.activeColor"
          :inactiveColor="reTabOptions.inactiveColor"
          :duration="reTabOptions.duration"
          :barHeight="reTabOptions.barHeight"
          :barWidth="reTabOptions.barWidth"
          :flexBetween="reTabOptions.flexBetween"
          :itemStyle="reTabOptions.itemStyle"
          :customClass="reTabOptions.customClass"
          @change='handleChangeTab'
      />
    </view>
    <view class="content">
      <slot></slot>
    </view>
  </view>
</template>

<script>
import easyTabs from "../easy-tabs/easy-tabs"
/**
 * @description props参数说明
 * @param scrollTop 外部传入的滚动条高度
 * @param tabs tabs
 * @param current 当前选中的tab
 * @param stickyTop 固定定位的高度
 * @param sticky 是否设置tab为固定定位
 * @param itemOffsetTop 自定义滚动相隔间距，到达会切换tab
 * @param tabOptions tab的配置
 * @param scrollTab 滚动时是否切换到指定的tab
 * @param clickScroll 点击时是否滚动到相应位置
 * @param duration  滚动持续时长
 * @param offsetTop 点击滚动到某一模块时的偏离值，每次点击滚动都会减去这个偏离值,这个值的单位是px，
 */
export default {
  components:{easyTabs},
  props:{
    scrollTab:{
      type:Boolean,
      default:true
    },
    clickScroll:{
      type:Boolean,
      default:true
    },
    scrollTop:{
      type:Number,
      default:0
    },
    tabs:{
      type:Array,
      default:() => ([])
    },
    // 当前选中的tab
    current:{
      type:Number,
      default:0
    },
    stickyTop:{
      type:String,
      default:null
    },
    // 是否将tab设为固定定位
    sticky:{
      type:Boolean,
      default:true
    },
    // 自定义间距 ,当 top小于等于这个距离的时候会切换tab
    itemOffsetTop:{
      type:Number,
      default:60
    },
    tabOptions:{
      type:Object,
      default:()=>({})
    },
    duration:{
      type:Number,
      default:300
    },
    offsetTop:{
      type:Number,
      default:0
    }
  },
  data(){
    return {
      tabTopHeight:0,
      active:0,
      click:false,
      timer:null,
      localStickyTop:'0rpx'
    }
  },
  computed:{
    reTabOptions(){
      const defaultOptions = {
        label:'label',
        activeColor:'#62C085',
        inactiveColor:'#666666',
        duration:0.5,
        barHeight:6,
        barWidth:50,
        flexBetween:false,
        customClass:''
      }
      return Object.assign(defaultOptions,this.tabOptions)
    }
  },
  watch:{
    scrollTop(){
      if (!this.click){
        this.scrollToTab()
      }
    },
    current:{
      immediate: true,
      handler:function (nVal) {
        this.active = nVal
        this.$nextTick(() => {
          this.scrollToElement()
        })
      }
    }
  },
  mounted() {
    this.getScrollTabTopHeight()
    // this.scrollToElement()
    // const res = uni.getSystemInfoSync();
    // this.$nextTick(() => {
    //   // * 2是因为获取到的高度是根据px来算的
    //   // console.log(res,'res')
    //   // this.localStickyTop = `${res.statusBarHeight * 2}rpx`
    // })
  },
  methods:{
    // 获取选项卡的高度
    async getScrollTabTopHeight(){
      // 获取tab的高度
      const query  = await this.createSelectorQueryForThis('#tabScrollTop')
      if (!query) return ;
      this.tabTopHeight = query.height
    },
    handleChangeTab({index,tab}){
      this.active = index
      this.scrollToElement()
      this.$emit('onChange',tab,index)
    },

    // 滚动时切换到指定的tab
    async scrollToTab(){
      if (!this.scrollTab && !this.click) return false
      const length = this.tabs.length
      let allClass = ''
      for (let i = 0; i < length; i++){
        const {scroll_id} = this.tabs[i]
        allClass += i < length - 1 ? `.${scroll_id},` : `.${scroll_id}`
      }
      const queryData = await this.createSelectorQuery(allClass,true)
      for (let i = 0; i < queryData.length; i++) {
        if (queryData[i].top <= this.itemOffsetTop){
          this.active  = i
        }
      }
    },
    // 点击滑动到指定元素
    async scrollToElement(){
      if (!this.clickScroll) return false
      const tab = this.tabs[this.active]
      if(!tab)return
      const {scroll_id} = tab
      if (!scroll_id) return false
      this.click =  true
      clearTimeout(this.timer)
      const queryData =  await this.createSelectorQuery(`.${scroll_id}`)
      if(this.tabTopHeight === 0) await this.getScrollTabTopHeight()

      if (!queryData){
        this.click = true
        return false
      }
      // 页面滚动函数
      uni.pageScrollTo({
        scrollTop: this.scrollTop + queryData.top - this.tabTopHeight - this.offsetTop,
        duration: this.duration,
        success:()=>{
          this.timer =  setTimeout(()=>{
            this.click = false
          },this.duration + 500)
        }
      });
    },
    createSelectorQueryForThis(selector,all){
      return new Promise(resolve => {
        uni.createSelectorQuery().in(this)[all ? 'selectAll' : 'select'](selector).boundingClientRect(rect => {
          resolve(rect)
        }).exec()
      })
    },
    createSelectorQuery(selector,all){
      return new Promise(resolve => {
        uni.createSelectorQuery()[all ? 'selectAll' : 'select'](selector).boundingClientRect(rect => {
          resolve(rect)
        }).exec()
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.scroll-tabs-container{
  .scroll-tabs-sticky{
    position: sticky;
    z-index: 9999;
    overflow: hidden;
    -webkit-transform: translateZ(0);
    transform: translate3d(0, 0, 0);
    background: white;
  }
}
</style>
