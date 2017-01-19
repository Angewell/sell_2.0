<template>
  <div class="header">
    <div class="header_bg" :style="{'background-image': 'url('+ seller.avatar+')'}"></div>
    <div class="header_wrapper">
      <div class="header_img">
        <img :src="seller.avatar">
      </div>
      <div class="header_cont">
        <div class="header_title">
          <i class="brand"></i>
          <span class="name">{{ seller.name }}</span>
        </div>
        <div class="header_peisong">
          {{seller.description}}/{{seller.deliveryTime}}分钟送达
        </div>
        <div class="header_youhui" v-if="seller.supports">
          <i class="decrease"></i>
          <span class="text">{{seller.supports[0].description}}</span>
        </div>
        <div class="support_count" v-if="seller.supports" @click="showDetail">
          <span class="number">{{seller.supports.length}}个</span>
          <i class="icon icon-keyboard_arrow_right"></i>
        </div>
      </div>
    </div>
    
    <!-- 公告条 -->
    <div class="header_notice" @click="showDetail">
      <i class="bulletin"></i>
      <span class="text">{{seller.bulletin}}</span>
      <i class="icon icon-keyboard_arrow_right"></i>
    </div>
    
    <!-- 显示商家详情 -->
    <transition name="fade">
      <div class="detail" v-show="detailShow">
        <div class="detail_content">
          <h1 class="title">{{seller.name}}</h1>
          <div class="star_wrap">
            <star :score="seller.score" :size="48"></star>
          </div>
          <div class="line_title_wrap">
            <line-title title="优惠信息"></line-title>
          </div>
          <ul class="supports" v-if="seller.supports">
            <li class="supports_item" v-for="item in seller.supports">
              <i class="icon" :class="'icon_' + classMap[item.type]"></i>
              <span class="text">{{item.description}}</span>
            </li>
          </ul>
          <div class="line_title_wrap">
            <line-title title="商家公告"></line-title>
          </div>
          <div class="detail_bulletin">{{seller.bulletin}}</div>
          <div class="detail_close_wrap">
            <i class="icon-close" @click="hideDetail"></i>
          </div>
        </div>
      </div>
    </transition>
    
  </div>
</template>

<script>
// 导入外部组件
import star from 'components/star/star';
import lineTitle from 'components/line-title/line-title';

export default {
  data() {
    return {
      // 商家的优惠活动的图标对应关系数组，跟supports数据的type字段一一对应
      classMap: ['decrease', 'discount', 'guarantee', 'invoice', 'special'],
      detailShow: false
    };
  },
  props: {
    // 接受seller属性
    seller: Object
  },
  methods: {
    showDetail() {
      this.detailShow = true;
    },
    hideDetail() {
      this.detailShow = false;
    }
  },
  // 注册外部组件
  components: {
    star,
    lineTitle
  }
};
</script>

<style lang="stylus">
  @import '../../common/stylus/common';
  
  .header
    position: relative
    overflow:hidden
    background: rgba(7,17,27,.5)
    .header_bg
      position: absolute
      top: 0
      left: 0
      z-index: 1
      width: 100%
      height: 100%
      background-size: cover
      background-position: center
      -webkit-filter: blur(10px)
      opacity: 0.5
    .header_wrapper
      position: relative
      z-index: 2
      font-size: 0
      padding: 24px 12px 18px 24px
      .header_img
        inline_block()
        width: 64px
        height: 64px
        border-radius: 2px
        overflow: hidden
        img
          width: 100%
          height: 100%
      .header_cont
        inline_block()
        padding-left: 16px
        color: #fff
        font-weight: bold
        .header_title
          height: 18px
          padding: 2px 0 8px 0
          line-height: 18px
          font-size: 16px
          .brand
            inline_block()
            bg_image('img/brand',30px,18px)
        .header_peisong
          height: 12px
          line-height: 12px
          font-size: 12px
          color: #fff
          font-weight: 200
        .header_youhui
          padding-top: 10px
          .decrease
            margin-right: 4px
            inline_block()
            bg_image('img/decrease_1', 12px, 12px)
          .text
            height: 12px
            line-height: 12px
            font-size: 10px
            font-weight: 200
            color: #fff
        .support_count
          position: absolute
          right: 12px
          bottom: 12px
          padding: 7px 8px
          height: 10px
          line-height: 10px
          font-size: 10px
          font-weight: 200
          border-radius: 18px
          color: #fff
          background: rgba(0,0,0,.2)
          .icon
            margin-left: 2px
            font-size: 10px
    .header_notice
      position: relative
      z-index: 9
      height: 28px
      line-height: 28px
      padding: 0 24px 0 12px
      color: #fff
      background: rgba(7,17,27,.2)
      overflow: hidden
      text-overflow: ellipsis
      white-space: nowrap
      .bulletin
        inline_block()
        bg_image('img/bulletin', 22px, 12px)
        margin: 8px 4px 0 0
      .text
        vertical-align: top
        font-size: 10px
        font-weight: 200
      .icon
        position: absolute
        top: 8px
        right: 12px
        font-size: 12px
    .detail
      position: fixed
      top: 0
      left: 0
      z-index: 250
      width: 100%
      height: 100%
      background: rgba($dark7,.8)
      overflow: auto
      transition: all 0.5s ease
      &.fade-enter
      &.fade-leave-active
        opacity: 0

      .detail_content
        position: relative
        padding: 64px 36px 96px
        // sticky footer layout 关键，内容少时，让关闭按钮在最下部固定，内容超出时，也能正常显示
        min-height: 100%
        box-sizing: border-box
        .title
          text-align: center
          line-height: 16px
          font-size: 16px
          font-weight: 700
          color: #fff
        .star_wrap
          padding: 16px 0 0
          text-align: center
        .line_title_wrap
          padding: 28px 0 24px
        .supports
          padding: 0 12px
          .supports_item
            height: 16px
            line-height: 16px
            padding-bottom: 12px
            font-size: 0
            color: #fff
            &:last-child
              padding-bottom: 0
            .icon
              inline_block()
              width: 16px
              height: 16px
              &.icon_decrease
                bg_image('img/decrease_2', 16px, 16px)
              &.icon_discount
                bg_image('img/discount_2', 16px, 16px)
              &.icon_guarantee
                bg_image('img/guarantee_2', 16px, 16px)
              &.icon_invoice
                bg_image('img/invoice_2', 16px, 16px)
              &.icon_special
                bg_image('img/special_2', 16px, 16px)
            .text
              inline_block()
              padding-left: 6px
              line-height: 16px
              font-size: 12px
              font-weight: 200
        .detail_bulletin
          padding: 0 12px
          line-height: 24px
          font-size: 12px
          font-weight: 200
          color: #fff
        .detail_close_wrap
          position: absolute
          left: 0
          bottom: 32px
          width: 100%
          font-size: 32px
          text-align: center
          color: rgba(255, 255, 255, 0.5)
              
          
</style>