<template>
  <div id="app">
    <m-header></m-header>
    <tab></tab>
    <!--缓存到内存中，减少数据请求-->
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
  </div>
</template>

<script type="text/ecmascript-6">
  import MHeader from 'components/m-header/m-header.vue'
  import Tab from 'components/tab/tab'

  export default {
    name: 'App',
    components: {
      MHeader,
      Tab
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import 'common/stylus/variable.styl'
  #app
    color: $color-theme
</style>
