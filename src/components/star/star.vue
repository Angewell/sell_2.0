<template>
  <div class="star" :class="starType">
    <span v-for="itemClass in itemClasses" class="star_item" :class="itemClass"></span>
  </div>
</template>

<script>
  // 星星组件中，星星的总个数，以及各种状态星星的样式
  const LENGTH = 5;
  const CLS_ON = 'on';
  const CLS_OFF = 'off';
  const CLS_HALF = 'half';

  export default {
    props: {
      // 星星组件的尺寸，48px、36px、24px之一
      size: {
        type: Number
      },
      // 评分，星星组件根据这个来显示几星
      score: {
        type: Number
      }
    },
    computed: {
      // 星星组件的类型
      starType() {
        return 'star_' + this.size;
      },
      // 定义每个星星的样式的数组
      itemClasses() {
        // 要返回的数组
        let result = [];
        // 评分,经过计算，小数部分不足0.5的部分会被舍去，等于0.5的保留，大于0.5的进一位
        let score = Math.floor(this.score * 2) / 2;
        // 看计算后的评分中，是否含有小数
        let hasDecimal = score % 1 !== 0;
        // 计算得分的整数部分
        let integer = Math.floor(score);

        // 遍历得分的整数部分，并把对应的星星的样式push到result中
        for (let i = 0; i < integer; i++) {
          result.push(CLS_ON);
        }

        // 如果评分中有小数部分（即半颗星状态）
        if (hasDecimal) {
          result.push(CLS_HALF);
        }

        // 如果result的数量不够五个，剩下的都添加off状态
        while (result.length < LENGTH) {
          result.push(CLS_OFF);
        }

        // 最后，一定要记得返回result
        return result;
      }
    }
  };
</script>

<style lang="stylus">
  @import '../../common/stylus/common';
  .star
    font-size: 0
    .star_item
      display: inline-block
      &:last-child
        margin-right: 0!important;
    &.star_48
      .star_item
        margin-right: 22px
        &.on
          bg_image('./img/star48_on', 20px, 20px)
        &.off
          bg_image('./img/star48_off', 20px, 20px)
        &.half
          bg_image('./img/star48_half', 20px, 20px)
    &.star_36
      .star_item
        margin-right: 16px
        &.on
          bg_image('./img/star48_on', 15px, 15px)
        &.off
          bg_image('./img/star48_off', 15px, 15px)
        &.half
          bg_image('./img/star48_half', 15px, 15px)
    &.star_24
      .star_item
        margin-right: 3px
        &.on
          bg_image('./img/star48_on', 10px, 10px)
        &.off
          bg_image('./img/star48_off', 10px, 10px)
        &.half
          bg_image('./img/star48_half', 10px, 10px)
</style>
