<template>
    <div class="shopcart">
        <div class="content" @click="togglelist">
            <div class="content-left">
                <div class="logo-wrapper">
                    <div class="logo">
                        <i class="icon-shopping_cart"></i>
                    </div>
                    <div class="num" v-show="totalCount > 0">{{totalCount}}</div>
                </div>
                <div class="price">￥{{totalPrice}}</div>
                <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
            </div>
            <div class="content-right" @click.stop.prevent="pay">
                <div class="pay" :class="payClass">{{payDesc}}</div>
            </div>
        </div>
        
        <transition name="fold">
            <div class="shopcart-list" v-show="listShow">
                <div class="list-header">
                    <h1 class="title">购物车</h1>
                    <span class="empty" @click="empty">清空</span>
                </div>
                <div class="list-content" ref="listContent">
                    <ul>
                        <li class="food" v-for="food in selectFoods">
                            <span class="name">{{food.name}}</span>
                            <span class="price">￥{{food.price}}</span>
                            <div class="cartcontrol-wrapper">
                                <cartcontrol :food="food"></cartcontrol>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
        </transition>
        
    </div>
</template>

<script>
import cartcontrol from "@/components/cartcontrol/cartcontrol";
import BScroll from 'better-scroll';

export default{
    data(){
        return{
            fold: true
        }
    },
    components:{
        cartcontrol
    },
    props:['deliveryPrice','minPrice','selectFoods','food'],
    computed:{
        //选中的总个数
        totalCount(){
            let count = 0;
            this.selectFoods.forEach((food)=>{
                count += food.count
            });
            return count;
        },
        //选中的总价钱
        totalPrice(){
            let price = 0;
            this.selectFoods.forEach((food)=>{
                price += food.price * food.count
            });
            return price;
        },
        //判断显示按钮
        payDesc(){
            if(this.totalPrice === 0){
                return `￥${this.minPrice}起送`
            }
            else if(this.totalPrice < this.minPrice){
                let diff = this.minPrice - this.totalPrice;
                return `还差${diff}起送`
            }
            else{
                return "去结算"
            }
        },
        //判断显示类名
        payClass(){
            if(this.totalPrice < this.minPrice){
                return "not-enough"
            }
            else{
                return "enough"
            }
        },
        //设置滚动条
        listShow(){
            if(!this.totalCount){
                this.fold = false;
                return false;
            }
            let show = !this.fold;
            if(show){
                this.$nextTick(()=>{
                    if(!this.listScroll){
                        this.listScroll = new BScroll(this.$refs.listContent,{
                            click: true
                        });
                    }
                    else{
                        this.listScroll.refresh();
                    }
                    
                })
            }
            return show;
        }
    },
    methods:{
        togglelist(){
            if(!this.totalCount){
                return;
            }
            this.fold = !this.fold;
        },
        //清空购物车
        empty(){
            this.selectFoods.forEach((food)=>{
                food.count = 0;
            })
        }
    }
}
</script>

<style lang="scss" scoped>
.icon-shopping_cart:before {
  content: "\e907";
}
.shopcart {
    position: fixed;
    left: 0;
    bottom: 0;
    z-index: 50;
    width: 100%;
    height: 48px;
    .content {
        display: flex;
        background: #141d27;
        font-size: 0;
        color: rgba(255, 255, 255, .4);
        .content-left {
            flex: 1;
            .logo-wrapper {
                display: inline-block;
                vertical-align: top;
                position: relative;
                top: -10px;
                margin: 0 18px;
                padding: 6px;
                width: 56px;
                height: 56px;
                box-sizing: border-box;
                border-radius: 50%;
                background: #141d27;
                .logo {
                    width: 100%;
                    height: 100%;
                    border-radius: 50%;
                    background: #2b343c;
                    text-align: center;
                    .icon-shopping_cart {
                        font-size: 24px;
                        line-height: 44px;
                        color: #80858a;
                    }
                }
                .num {
                    position: absolute;
                    top: 0;
                    right: 0;
                    width: 24px;
                    height: 16px;
                    line-height: 16px;
                    text-align: center;
                    border-radius: 16px;
                    font-size: 9px;
                    font-weight: 700;
                    color: #fff;
                    background: rgb(240, 20, 20);
                    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, .4);
                }
            }
            .price {
                vertical-align: top;
                display: inline-block;
                line-height: 24px;
                margin-top: 12px;
                padding-right: 12px;
                box-sizing: border-box;
                border-right: 1px solid rgba(255, 255, 255, .1);
                font-size: 16px;
                font-weight: 700;
            }
            .desc {
                display: inline-block;
                vertical-align: top;
                line-height: 24px;
                margin: 12px 0 0 12px;
                font-size: 10px;
            }
            &.highlight {
                .logo-wrapper {
                    .logo {
                        background: rgb(0, 160, 220);
                        .icon-shopping_cart {
                            color: #fff;
                        }
                    }
                }
                .price {
                    color: #fff;
                }
            }
        }
        .content-right {
            flex: 0 0 105px;
            width: 105px;
            .pay {
                height: 48px;
                line-height: 48px;
                text-align: center;
                font-size: 12px;
                font-weight: 700;
                &.not-enough {
                    background: #2b333b;
                }
                &.enough {
                    background: #00b43c;
                    color: #fff;
                }
            }
        }
    }
    .ball-container {
        .ball {
            position: fixed;
            left: 32px;
            bottom: 22px;
            z-index: 200;
            .inner {
                width: 16px;
                height: 16px;
                border-radius: 50%;
                background: rgb(0, 160, 220);
            }
            &.drop-enter-active {
                transition: all .4s cubic-bezier(.49, -0.29, .75, .41);
                .inner {
                    transition: all .4s linear;
                }
            }
        }
    }
    .shopcart-list {
        position: absolute;
        top: 0;
        left: 0;
        z-index: -1;
        width: 100%;
        box-shadow: -1px -1px 2px #999;
        transform: translate3d(0, -100%, 0);
        &.fold-enter-active, &.fold-leave-active {
            transition: all .5s;
        }
        &.fold-enter, &.fold-leave-active {
            transform: translate3d(0, 0, 0);
        }
        .list-header {
            height: 40px;
            line-height: 40px;
            padding: 0 18px;
            background: #f3f5f7;
            border-bottom: 1px solid rgba(7, 17, 27, .1);
            .title {
                float: left;
                font-size: 14px;
                color: rgb(7, 17, 27);
            }
            .empty {
                float: right;
                font-size: 12px;
                color: rgb(0, 160, 220);
            }
        }
        .list-content {
            padding: 0 18px;
            max-height: 217px;
            background: #fff;
            overflow: hidden;

            .food {
                position: relative;
                padding: 12px 0;
                box-sizing: border-box;
                list-style: none;
                border-bottom: 1px solid rgba(7, 17, 27, .1);
                .name {
                    line-height: 24px;
                    font-size: 14px;
                    color: rgb(7, 17, 27);
                }
                .price {
                    position: absolute;
                    right: 90px;
                    bottom: 12px;
                    line-height: 24px;
                    font-weight: 700;
                    font-size: 14px;
                    color: rgb(240, 20, 20);
                }
                .cartcontrol-wrapper {
                    position: absolute;
                    bottom: 6px;
                    right: 0;
                }
            }
        }
    }
}

.list-mask {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 40;
    background: rgba(7, 17, 27, .6);
    backdrop-filter: blur(10px);
    &.fade-enter-active, .fade-leave-active {
        transition: all .5s;
    }
    .fade-enter, .fade-leave-active {
        opacity: 0
    }
}
</style>