<template>
  <div class="cartcontrol">
    <transition name="roll">
      <div @click="decreaseCount" v-show="this.food.count > 0" class="cart_decrease icon-remove_circle_outline">
      </div>
    </transition>
    <div v-show="this.food.count > 0" class="cart_count">{{food.count}}</div>
    <div @click="addCount" class="cart_add icon-add_circle"></div>
  </div>
</template>
<script>
  export default {
    props: {
      food: {
        type: Object
      }
    },
    methods: {
       // 添加数量
      addCount(event) {
        // event在pc端触发两次，一次是有原生支持的事件，一次是better-scroll产生的
        // event._constructed 为 better-scroll 给事件对象添加的属性，如果检测到该属性，
        // 说明此次事件是由better-scroll产生的，在移动端和pc端都会触发，这是我们需要用的事件，
        // 而在移动端，better-scroll不触发“click”事件，不会达到我们预期的效果，所以，这里需要用
        // 移动端和pc端都能触发的 better-scroll 的事件，即如果没有 event._constructed的时候，
        // 说明该事件只在pc端触发，在移动端不会触发，则直接ruturn
        if (!event._constructed) {
          return;
        }
        // 如果没有数量，则用实例方法vm.$set给food对象添加count属性，并设置为1
        if (!this.food.count) {
          this.$set(this.food, 'count', 1);
        } else {
          this.food.count++;
        }

        // 触发 cartAdd 自定义事件，引用 cartcontrol 组件的父组件可以直接“监听” cartAdd 事件
        // 并把事件对象的target作为参数传入
        // 用来做加入购物车小球动画的
        this.$emit('cartAdd', event.target);
      },
      // 减少数量
      decreaseCount(event) {
        // event在pc端触发两次，一次是有原生支持的事件，一次是better-scroll产生的
        // event._constructed 为 better-scroll 给事件对象添加的属性，如果检测到该属性，
        // 说明此次事件是由 better-scroll 产生的，在移动端和pc端都会触发，这是我们需要用的事件，
        // 而在移动端，better-scroll不触发“click”事件，不会达到我们预期的效果，所以，这里需要用
        // 移动端和pc端都能触发的 better-scroll 的事件，即如果没有 event._constructed的时候，
        // 说明该事件只在pc端触发，在移动端不会触发，则直接ruturn
        if (!event._constructed) {
          return;
        }
        this.food.count && this.food.count--;
      }
    }
  };
</script>
<style lang="stylus" scoped>
  @import '../../common/stylus/common';
  .cartcontrol
    .cart_decrease,
    .cart_add
      inline_block()
      width: 24px
      height: 24px
      padding: 3px
      font-size: 24px
      line-height: 24px
      color: rgb(0, 160, 220)
    .cart_decrease
      &.roll-enter-active,
      &.roll-leave-active
        transition: all .3s linear
      &.roll-enter,
      &.roll-leave-active
        transform: translate3d(40px, 0, 0) rotate(180deg)
        opacity: 0
    .cart_count
      inline_block()
      width: 15px
      height: 24px
      padding: 3px 0
      text-align: center
      line-height: 24px
      font-size: 10px
      color: rgb(147, 153, 159)
</style>