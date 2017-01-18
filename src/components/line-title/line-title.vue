<template>
  <div class="title_line">
    <div class="line"></div>
    <div class="text">{{title}}</div>
    <div class="line"></div>
  </div>  
</template>

<script>
  export default {
    props: {
      title: {
        type: String
      }
    }
  };
</script>

<style lang="stylus" scoped>
  .title_line
    display: flex
    .text
      padding: 0 12px
      line-height: 15px
      font-size: 15px
      font-weight: 700
      color: #fff
    .line
      flex: 1
      margin-top: 7px
      border-top: 1px solid rgba(255, 255, 255, .2)
</style>