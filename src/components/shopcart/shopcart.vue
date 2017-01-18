<template>
  <div class="shopcart">
    <div class="content" @click="showList">
      <div class="cartIcon_wrapper">
        <div class="cartIcon" :class="{'hasSelect': totalCount > 0}">
          <div v-show="totalCount > 0" class="cartNum">{{totalCount}}</div>
          <i class="icon-shopping_cart"></i>
        </div>
      </div>

      <div class="total" :class="{'hasSelect': totalPrice > 0}">￥{{totalPrice}}</div>
      <div class="tips">另需配送费￥{{deliveryPrice}}元</div>
      <div class="startSend" :class="{'pay': totalPrice > minPrice}">{{payDesc}}</div>

      <div class="ball_container">
        <transition 
          name="drop"
          v-for="ball in balls"
          @before-enter="dropBeforeEnter"
          @enter="dropEnter"
          @after-enter="dropAfterEnter">
          <div class="ball" v-show="ball.show">
            <div class="ball_inner"></div>
          </div>
        </transition>
      </div>
    </div>
    
    <transition name="slide">
      <div class="list" v-show="listShow">
        <div class="list_header">
          <span>购物车</span>
          <div @click="emptyCart" class="clearBtn">清空</div>
        </div>
        <div class="list_content">
          <ul class="list_wrapper">
            <li v-for="food in selectFoods" class="list_item">
              <div class="item_name">{{food.name}}</div>
              <div class="item_price">￥<span class="num">{{food.count * food.price}}</span></div>
              <div class="cartcontrol_wrapper">
                <cartcontrol :food="food"></cartcontrol>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </transition>
    
    <transition name="fade">
      <div @click="hideList" class="shopcart_bg" v-show="listShow"></div>
    </transition>
  </div>
</template>

<script>
  import BScroll from 'better-scroll';
  import cartcontrol from 'components/cartcontrol/cartcontrol';

  export default {
    props: {
      // 添加到购物车中的菜品，数组
      selectFoods: {
        type: Array,
        default() {
          return [];
        }
      },
      // 配送费
      deliveryPrice: {
        type: Number,
        default: 0
      },
      // 起送价格
      minPrice: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        // 是否显示购物车的列表
        listShow: false,
        // 加入购物车的小球动画
        balls: [
          {show: false},
          {show: false},
          {show: false},
          {show: false},
          {show: false}
        ]
      };
    },
    computed: {
      // 购物车中的总数
      totalCount() {
        let count = 0;
        this.selectFoods.forEach((food) => {
          count += food.count;
        });
        return count;
      },
      // 购物车中的总价
      totalPrice() {
        let price = 0;
        this.selectFoods.forEach((food) => {
          price += food.count * food.price;
        });
        return price;
      },
      // 最右侧按钮，根据总价不同，显示不同
      payDesc() {
        let desc = '';
        if (this.totalPrice === 0) {
          desc = `￥${this.minPrice}起送`;
        }
        if (this.totalPrice > 0 && this.totalPrice < this.minPrice) {
          desc = `还差￥${this.minPrice - this.totalPrice}配送`;
        }
        if (this.totalPrice >= this.minPrice) {
          desc = '去结算';
        }
        return desc;
      }
    },
    methods: {
      // 显示购物车列表
      showList() {
        this.listShow = true;

        this.$nextTick(() => {
          if (!this.cartScroll) {
            this.cartScroll = new BScroll('.list_content', {
              click: true
            });
          } else {
            this.cartScroll.refresh();
          }
        });
      },
      // 隐藏购物车列表
      hideList() {
        this.listShow = false;
      },
      // 清空购物车
      emptyCart() {
        // 不能简单粗暴的直接将selectFoods设为空数组
        // this.selectFoods = [];
        // 要遍历selectFoods把每个food的count设为0
        this.selectFoods.forEach((food) => {
          food.count = 0;
        });
      },

      // 改变数据模型中的balls，来触发小球的drop动画
      drop(el) {
        // console.log(el.getBoundingClientRect());
        for (let i = 0, length = this.balls.length; i < length; i++) {
          let ball = this.balls[i];
          if (!ball.show) {
            ball.show = true;
            ball.el = el;
            return;
          }
        }
      },
      // 小球动画开始之前
      dropBeforeEnter(el) {
        this.$nextTick(() => {
          for (let i = 0, len = this.balls.length; i < len; i++) {
            let ball = this.balls[i];
            if (ball.show) {
              let rect = ball.el.getBoundingClientRect();
              let x = (rect.left - 53) + 'px';
              let y = -(rect.bottom - 42) + 'px';
              let inner = el.querySelector('.ball_inner');
              el.style.transform = `translate3d(0, ${y}, 0)`;
              el.style.webkitTransform = `translate3d(0, ${y}, 0)`;
              el.style.transition = 'all 4s cubic-bezier(.42,-0.36,.85,.57)';
              el.style.webkitTransition = 'all 4s cubic-bezier(.42,-0.36,.85,.57)';
              inner.style.transform = `translate3d(${x}, 0, 0)`;
              inner.style.webkitTransform = `translate3d(${x}, 0, 0)`;
              inner.style.transition = 'all 4s linear';
              inner.style.webkitTransition = 'all 4s linear';
            }
          }
        });
      },

      // 小球动画开始
      dropEnter(el) {
        this.$nextTick(() => {
          el.style.transform = 'translate3d(0,0,0)';
          el.style.webkitTransform = 'translate3d(0,0,0)';
          let inner = el.querySelector('.ball_inner');
          inner.style.transform = 'translate3d(0, 0, 0)';
          inner.style.webkitTransform = 'translate3d(0, 0, 0)';
        });
      },
      // 小球动画结束后
      dropAfterEnter(el) {
        for (let i = 0, length = this.balls.length; i < length; i++) {
          let ball = this.balls[i];
          if (ball.show) {
            ball.show = false;
            ball.el.style.transform = 'translate3d(0, 0, 0)';
            ball.el.style.webkitTransform = 'translate3d(0, 0, 0)';
            let inner = el.querySelector('.ball_inner');
            inner.style.transform = 'translate3d(0, 0, 0)';
            inner.style.webkitTransform = 'translate3d(0, 0, 0)';
            return;
          }
        }
      }
    },
    components: {
      cartcontrol
    }
  };
</script>

<style lang="stylus" scoped>
  @import '../../common/stylus/common';
  .shopcart
    position: fixed
    left: 0
    bottom: 0
    z-index: 60
    width: 100%
    height: 48px
    background: #141427
    .content
      position: relative
      z-index: 102
      display: flex
      .cartIcon_wrapper
        width: 58px
        height: 58px
        padding: 6px
        margin: -10px 0 0 12px
        border-radius: 50%
        box-sizing: border-box
        background: #141427
        .cartIcon
          position: relative
          width: 46px
          height: 46px
          border-radius: 50%
          line-height: 46px
          text-align: center
          background: #141d27
          .icon-shopping_cart
            font-size: 24px
            line-height: 24px
            color: rgba(255, 255, 255, .4)
          &.hasSelect
            background: rgb(0,160,220)
            .icon-shopping_cart
              color: #fff
        .cartNum
          position: absolute
          top: -6px
          right: -6px
          padding: 0 6px
          height: 18px
          line-height: 18px
          font-size: 9px
          border-radius: 12px
          background: rgb(240, 20, 20)
          box-shadow: 0 4px 8px 0 rgba(0,0,0,.2)
          color: #fff
      .total
        height: 24px
        margin: 12px 0 12px 12px
        padding-right: 12px
        border-right: 1px solid rgba(255, 255, 255, .1)
        line-height: 24px
        font-size: 16px
        color: rgba(255,255,255,.4)
        &.hasSelect
          color: #fff
      .tips
        flex: 1
        height: 48px
        line-height: 48px
        padding-left: 12px
        font-size: 14px
        color: rgba(255,255,255,.4)
      .startSend
        width: 105px
        height: 48px
        line-height: 48px
        font-size: 14px
        text-align: center
        color: rgba(255,255,255,.4)
        background: #141d27
        &.pay
          background: #00b43c
          color: #fff
      .ball_container
        .ball
          position: fixed
          left: 53px
          bottom: 42px
          .ball_inner
            width: 16px
            height: 16px
            border-radius: 50%
            background: rgb(240, 20, 20)
    .list
      position: fixed
      left: 0
      bottom: 48px
      z-index: 101
      width: 100%
      &.slide-enter,
      &.slide-leave-active
        opacity: 0
        transform: translate3d(0, -100%, 0)
      &.slide-enter-active,
      &.slide-leave-active
        transition: all 0.3s ease
      .list_header
        position: relative
        height: 40px
        padding: 0 18px
        line-height: 40px
        font-size: 14px
        font-weight: 200
        color: rgb(7, 17, 27)
        background: #f3f5f7
        border_1px(rgba(7,17,27,.1))
        .clearBtn
          position: absolute
          top: 5px
          right: 13px
          height: 30px
          padding: 0 5px
          line-height: 30px
          font-size: 12px
          color: rgb(0, 160, 220)
      .list_content
        padding: 0 18px
        background: #fff
        max-height: 260px
        overflow: hidden
        .list_wrapper
          .list_item
            display: flex
            height: 48px
            line-height: 48px
            border_1px(rgba(7,17,27,.1))
            &:last-child
              border_none()
            .item_name
              flex: 1
            .item_price
              padding: 0 12px 0 18px
              font-size: 10px
              color: rgb(240, 20, 20)
              .num
                font-size: 14px
                font-weight: 700
            .cartcontrol_wrapper
              width: 90px
              padding-top: 10px
    .shopcart_bg
      position: fixed
      top: 0
      left: 0
      z-index: 100
      width: 100%
      height: 100%
      background: rgba(7,17,27,.8)
      &.fade-enter,
      &.fade-leave-active
        opacity: 0
      &.fade-enter-active,
      &.fade-leave-active
        transition: all 0.3s ease
</style>