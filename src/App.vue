<template>
  <div id="app">
    <v-header :seller="seller"></v-header>

    <div class="tab border_1px">
      <router-link class="tab_item" to="/goods">商品</router-link>
      <router-link class="tab_item" to="/ratings">评价</router-link>
      <router-link class="tab_item" to="/seller">商家</router-link>
    </div>
    <!-- keep-alive能确保切换组件的时候，组件保留在内存中不被重新渲染，这样切换后，加入到购物车中的商品不会被清零 -->
    <keep-alive>
      <router-view :seller="seller"></router-view>
    </keep-alive>
  </div>
</template>

<script>
import vHeader from './components/vHeader/v-header';

const ERR_OK = 0;
export default {
  data() {
    return {
      seller: {}
    };
  },
  components: {
    vHeader
  },
  created() {
    this.$http.get('/api/seller').then((res) => {
      console.log(res);
      res = res.body;
      if (res.errno === ERR_OK) {
        this.seller = res.data;
      }
    });
  }
};
</script>

<style lang="stylus">
  @import './common/stylus/common.styl';
  
  .tab
    display: flex
    border_1px(rgba(7,17,27,.1))
    .tab_item
      flex: 1
      height: 40px
      line-height: 40px
      text-align: center
      font-size: 14px
      color: rgb(77,85,93)
      text-decoration: none
      &.router-link-active
        color: $red
      
</style>
