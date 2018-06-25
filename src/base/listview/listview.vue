<template>
  <scroll class="listview" ref="listview" :data="data" :listen-scroll="listenScroll" @scroll="scroll"
          :probe-type="probeType">
    <ul>
      <li class="list-group" v-for="(group,index) in data" :key="index" ref="listGroup">
        <h2 class="list-group-title">{{group.title}}</h2>
        <ul>
          <li class="list-group-item" v-for="(item, index) in group.items" :key="index" @click="selectItem(item)">
            <img v-lazy="item.avatar" class="avatar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div class="list-shortcut" @touchstart="onShortcutTouchStart"
         @touchmove.stop.prevent="onShortcutTouchMove">
      <ul>
        <li class="item" v-for="(item, index) in shortcutList" :data-index="index"
            :class="{'current': currentIndex === index}">{{item}}
        </li>
      </ul>
    </div>
    <div class="list-fixed" ref="fixed" v-show="fixedTitle">
      <div class="fixed-title">{{fixedTitle}}</div>
    </div>
    <div class="loading-container" v-show="!data.length">
      <loading></loading>
    </div>
  </scroll>
</template>
<script type="text/ecmascript-6">
  import Scroll from 'base/scroll/scroll'
  import Loading from 'base/loading/loading'
  import { getData } from 'common/js/dom'

  const ANCHOR_HEIGHT = 18
  const TITLE_HEIGHT = 30

  export default {
    props: {
      data: {
        type: Array,
        default: []
      }
    },
    computed: {
      shortcutList () {
        /*获取滚动列表*/
        return this.data.map((group) => {
          return group.title.substr(0, 1)
        })
      },
      fixedTitle () {
        if (this.scrollY > 0) {
          return ''
        }
        return this.data[this.currentIndex] ? this.data[this.currentIndex].title : ''
      }
    },
    data () {
      return {
        scrollY: -1,
        currentIndex: 0,
        diff: -1
      }
    },
    /*为什么不写在data中？-> 因不需要getter和setter，提高性能*/
    created () {
      /*用于在touchstart和touchmove间传递参数*/
      this.touch = {}
      /*监听scroll事件*/
      this.listenScroll = true
      /*屏幕滚动过程中和momentum滚动动画运行过程中都会实时派发scroll*/
      this.probeType = 3
      /*listGroup组 每个list距离顶部的高度*/
      this.listHeight = []
    },
    methods: {
      selectItem(item) {
        this.$emit('select', item)
      },
      onShortcutTouchStart (e) {
        let anchorIndex = getData(e.target, 'index')
        let firstTouch = e.touches[0]
        /*touches当前屏幕上所有触摸点的列表*/
        this.touch.y1 = firstTouch.pageY
        this.touch.anchorIndex = parseInt(anchorIndex)
        this._scrollTo(anchorIndex)
      },
      onShortcutTouchMove (e) {
        let firstTouch = e.touches[0]
        this.touch.y2 = firstTouch.pageY
        /* |0 向下取整，等同于Math.floor*/
        let delta = (this.touch.y2 - this.touch.y1) / ANCHOR_HEIGHT | 0
        let anchorIndex = parseInt(this.touch.anchorIndex) + delta
        this._scrollTo(anchorIndex)
      },
      _scrollTo (index) {
        /*点击右侧导航栏最上边和最下边空白区域的时候*/
        if (!index && index !== 0) {
          return
        }
        /*touchumove到最顶端和最低端的兼容*/
        if (index < 0) {
          index = 0
        } else if (index > this.listHeight.length - 2) {
          index = this.listHeight.length - 2
        }
        this.scrollY = -this.listHeight[index]
        this.$refs.listview.scrollToElement(this.$refs.listGroup[index], 0)
      },
      scroll (pos) {
        this.scrollY = pos.y
      },
      refresh () {
        this.$refs.listview.refresh()
      },
      _calculateHeight () {
        const list = this.$refs.listGroup
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < list.length; i++) {
          let item = list[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
      }
    },
    watch: {
      data () {
        setTimeout(() => {
          this._calculateHeight()
        }, 20)
      },
      scrollY (newVal) {
        const listHeight = this.listHeight
        /*当滚动到顶部以上, newVal>0*/
        if (newVal > 0) {
          this.currentIndex = 0
          return
        }

        /*在中间部分和最下边滚动的时候*/
        for (let i = 0; i < listHeight.length - 1; i++) {
          let height1 = listHeight[i]
          let height2 = listHeight[i + 1]
          if (!height2 || -newVal >= height1 && -newVal < height2) {
            this.currentIndex = i
            this.diff = height2 + newVal
            return
          }
        }
        /*this.currentIndex = 0*/
        /*当滚动到底部，且大于最后一个元素的上限*/
      },
      diff (newVal) {
        let fixedTop = (newVal > 0 && newVal < TITLE_HEIGHT) ? newVal - TITLE_HEIGHT : 0
        if( this.fixedTop === fixedTop) {
          return
        }
        this.fixedTop = fixedTop
        this.$refs.fixed.style.transform = `translate3d(0, ${this.fixedTop}px, 0)`
      }
    },
    components: {
      Scroll,
      Loading
    }
  }

</script>
<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .listview
    position: relative
    width: 100%;
    height: 100%;
    overflow: hidden
    background: $color-background
    .list-group
      padding-bottom: 30px
      .list-group-title
        height: 30px
        line-height: 30px
        padding-left 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
      .list-group-item
        display: flex
        align-items: center /*主轴交叉轴对其方式*/
        padding: 20px 0 0 30px
        .avatar
          width: 50px
          height: 50px
          border-radius: 50%
        .name
          margin-left: 20px
          color: $color-text-l
          font-size: $font-size-medium
    .list-shortcut
      position: absolute
      z-index: 30px
      right: 0px
      top: 50%
      transform: translateY(-50%)
      width: 20px
      padding: 20px 0
      border-radius: 10px
      text-align: center
      background: $color-background-d
      font-family: Helvetica
      .item
        padding: 3px
        line-height: 1
        color: $color-text-l
        font-size: $font-size-small
        &.current
          color: $color-theme
    .list-fixed
      position: absolute
      top: 0
      left: 0
      width: 100%
      .fixed-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
    .loading-container
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
</style>
