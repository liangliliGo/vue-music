<template>
  <div ref="wrapper">
    <slot></slot>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'

  export default {
    props: {
      probeType: {
        type: Number, //什么时候需要知道滚动的位置 1.超过一定时间派发Scroll事件 2.滚动过程中实时派发 3.滚动过程中和momentum滚动动画运行过程中都会实时派发
        default: 1
      },
      click: {
        type: Boolean,
        default: true
      },
      listenScroll: {
        type: Boolean,
        default: false
      },
      data: {
        type: Array,
        default: null
      },
      pullup: {
        type: Boolean,
        default: false
      },
      beforeScroll: {
        type: Boolean,
        default: false
      }
    },
    mounted () {
      setTimeout(() => {
        this._initScroll()
      }, 20)
    },
    methods: {
      _initScroll () {
        if (!this.$refs.wrapper) {
          return
        }
        this.scroll = new BScroll(this.$refs.wrapper, {
          probeType: this.probeType,
          click: this.click
        })

        if(this.listenScroll) {
          /*作用域导致的this指向问题，this指向当前vue实例*/
          let _this = this;
          this.scroll.on('scroll', (pos) => {
            _this.$emit('scroll', pos)
          })
        }

        if(this.pullup) {
          this.scroll.on('scrollEnd', () => {
            if(this.scroll.y <= (this.scroll.maxScrollY + 50)){
              this.$emit('scrollToEnd')
            }
          })
        }

        if(this.beforeScroll) {
          this.scroll.on('beforeScrollStart', () => {
            this.$emit('beforeScroll')
          })
        }
      },
      disable () {
        this.scroll && this.scroll.disable()
      },
      enable () {
        this.scroll && this.scroll.enable()
      },
      refresh () {
        this.scroll && this.scroll.refresh()
      },
      scrollTo () {
        this.scroll && this.scroll.scrollTo.apply(this.scroll, arguments)
      },
      scrollToElement() {
        this.scroll && this.scroll.scrollToElement.apply(this.scroll, arguments)
      }
    },
    watch: {
      data () {
        setTimeout(() => {
          this.refresh()
        }, 20)
      }
    }
  }
</script>
<stylus scoped lang="stylus" rel="stylesheet/stylus">
</stylus>
