<template>
  <transition name="slide">
    <div v-show="isShow" class="foodDetail">
      <div class="foodDetail_inner">
        <div class="detail_pic" :style="{'background-image': `url(${food.image})`}">
          <div @click="hideFoodDetail" class="detail_backBtn icon-arrow_lift"></div>
        </div>

        <div class="detail_info border_1px">
          <h1 class="food_name">{{ food.name }}</h1>
          <div class="food_info">
            <span>月售{{food.sellCount}}份</span>
            <span>好评率{{food.rating}}%</span>
          </div>
          <div class="food_price">
            <span class="now">￥<b>{{food.price}}</b></span>
            <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol_wrapper">
            <cartcontrol 
              @cartAdd="_drop" 
              :food="food"></cartcontrol>
          </div>
          <div 
            class="buy"
            @click="addFirst" 
            v-show="!food.count || food.count === 0">加入购物车</div>
        </div>

        <div v-show="food.info" class="detail_intro border_1px">
          <h1>商品介绍</h1>
          <p class="detail_intro_p">{{food.info}}</p>
        </div>

        <div v-show="food.ratings && food.ratings.length > 0" class="detail_ratings border_1px">
          <h1>商品评价</h1>

          <div class="rating_types border_1px">
            <div class="rating_type_item rating_all">全部 <span v-show="allRatings">{{allRatings}}</span></div>
            <div class="rating_type_item rating_good">推荐 <span>{{goodRatings}}</span></div>
            <div class="rating_type_item rating_bad">吐槽 <span>{{badRatings}}</span></div>
          </div>

          <div class="rating_showContentOnly border_1px">
            <div 
              class="showContentOnly" 
              :class="{'checked': isShowContentOnly}"
              @click="toggleContentOnly">
              <i class="icon-check_circle"></i><span>只看有内容的评论</span>
            </div>
          </div>

          <ul class="rating_list">
            <li 
              class="rating_item"
              v-for="rating in food.ratings"
              v-show="(isShowContentOnly && rating.text) || !isShowContentOnly">
              <div class="rating_info clearfix">
                <div class="rating_date f_l">{{rating.rateTime | dateFormat}}</div>
                <div class="rating_user f_r"><span class="user_name">{{rating.username}}</span><img class="user_avatar" :src="rating.avatar"></div>
              </div>
              <div class="rating_content">
                <i :class="{'icon-thumb_down': rating.rateType === 1, 'icon-thumb_up': rating.rateType === 0}"></i>
                <p>{{rating.text}}</p>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import BScroll from 'better-scroll';

  export default {
    data() {
      return {
        isShow: false,
        food: {},
        // 是否只显示有内容的评价
        isShowContentOnly: false
      };
    },
    // 注册子组件
    components: {
      cartcontrol
    },

    computed: {
      // 全部评论
      allRatings() {
        if (this.food.ratings) {
          return this.food.ratings.length;
        }
        return false;
      },
      // 推荐评论
      goodRatings() {
        let ratings = this.food.ratings;
        if (!ratings) {
          return 0;
        }
        let good = 0;
        for (let i = 0, len = ratings.length; i < len; i++) {
          let rating = ratings[i];
          if (rating.rateType === 0) {
            good++;
          }
        }
        return good;
      },
      // 吐槽评论
      badRatings() {
        if (this.allRatings) {
          let bad = this.allRatings - this.goodRatings;
          return bad;
        }
        return 0;
      }
    },

    methods: {
      // 处理子组件cartcontrol派发($emit)的 cartAdd 自定义事件，
      // 并继续把 cartAdd 事件，向父级传递，
      // 参数为cartcontrol中，触发cartAdd事件的元素
      _drop(el) {
        this.$emit('cartAdd', el);
      },
      // 在goods组件中要通过ref调用的方法
      _showFoodDetail(food) {
        this.isShow = true;
        this.food = food;
        this.$nextTick(() => {
          if (this.foodScroll) {
            this.foodScroll.refresh();
          } else {
            this.foodScroll = new BScroll('.foodDetail', {
              click: true
            });
          }
        });
      },
      // 隐藏菜品详情页
      hideFoodDetail() {
        this.isShow = false;
        this.isShowContentOnly = false;
      },
      // 点击“立即购买”按钮添加到购物车
      addFirst(event) {
        if (!event._constructed) {
          return;
        }
        this.$set(this.food, 'count', 1);
        // 添加第一次时，为了小球动画正确执行，要确定小球动画的起始位置的元素
        let el = event.target.parentNode.querySelector('.cart_add');
        this.$emit('cartAdd', el);
      },
      // 切换 只看有内容的评论
      toggleContentOnly() {
        this.isShowContentOnly = !this.isShowContentOnly;
        // 有内容显示和隐藏，要记得刷新better-scroll，否则会出现下边的内容滚动不出来，或下边留白的问题
        this.$nextTick(() => {
          this.foodScroll.refresh();
        });
      }
    },
    // 过滤器
    filters: {
      // 格式化日期
      dateFormat(val) {
        let date = new Date(val);
        let year = date.getFullYear();
        let month = date.getMonth() + 1 < 10 ? '0' + date.getMonth() : date.getMonth();
        let day = date.getDate() < 10 ? '0' + date.getDate() : date.getDate();
        let hour = date.getHours() < 10 ? '0' + date.getHours() : date.getHours();
        let min = date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes();
        let str = year + '-' + month + '-' + day + ' ' + hour + ':' + min;
        return str;
      }
    }
  };
</script>

<style lang="stylus" scoped>
  @import '../../common/stylus/common';
  
  .foodDetail
    position: fixed
    top: 0
    left: 0
    bottom: 48px
    z-index: 9
    width: 100%
    overflow: hidden
    background: #f3f5f7
    &.slide-enter,
    &.slide-leave-active
      transform: translate3d(100%, 0, 0)
    &.slide-enter-active,
    &.slide-leave-active
      transition: all 0.3s ease
    .foodDetail_inner
      .detail_pic
        position: relative
        width: 0
        height: 0
        padding: 100% 0 0 100%
        background: #eee no-repeat center center
        background-size: cover
        .detail_backBtn
          position: absolute
          top: 10px
          left: 10px
          padding: 8px
          width: 20px
          height: 20px
          border-radius: 50%
          text-align: center
          line-height: 20px
          background: rgba(7,17,27,.5)
          font-size: 16px
          color: #fff
      .detail_info
        position: relative
        padding: 18px
        background: #fff
        border_1px(rgba(7,17,27,.1))
        .food_name
          font-size: 14px
          line-height: 14px
          color: rgb(7,17,27)
        .food_info
          padding: 8px 0 18px
          font-size: 0
          color: rgb(147,158,159)
          span
            &:first-child
              padding-right: 12px
            line-height: 10px
            font-size: 10px
        .food_price
          font-size: 0
          line-height: 24px
          .now
            font-size: 10px
            color: rgb(240,20,20)
            b
              font-size: 14px
          .old
            padding-left: 12px
            text-decoration: line-through
            font-size: 10px
            color: rgb(147,153,159)
        .cartcontrol_wrapper
          position: absolute
          right: 18px
          bottom: 18px
        .buy
          position: absolute
          right: 18px
          bottom: 21px
          width: 74px
          height: 24px
          line-height: 24px
          text-align: center
          font-size: 12px
          border-radius: 12px
          color: #fff
          background: rgb(0,160,220)
      .detail_intro
        margin-top: 16px
        padding: 18px
        background: #fff
        border_1px(rgba(7,17,27,.1))
        border_top_1px(rgba(7,17,27,.1))
        h1
          font-size: 14px
          font-weight: normal
          color: rgb(7,17,27)
        .detail_intro_p
          padding: 6px 8px 0
          line-height: 24px
          font-size: 12px
          color: rgb(77,85,93)
      .detail_ratings
        margin-top: 16px
        padding: 18px 18px 0
        border_top_1px(rgba(7,17,27,.1))
        background: #fff
        h1
          padding-bottom: 6px
          font-size: 14px
          font-weight: normal
          color: rgb(7,17,27)
        .rating_types
          padding: 12px 0 18px
          font-size: 0
          border_1px(rgba(7,17,27,.1))
          .rating_type_item
            inline_block()
            padding: 8px 12px
            margin-right: 8px
            font-size: 12px
            color: rgb(77,85,93)
            &.rating_all
              background: rgb(0,160,220)
              color: #fff
            &.rating_good
              background: rgba(0,160,220,.2)
            &.rating_bad
              background: rgba(77,85,93,.2)
            span
              font-size: 8px
        .rating_showContentOnly
          margin: 0 -18px
          padding: 12px 18px
          height: 24px
          line-height: 24px
          border_1px(rgba(7,17,27,.1))
          font-size: 12px
          color: rgb(147,153,159)
          .showContentOnly
            display: inline-block
            &.checked
              color: rgb(0,160,220)
            .icon-check_circle
              height: 24px
              font-size: 16px
              vertical-align: middle
            span
              vertical-align: middle
        .rating_list
          .rating_item
            padding: 16px 0
            border_1px(rgba(7,17,27,.2))
            &:last-child
              border_none()
            .rating_info
              color: rgb(147,153,159)
              .rating_date
                line-height: 12px
                font-size: 10px
              .rating_user
                font-size: 0
                text-align: right
                line-height: 18px
                .user_name
                  inline_block()
                  margin-right: 6px
                  font-size: 10px
                .user_avatar
                  width: 18px
                  height: 18px
                  border-radius: 50%
            .rating_content
              font-size: 12px
              color: rgb(7,17,27)
              i
                font-size: 12px
                line-height: 24px
                &.icon-thumb_down
                  color: rgb(147,153,159)
                &.icon-thumb_up
                  color: rgb(0,160,220)
              p
                display: inline-block
                  
                
              
        

          
</style>