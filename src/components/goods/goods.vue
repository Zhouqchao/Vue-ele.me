 <template>
  <div>
    <div class="goods">
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li v-for="(item, index) in goods" :key="index" class="menu-item"
              :class="{'current':currentIndex === index }"
              @click="selectMenu(index,$event)" ref="menuList">
            <span class="text border-1px">
              <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
              {{item.name}}
            </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodsWrapper">
        <ul>
          <li v-for="(item,index) in goods" class="food-list" :key="index" ref="foodList">
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li v-for="(food, index) in item.foods" :key="index" @click="selectFood(food,$event)" class="food-item">
                <div class="icon">
                  <img width="57" height="57" :src="food.icon">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{food.price}}</span>
                    <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <app-cart-control @add="addFood" :food="food"></app-cart-control>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <app-shop-cart ref="shopcart" :selectFoods="selectFoods" :deliveryPrice="seller.deliveryPrice" :minRrice="seller.minPrice"></app-shop-cart>
    </div>
    <app-food :food="selectedFood" @add="addFood" ref="food"></app-food>   
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import ShopCart from 'components/shopcart/shopcart';
  import CartControl from 'components/cartcontrol/cartcontrol';
  import Food from 'components/food/food';

  export default {
    props:{
      seller:{
        type:Object
      }
    },
    data(){
      return {
        goods:[],
        listHeight:[],
        scrollY:0,
        selectedFood:{}
      };
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i;
          }
        }
        return 0;
      },
      selectFoods(){
        let foods = [];
        this.goods.forEach((good)=>{
          good.foods.forEach((food)=>{
            if(food.count){
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    created(){
      this.classMap = ['decrease','discount','special','invoice','guarantee'];
      this.$http.get('https://gist.githubusercontent.com/Zhouqchao/64bfb02f304a0d6075c171285569e54d/raw/4590f3e79dccfc78caf740e1c1507426f7de84f0/vue-sell-data.json').then((res)=>{
        console.clear();
        res = res.body.goods;
        this.goods = res;
        this.$nextTick(() => {
          this._initScroll();
          this._calculateHeight();
        });
      });
    },
    components:{
      appShopCart: ShopCart,
      appCartControl: CartControl,
      appFood: Food
    },
    methods:{
      _drop(target){
        this.$nextTick(()=>{
          this.$refs.shopcart.drop(target);
        });
      },
      selectMenu(index,event){
        if(!event._constructed){
          return;
        }
        let foodList = this.$refs.foodList;
        let el = foodList[index];
        this.foodsScroll.scrollToElement(el,300);
      },
      _initScroll(){
       this.menuScroll = new BScroll(this.$refs.menuWrapper,{
         click:true
       });
       this.foodsScroll = new BScroll(this.$refs.foodsWrapper,{
         click:true,
         probeType:3
       });
       this.foodsScroll.on('scroll',(pos)=>{
         this.scrollY = Math.abs(Math.round(pos.y));
       });
      },
      _calculateHeight(){
        let foodList = this.$refs.foodList;
        let height = 0;
        this.listHeight.push(height);
        for(let i = 0; i < foodList.length; i++){
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      },
      selectFood(food,e){
        if(!e._constructed){
          return;
        }
        this.selectedFood = food;
        this.$refs.food.show();
      },
      addFood(target) {
        this._drop(target);
      }
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin";

  .goods
    display: flex
    position:absolute
    top: 174px
    bottom: 46px
    width: 100%
    overflow:hidden
    .menu-wrapper
      flex:0 0 80px
      width: 80px
      background: #cccccc
      .menu-item
        display:table
        height: 54px
        width: 56px
        padding:0 12px
        line-height: 14px
        cursor: pointer
        &.current
          position:relative
          margin-top:-1px
          z-index: 10
          background: #ffffff
          font-weight: 700
          .text
            border:none
        .icon
          display:inline-block
          vertical-align top
          width: 12px
          height: 12px
          margin-right 4px
          background-size:12px 12px
          background-repeat:no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display:table-cell
          width: 56px
          vertical-align:middle
          font-size:12px
          border-1px(rgba(7,17,27,0.2))
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height: 26px
        line-height: 26px
        border-left:2px solid #d9dde1
        font-size: 12px
        color:#93999f
        background: #f3f5f7
      .food-item
        display:flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7,17,27,0.1))
        cursor: pointer
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex:0 0 57px
          margin-right: 10px
        .content
          flex:1
          .name
            margin:2px 0 8px 0
            height: 14px
            line-height: 14px
            font-size: 14px
            color:rgb(7,17,27)
          .desc,.extra
            line-height: 10px
            font-size: 10px
            color: #93999f
          .desc
            margin-bottom: 8px
            line-height:12px
          .extra
            .count
              margin-right: 12px
          .price
            line-height:24px
            font-weight:700
            .now
              margin-right: 8px
              font-size: 14px
              color:rgb(240,20,20)
            .old
              text-decoration:line-through
              font-size: 10px
              color: #93999f
          .cartcontrol-wrapper
            position:absolute
            right: 0
            bottom:12px

</style>
