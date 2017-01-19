<template>
  <div class="shopcart">
    <div class="content">
      <div class="cartIcon_wrapper">
        <div  @click="toggleList" class="cartIcon" :class="{'hasSelect': totalCount > 0}">
          <div v-show="totalCount > 0" class="cartNum">{{totalCount}}</div>
          <i class="icon-shopping_cart"></i>
        </div>
      </div>

      <div class="total" :class="{'hasSelect': totalPrice > 0}">￥{{totalPrice}}</div>
      <div class="tips">另需配送费￥{{deliveryPrice}}元</div>
      <div @click="pay" class="startSend" :class="{'pay': totalPrice > minPrice}">{{payDesc}}</div>

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
                <cartcontrol 
                  @cartAdd="drop"
                  :food="food"></cartcontrol>
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
        isShow: false,
        // 加入购物车的小球动画
        balls: [
          {show: false},
          {show: false},
          {show: false},
          {show: false},
          {show: false}
        ],
        // 执行完动画的小球，临时存储在这
        dropBalls: []
      };
    },
    computed: {
      // listShow用计算属性，而不用方法的原因在于清空购物车的时候，
      // 计算属性会自动清空所有，并隐藏购物车列表，如果在方法中则需手动调用hideList方法
      listShow() {
        if (!this.totalCount) {
          this.isShow = false;
          return false;
        }
        if (this.isShow) {
          this.$nextTick(() => {
            if (!this.cartScroll) {
              this.cartScroll = new BScroll('.list_content', {
                click: true
              });
            } else {
              this.cartScroll.refresh();
            }
          });
        }
        return this.isShow;
      },
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
      // 隐藏购物车列表
      hideList() {
        this.isShow = false;
      },
      // 切换购物车列表的显示/隐藏
      toggleList() {
        if (this.totalCount) {
          this.isShow = !this.isShow;
        }
      },
      // 模拟结算
      pay() {
        if (this.totalPrice >= this.minPrice) {
          window.alert(`已支付${this.totalPrice}元`);
        }
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
      // 这个方法还在goods组件中，被_drop方法间接调用
      drop(el) {
        for (let i = 0, length = this.balls.length; i < length; i++) {
          let ball = this.balls[i];
          if (!ball.show) {
            ball.show = true;
            // 把当前点击的元素，设置成ball的属性，这样在dropBeforeEnter钩子函数里，
            // 就能通过ball.el来确定小球的起始位置了
            ball.el = el;
            // 把开始运动的小球，放在dropBalls里，以便在运动结束之后隐藏该小球
            // 是为了能确定哪个小球，防止点击过快的话不能确定哪个小球是执行动画的
            this.dropBalls.push(ball);
            return;
          }
        }
      },
      // 小球动画开始之前
      dropBeforeEnter(el) {
        // 小球运动之前，先确定开始的位置
        for (let i = 0, len = this.balls.length; i < len; i++) {
          let ball = this.balls[i];
          if (ball.show) {
            let rect = ball.el.getBoundingClientRect();
            let x = (rect.left - 46) + 'px';
            let y = -(window.innerHeight - rect.top - 65) + 'px';
            let inner = el.querySelector('.ball_inner');
            el.style.transform = `translate3d(0, ${y}, 0)`;
            el.style.webkitTransform = `translate3d(0, ${y}, 0)`;
            inner.style.transform = `translate3d(${x}, 0, 0)`;
            inner.style.webkitTransform = `translate3d(${x}, 0, 0)`;
          }
        }
      },

      // 小球动画开始
      dropEnter(el) {
        // 用el.offsetHeight来触发浏览器重绘，这样就能正确的计算出位置了
        /* eslint-disable no-unused-vars */
        let rf = el.offsetHeight;
        // 开始运动的时候，就设置相应的transition和transfrom
        this.$nextTick(() => {
          el.style.transform = 'translate3d(0,0,0)';
          el.style.webkitTransform = 'translate3d(0,0,0)';
          el.style.transition = 'all .4s cubic-bezier(.32,-0.59,.85,.57)';
          el.style.webkitTransition = 'all .4s cubic-bezier(.32,-0.59,.85,.57)';
          let inner = el.querySelector('.ball_inner');
          inner.style.transform = 'translate3d(0, 0, 0)';
          inner.style.webkitTransform = 'translate3d(0, 0, 0)';
          inner.style.transition = 'all .4s linear';
          inner.style.webkitTransition = 'all .4s linear';
        });
      },
      // 小球动画结束后
      dropAfterEnter(el) {
        // 在dropBalls里，要以队列的形式来使运动完成的小球隐藏
        // 因为开始运动之前，是把小球push到dropBalls的最后一个
        // 以这种队列的形式，才不至于因为点击过快而露掉哪个小球
        let ball = this.dropBalls.shift();
        if (ball) {
          ball.show = false;
        }
        // 运动结束后，要把transition设置为空，否则会影响该小球下次出现时候的动画
        el.style.transition = '';
        el.style.webkitTransition = '';
        let inner = el.querySelector('.ball_inner');
        inner.style.transition = '';
        inner.style.webkitTransition = '';
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
      transform-origin: left bottom
      &.slide-enter,
      &.slide-leave-active
        opacity: 0
        // transform: translate3d(0, 100%, 0)
        transform: scaleY(0)
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