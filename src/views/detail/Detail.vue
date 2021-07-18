<template>
    <div id="detail">
        <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"></detail-nav-bar>
        <scroll class="content" ref="scroll" @scroll="contentScroll" :probe-type="3">
            <detail-swiper :top-images="topImages"></detail-swiper>
            <detail-base-info :goods="goods"></detail-base-info>
            <detail-shop-info :shop="shop"></detail-shop-info>
            <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"></detail-goods-info>
            <detail-param-info ref="params" :param-info="paramInfo"></detail-param-info>
            <detail-comment-info ref="comment" :comment-info="commentINfo"></detail-comment-info>
            <goods-list ref="recommend" :goods="recommends"></goods-list>
        </scroll>
        <detail-bottom-bar @addCart="addToCart"></detail-bottom-bar>
        <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
        <toast :message="message" :show="show"></toast>
    </div>
</template>

<script>
    import DetailNavBar from "./childComps/DetailNavBar";
    import DetailSwiper from "./childComps/DetailSwiper";
    import DetailBaseInfo from "./childComps/DetailBaseInfo";
    import DetailShopInfo from "./childComps/DetailShopInfo";
    import DetailGoodsInfo from "./childComps/DetailGoodsInfo";
    import DetailParamInfo from "./childComps/DetailParamInfo";
    import DetailCommentInfo from "./childComps/DetailCommentInfo";
    import DetailBottomBar from "./childComps/DetailBottomBar";

    import Scroll from "../../components/common/scroll/Scroll";
    import GoodsList from "../../components/content/goods/GoodsList";
    import BackTop from "../../components/content/backtop/BackTop";
    import Toast from "../../components/common/toast/Toast";

    import {getDetail,Goods,Shop,GoodsParam,getRecommend} from "../../network/detail";
    import {debounce} from "../../common/utils";

    export default {
        name: "Detail",
        components: {
            DetailNavBar,
            DetailSwiper,
            DetailBaseInfo,
            DetailShopInfo,
            Scroll,
            DetailGoodsInfo,
            DetailParamInfo,
            DetailCommentInfo,
            GoodsList,
            DetailBottomBar,
            BackTop,
            Toast
        },
        data() {
            return {
                iid: null,
                topImages: [],
                goods: {},
                shop: {},
                detailInfo: {},
                paramInfo: {},
                commentINfo: {},
                recommends: [],
                themeTopYs: [],
                getThemeTopY: null,
                isShowBackTop: 'false',
                message: '',
                show: false
            }
        },
        created() {
            //保存传入的iid
            this.iid = this.$route.params.iid

            //根据iid取详细数据
            getDetail(this.iid).then(res => {
                const data = res.result
                this.topImages = data.itemInfo.topImages
                //获取商品信息
                this.goods = new Goods(data.itemInfo, data.columns, data.shopInfo.services)
                //获取店铺信息
                this.shop = new Shop(data.shopInfo)
                //保存商品详情页的数据
                this.detailInfo = data.detailInfo;
                //获取参数信息
                this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule)
                //取出评论信息
                if (data.rate.cRate !== 0) {
                    this.commentINfo = data.rate.list[0]
                }
            })
            // console.log(this.recommends)
            getRecommend().then(res => {
                // console.log(res.data.list)
                this.recommends = res.data.list
                // console.log(this.recommends)
            })
            //请求推荐数据
            this.getThemeTopY = debounce(() => {
                this.themeTopYs = []
                this.themeTopYs.push(0)
                this.themeTopYs.push(this.$refs.params.$el.offsetTop)
                this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
                this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
            },100)
        },
        methods:{
            imageLoad() {
                this.$refs.scroll.refresh()
                this.getThemeTopY()
            },
            titleClick(index) {
                this.getThemeTopY()
                this.$refs.scroll.scrollTo(0,-this.themeTopYs[index],100)
            },
            contentScroll(position){
                const positionY = -position.y
                let length = this.themeTopYs.length
                for(let i=0;i<length;i++){
                    if ((i<length - 1&&positionY>=this.themeTopYs[i]&&positionY<this.themeTopYs[i+1])||(i===length-1&&positionY>=this.themeTopYs[i])){
                       this.$refs.nav.currentIndex = i
                    }
                }
                this.isShowBackTop = (-position.y) > 1000
            },
            backClick(){
                this.$refs.scroll.scrollTo(0,0)
            },
            addToCart(){
                const product = {}
                product.image = this.topImages[0]
                product.title = this.goods.title
                product.desc = this.goods.desc
                product.price = this.goods.realPrice
                product.iid = this.iid

                // this.$store.commit('addCart',product)
                this.$store.dispatch('addCart',product).then(res => {
                    this.show = true;
                    this.message = res;
                    setTimeout(() => {
                        this.show = false;
                        this.message = ''
                    },1500)
                })
            }
        },
        mounted(){
            const refresh = debounce(this.$refs.scroll.refresh,50)
            this.$bus.$on('detailImageLoad',() => {
                refresh()
            })

        },
    }

</script>

<style scoped>
    #detail {
        position:  relative;
        z-index: 9;
        background-color: #fff;
        height: 100vh;
    }

    .detail-nav{
        position: relative;
        z-index: 9;
        background-color: #fff;
    }

    .content{
        height: calc(100% - 44px - 58px);
    }
</style>