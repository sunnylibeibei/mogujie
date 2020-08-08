<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
    <scroll class='content' 
            ref="scroll" 
            :probe-type="3"
            @scroll="contentScroll">
    <detail-swiper :top-images="topImages"/>
    <detail-base-info :goods="goods"/>
    <detail-shop-info :shop="shop"/>
    <detail-goods-info ref="params" :detail-info="detailInfo" @detailImageLoad="detailImageLoad"/>
    <detail-param-info ref="comment" :param-info="paramInfo"/>
    <detail-comment-info :comment-info="commentInfo"/>
    <goods-list ref="recommend" :goods="recommends"/>
    </scroll>
    <detail-bottom-bar/>
  </div>
</template>

<script>
import DetailNavBar from "./childComps/DetailNavBar"
import DetailSwiper from "./childComps/DetailSwiper"
import DetailBaseInfo from "./childComps/DetailBaseInfo"
import DetailShopInfo from "./childComps/DetailShopInfo"
import DetailGoodsInfo from "./childComps/DetailGoodsInfo"
import DetailParamInfo from "./childComps/DetailParamInfo"
import DetailCommentInfo from "./childComps/DetailCommentInfo"
import DetailBottomBar from "./childComps/DetailBottomBar"


import Scroll from "components/common/scroll/Scroll"
import GoodsList from 'components/content/goods/GoodsList'

import {getDetail,Goods,Shop,GoodsParam,getRecommend} from "network/detail"
import {debounce} from "common/utils"
import {itemListenerMixin} from "common/mixin"

export default {
  name:"Detail",
  components:{
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    Scroll,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    DetailBottomBar,
    GoodsList
  },
  mixins:[itemListenerMixin],
  data(){
    return {
      iid:null,
      topImages:[],
      goods:{},
      shop:{},
      detailInfo:{},
      paramInfo:{},
      commentInfo:{},
      recommends:[],
      themeTopYs:[],
      getThemeTopY:null,
      currentIndex:0
    }
  },
  created(){
    // cosnole.log(this.$route.params.iid)
    // 获取iid
    this.iid=this.$route.params.iid
    // 请求详情数据
    getDetail(this.iid).then(res=>{
      console.log(res);
      const data = res.result;
      // 获取顶部的图片轮播数据
      this.topImages = data.itemInfo.topImages
    
      //获取商品信息
      this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)

      //创建店铺信息的对象
      this.shop = new Shop(data.shopInfo)

      // 获取商品的详情数据
      this.detailInfo = data.detailInfo

      // 获取参数信息
      this.paramInfo = new GoodsParam(data.itemParams.info,data.itemParams.rule)

      // 取出评论的信息
      if(data.rate.cRate !== 0){
        this.commentInfo = data.rate.list[0]
      }
      // 第一次获取值不对的原因：this.$refs.params.$el压根没有渲染
      //  this.themeTopYs = [];
      //   this.themeTopYs.push(0);
      //   this.themeTopYs.push(this.$refs.params.$el.offsetTop);
      //   this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
      //   this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);
      //   console.log(this.themeTopYs)
      // this.$nextTick(()=>{
        // 根据最新的数据，对应的DOM树已经被渲染出来
        // 但是图片依然没有加载完（目前获取到的offsetTop不包含其中的图片）
        // offsetTop值不对的时候，都是因为图片的问题
        // 第二次获取的值不对：图片没有计算在内
        // this.themeTopYs = [];
        // this.themeTopYs.push(0);
        // this.themeTopYs.push(this.$refs.params.$el.offsetTop);
        // this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
        // this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);
        // console.log(this.themeTopYs)
      // })
    })
  //  请求推荐数据
    getRecommend().then(res=>{
      this.recommends = res.data.list
    })

    // 给getThemeTopY赋值（对给themeTopYs赋值的操作进行防抖）
    this.getThemeTopY=debounce(()=>{
      this.themeTopYs = [];
      this.themeTopYs.push(0);
      this.themeTopYs.push(this.$refs.params.$el.offsetTop);
      this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
      this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);
      this.themeTopYs.push(Number.MAX_VALUE)
      console.log(this.themeTopYs)
    },1000)
  },
  mounted(){
    
  },
  updated(){
    
  },
  destroyed(){
    this.$bus.$off('itemImagLoad',this.itemImgListener)
  },
  methods:{
    detailImageLoad(){
      this.refresh()
      this.getThemeTopY()
    },
    titleClick(index){
      this.$refs.scroll.scrollTo(0,-this.themeTopYs[index],200)
    },
    contentScroll(position){
      // 获取y值
      const positionY = -position.y
      // positionY和主题中的值进行对比
      // [0,7938,9120,9452]
      // positonY在0和7938之间，index=0
      // positonY在7938和9120之间，index=1
      // positonY在9120和9452之间，index=2
      // positonY超过9452，index=3
      let length = this.themeTopYs.length
      for(let i=0;i<length-1;i++){
        if(this.currentIndex !== i && (positionY >= this.themeTopYs[i]&&positionY < this.themeTopYs[i+1])){
          this.currentIndex = i;
          this.$refs.nav.currentIndex = this.currentIndex
        }
        // if(this.currentIndex !== i && ((i<length-1 && positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1])
        // ||(i===length-1 && positionY > this.themeTopYs[i]))){
        //     this.currentIndex = i;
        //     // console.log(this.currentIndex);
        //     this.$refs.nav.currentIndex = this.currentIndex
        // }
      }
    }
  }
}
</script>

<style scoped>
#detail{
position:relative;
z-index:1;
background-color: #fff;
height:100vh;
}
.content{
  /* height:calc(100%-44px); */
  height:100vh;
  position:relative;
  z-index:1;
  background-color: #fff;
}
.detail-nav{
  position:relative;
  z-index:9;
  background-color: #fff;
}
</style>