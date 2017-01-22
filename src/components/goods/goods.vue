<template>
  <div class="goods">
    <!-- 左侧菜单 -->
    <div class="goods_left">
      <ul>
        <li @click="selectMenu($event,index)" v-for="(item,index) in goods" class="goods_left_item" :class="{'current': currentIndex===index}">
          <div class="goods_left_item_inner">
            <i class="icon" v-if="item.type > -1" :class="classMap[item.type]"></i><span>{{item.name}}</span>
          </div>
        </li>
      </ul>
    </div>
    <!-- 右侧列表 -->
    <div class="goods_right">
      <!-- better-scroll 只会滚动容器的第一个子元素，所以容器里的所有内容要放在一个包裹元素里 -->
      <div class="foods_scroll_wrapper">
        <div v-for="item in goods" class="goods_right_section">
          <div class="section_title">{{item.name}}</div>
          <ul>
            <li v-for="food in item.foods" class="section_list">
              <div @click="showFoodDetail($event, food)" class="section_list_inner">
                <div class="section_list_icon" :style="{'background-image': 'url('+ food.icon +')'}"></div>
                <div class="section_list_content">
                  <h3 class="list_name">{{food.name}}</h3>
                  <p v-show="food.description"
                    class="list_desc">{{food.description}}</p>
                  <div class="list_extra">
                    <span class="sell">月售{{food.sellCount}}份</span><span class="rating">好评率{{food.rating}}%</span>
                  </div>
                  <div class="list_price">
                    <span class="now">￥<b>{{food.price}}</b></span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                  </div>

                </div>
              </div>
              <div class="cartcontrol_wrapper">
                <cartcontrol 
                  :food="food"
                  @cartAdd="_drop"></cartcontrol>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- 购物车 -->
    <!-- 在goods组件中，ref='shopcart' 来引用子组件shopcart，这样，就可以在goods组件的方法中通过this.$refs.shopcart.drop()来调用shopcart组件的drop方法了-->
    <shopcart
      ref="shopcart"
      :select-foods="selectFoods"
      :deliveryPrice="seller.deliveryPrice"
      :minPrice="seller.minPrice"></shopcart>

    <!-- 菜品详情 -->
    <!-- 监听fooddetail组件派发($emit)的cartAdd事件 -->
    <fooddetail
      @cartAdd="_drop"
      ref="foodDetail"></fooddetail>
  </div>
</template>

<script>
  import BScroll from 'better-scroll';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import shopcart from 'components/shopcart/shopcart';
  import fooddetail from 'components/foodDetail/foodDetail';

  export default {
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        // 这里一定要是一个空数组，不能是null，否则因为在计算属性selectFoods中循环会报错
        goods: [],
        // foodScroll的scrollY
        scrollY: 0,
        // goods_right_section的高度集合
        heightList: [],
        // goods_right_section的dom集合
        foodSections: null,
        // 左侧菜单列表中小图标样式
        classMap: ['decrease', 'discount', 'guarantee', 'invoice', 'special'],
        selectFoods: []
      };
    },
    // 计算属性
    computed: {
      // 左侧菜单当前选中第几个，默认第一个
      currentIndex() {
        for (let i = 0, len = this.heightList.length; i < len; i++) {
          let posY1 = this.heightList[i];
          let posY2 = this.heightList[i + 1];
          // 比较当前goods_right_section的clientY，跟下一个goods_right_section的clientY，来判断crrentIndex
          if (!posY2 || this.scrollY >= posY1 && this.scrollY < posY2) {
            return i;
          }
        }
        return 0;
      },
      // 被添加到购物车中的菜品，用在购物车组件中
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    created() {
      this.$http.get('/api/goods').then((res) => {
        this.goods = res.body.data;
        console.log(this.goods);

        // 在$nextTick()里调用BScroll的初始方法
        this.$nextTick(() => {
          this._initScroll();
          this._calculateHeight();
        });
      });
    },
    // 用到的方法集合
    methods: {
      // 初始化BScroll
      _initScroll() {
        this.menuScroll = new BScroll('.goods_left', {
          click: true
        });

        this.foodScroll = new BScroll('.goods_right', {
          click: true,
          probeType: 3
        });

        // 滚动的时候，实时改变scrollY的值，供计算属性crrentIndex使用
        this.foodScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(pos.y);
        });
      },

      // 计算右侧食物列表（goods_right_section）的高度集合
      _calculateHeight() {
        // this.$el指代当前组件的根节点元素,获取所有的食物分类的dom元素
        this.foodSections = this.$el.querySelectorAll('.goods_right_section');
        // 用来保存每个食物分类dom元素的高度
        let height = 0;
        // 先把第一个goods_right_section的clientY为0保存起来
        this.heightList.push(height);

        for (let i = 0, len = this.foodSections.length; i < len; i++) {
          let section = this.foodSections[i];
          // 计算得出当前goods_right_section的clientY的值，
          height += section.clientHeight;
          this.heightList.push(height);
        }
      },

      // 处理子组件 cartcontrol 触发的 cartAdd 自定义事件
      _drop(el) {
        this.$nextTick(() => {
          this.$refs.shopcart.drop(el);
        });
      },

      // 选中左侧菜单列表
      selectMenu(e, index) {
        this.foodScroll.scrollToElement(this.foodSections[index], 300);
      },
      // 显示选中的菜品的详情
      showFoodDetail(event, food) {
        if (!event._constructed) {
          return;
        }
        this.$refs.foodDetail._showFoodDetail(food);
      }
    },

    // 注册引入的组件
    components: {
      cartcontrol,
      shopcart,
      fooddetail
    }
  };
</script>

<style lang="stylus" scoped>
  @import '../../common/stylus/common';
  
  .goods
    position: absolute
    top: 174px
    bottom: 48px
    left: 0
    width: 100%
    display: flex
    .goods_left
      width: 80px
      height: 100%
      overflow: hidden
      background: #f3f5f7
      .goods_left_item
        display: table
        padding: 0 12px
        width: 56px
        &.current
          background: #fff
          .goods_left_item_inner
            border_none()
        &:last-child
          .goods_left_item_inner
            border_none()
        .goods_left_item_inner
          // display：table，display:table-cell结合能让内容文字垂直居中
          display: table-cell
          height: 54px
          vertical-align: middle
          border_1px(rgba(7,17,27,.1))
          font-size: 12px
          font-weight: 200
          line-height: 14px
          color: rgb(77,85,93)
          .icon
            inline_block()
            margin-right: 2px
            &.decrease
              bg_image('img/decrease_3', 12px, 12px)
            &.discount
              bg_image('img/discount_3', 12px, 12px)
            &.guarantee
              bg_image('img/guarantee_3', 12px, 12px)
            &.invoice
              bg_image('img/invoice_3', 12px, 12px)
            &.special
              bg_image('img/special_3', 12px, 12px)
    .goods_right
      flex: 1
      height: 100%
      overflow: hidden
      .goods_right_section
        .section_title
          height: 26px
          padding-left: 13px
          border-left: 2px solid #d9dde1
          line-height: 26px
          font-size: 12px
          background: #f3f5f7
          color: rgb(147,153,159)
        .section_list
          position: relative
          padding: 18px 18px 0
          .section_list_inner
            display: flex
            padding-bottom: 18px
            border_1px(rgba(7,17,27,.1))
            .section_list_icon
              width: 57px
              height: 57px
              margin-right: 10px
              background-size: cover
              background-repeat: no-repeat
              background-color: #eee
              border-radius: 2px
            .section_list_content
              flex: 1
              overflow: hidden
              .list_name
                padding-top: 2px
                height: 14px
                line-height: 14px
                font-size: 14px
                color: rgb(7,17,27)
              .list_desc,
              .list_extra
                height: 10px
                font-size: 10px
                line-height: 10px
                padding-top: 8px
                text_ellipsis()
                color: rgb(147,153,159)
                
              .list_extra
                .sell
                  padding-right: 12px
              .list_price
                font-size: 10px
                line-height: 24px
                .now
                  padding-right: 8px
                  color: rgb(240, 20, 20)
                  b
                    font-size: 14px
                .old
                  color: rgb(147,153,159)
                  text-decoration: line-through
          &:last-child
            .section_list_inner
              border_none()
          .cartcontrol_wrapper
            position: absolute
            right: 12px
            bottom: 12px
          
</style>