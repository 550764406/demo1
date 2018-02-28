<template>
<div class="goods-wrapper">
	<div class="header-wrapper" @click="showDetail">
		<div class="header_img">
			<img :src="seller.avatar" width="100%">
		</div>
		<div class="header_desc">
			<div class="title">{{seller.name}}</div>
			<div class="desc">{{ seller.description }}/{{ seller.deliveryTime }}分钟送达</div>
			<div class="extra">{{seller.bulletin}}</div>
		</div>
		<div class="background">
			<img :src="seller.avatar" width="100%" height="100%">
		</div>
	</div><!--header-over-->

	<div class="content-wrapper">
		<div class="menu-wrapper" ref="menuWrapper">
			<ul>
				<li v-for="(item,index) in menuItems" @click="selectMenu(index,$event)">
					<span :class="{'active':currentIndex===index}">{{item.name}}</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" ref="foodWrapper">
			<ul>
				<li v-for="item in menuItems" class="menuItem-hook food-list-hook">
					<h1 class="title">{{item.name}}</h1>
					<ul>
                        <li v-for="(food,index) in item.foods" class="food-item border-1px" @click="selectFood(food, $event)">
                            <div class="icon">
                                <img :src="food.icon" width="57" height="57">
                            </div>
                            <div class="content">
                                <h2 class="name">{{ food.name }}</h2>
                                <p class="desc">{{ food.description }}</p>
                                <div class="extra">
                                    <span class="count">月售{{ food.sellCount }}份</span><span>好评率{{ food.rating }}%</span>
                                </div>
                                <div class="price">
                                    <span class="now">￥{{ food.price }}</span><span v-show="food.oldPrice" class="old">￥{{ food.oldPrice }}</span>
                                </div>
                                <div class="cartcontrol-wrapper">
                                	<cartcontrol :food="food"></cartcontrol>
                                </div>
                            </div>
                        </li>
                    </ul>
				</li>
			</ul>
		</div>
	</div>

	<!--购物车start-->
	<shopcart :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" :select-foods="selectFoods"></shopcart>

    <!--food 详情页-->
    <food :food="selectedFood" ref="food"></food>

	<transition name="fade" mode="out-in">
		<div class="showDetailCon detail" v-show="detailShow" @click="closeDetailShow">
			 <div class="detail-wrapper clearfix">
                    <div class="detail-main">
                        <h1 class="name">{{ seller.name }}</h1>
                        <div class="star-wrapper">
                        </div>
                        <!--flex 布局-->
                        <div class="title">
                            <div class="line"></div>
                            <div class="text">优惠信息</div>
                            <div class="line"></div>
                        </div>
                        <ul v-if="seller.supports" class="supports">
                            <li class="support-item" v-for="(item, index) in seller.supports">
                                <span class="icon" :class="classMap[seller.supports[index].type]"></span>
                                <span class="text">{{ seller.supports[index].description }}</span>
                            </li>
                        </ul>
                        <div class="title">
                            <div class="line"></div>
                            <div class="text">商家公告</div>
                            <div class="line"></div>
                        </div>
                        <div class="bulletin">
                            <p class="content">{{ seller.bulletin }}</p>
                        </div>
                    </div>
                </div>
                <div class="detail-close">
                    <i class="icon-close">关闭</i>
                </div>
		</div>
	</transition><!--hideDetail-over-->
</div>
</template>

<script>
import BScroll from "better-scroll";
import shopcart from "@/components/shopcart/shopcart";
import cartcontrol from "@/components/cartcontrol/cartcontrol";
import Bus from "@/common/js/EventBus";
import food from "@/components/food/food"

export default{
	data(){
		return{
			seller: {},
			detailShow: false,
			menuItems: [],
			heightList: [],
			scrollY: 0,
            selectedFood: {}
		}
	},
	created(){
		//获取头部data
		this.axios.get("/goodData.json",{
		}).then((res) =>{
			this.seller = res.data.seller;

			//左侧菜单
			this.menuItems = res.data.goods;

			this.$nextTick(()=>{
				//初始化滚动条
				this._initScroll();
				this._calculateHeight();
			})

		});
		//活动图标icon 类名
		this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];

		//父元素定义cart.add方法
		Bus.$on('cart.add', target=>{
			//console.log(target)
		})	
	},
	components:{
		shopcart,
		cartcontrol,
        food
	},
	computed: {
		currentIndex(){
			for(let i =0 ; i < this.heightList.length; i++){
				let height1 = this.heightList[i];
				let height2 = this.heightList[i+1];
				if(!height2 || (this.scrollY >= height1 && this.scrollY < height2)){
					return i;
				}
			}
			return 0;
		},
		// 选中的元素
		selectFoods(){
			let foods = [];
            this.menuItems.forEach((good) => {
                good.foods.forEach((food) => {
                    if (food.count) {
                        foods.push(food);
                    }
                });
            });
            return foods;
		}
	},
	methods:{
		showDetail(){
			this.detailShow = true;
		},
		closeDetailShow(){
			this.detailShow = !this.detailShow;
		},
		//初始化滚动条
		_initScroll(){
			this.menuScroll = new BScroll(this.$refs.menuWrapper,{
				click: true
			});

			this.foodScroll = new BScroll(this.$refs.foodWrapper,{
			    scrollY: true,
			    click: true,
			    probeType: 3 // 监测实时滚动的位置
			})

			this.foodScroll.on( "scroll", (pos) =>{
				this.scrollY = Math.abs(Math.round(pos.y));
			})

		},
		//计算右侧高度
		_calculateHeight(){
			let height = 0;
			this.heightList.push(height);
			let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook');
            for (let i = 0; i < foodList.length; i++) {
                let item = foodList[i];
                height += item.clientHeight;
                this.heightList.push(height);
            }
		},
		//选择左侧菜单到对应位置
		selectMenu(index,event){
			let foodList = this.$refs.foodWrapper.getElementsByClassName("food-list-hook");
			let el = foodList[index];
			this.foodScroll.scrollToElement(el, 300)
		},
		//选择
		selectFood(food,event){
			this.selectedFood = food;
            this.$refs.food.show();
		}
	}
}
</script>

<style lang="scss" scoped>
.header-wrapper{
	display: flex;
	padding: 10px 0 8px 10px;
	background: rgba(0,0,0,.5);
	.background{
		position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: -1;
        filter: blur(10px);
	}
	.header_img{
		width: 80px;
		flex: 0 0 80px;
	}
	.header_desc{
		flex: 1;
		color: #fff;
		padding-left: 15px;
		.title{
			font-size: 22px;
		}
		.desc,.extra{
			margin-top: 8px;
			height: 16px;
			white-space: nowrap;
    		overflow: hidden;
    		text-overflow: ellipsis;
    		font-size: 12px;
    		display: block;
    		width: 20%;
		}
	}
}

.content-wrapper{
	background: #fff;
	position: absolute;
    top: 100px;
    bottom: 46px;
    width: 100%;
    overflow: hidden;
}
.showDetailCon{
	width: 100%;
	height: 100%;
	position: absolute;
	left: 0;
	top: 0;
	z-index: 999;
	overflow: auto;
    transition: all .5s;
    background: rgba(7, 17, 27, .8);
    color: #fff;
	
}
.detail {
    position: fixed;
    top: 0;
    left: 0;
    z-index: 100;
    width: 100%;
    height: 100%;
    overflow: auto;
    transition: all .5s;
    background: rgba(7, 17, 27, .8);
    backdrop-filter: blur(10px); /*ios 模糊背景*/
    &.fade-enter-active, &.fade-leave-active {
        transition: opacity .5s
    }
    &.fade-enter, &.fade-leave-active {
        opacity: 0
    }
    .detail-wrapper {
        width: 100%;
        min-height: 100%;
        .detail-main {
            margin-top: 64px;
            padding-bottom: 64px; /*sticky footer 必须有  高度为需要定位元素所占的高度*/
            .name {
                line-height: 16px;
                text-align: center;
                font-size: 16px;
                font-weight: 700;
            }
            .star-wrapper {
                margin-top: 18px;
                padding: 2px 0;
                text-align: center;
            }
            .title {
                display: flex;
                width: 80%;
                margin: 28px auto 24px;
                .line {
                    flex: 1;
                    position: relative;
                    top: -6px;
                    border-bottom: 1px solid rgba(255, 255, 255, .2);
                }
                .text {
                    padding: 0 12px;
                    font-weight: 700;
                    font-size: 14px;
                }
            }
            .supports {
                width: 80%;
                margin: 0 auto;
                .support-item {
                    padding: 0 12px;
                    margin-bottom: 12px;
                    font-size: 0;
                    &:last-child {
                        margin-bottom: 0;
                    }
                    .icon {
                        display: inline-block;
                        width: 16px;
                        height: 16px;
                        vertical-align: top;
                        margin-right: 6px;
                        background-size: 16px 16px;
                        background-repeat: no-repeat;
                        background-image: url("img/decrease_3@2x.png");
						&.decrease {
                           background-image: url("img/decrease_3@2x.png");
                        }
                        &.discount {
                        	background-image: url("img/discount_3@2x.png");
                        }
                        &.guarantee {
                        	background-image: url("img/discount_3@2x.png");
                        }
                        &.invoice {
                        	background-image: url("img/invoice_3@2x.png");
                        }
                        &.special {
                        	background-image: url("img/special_3@2x.png");
                        }
                    }
                    .text {
                        font-size: 12px;
                        line-height: 16px;
                    }
                }
            }
            .bulletin {
                width: 80%;
                margin: 0 auto;
                .content{
                    padding: 0 12px;
                    line-height: 24px;
                    font-size: 12px;
                }
            }
        }
    }
    .detail-close {
        position: relative;
        width: 32px;
        height: 32px;
        margin: -64px auto 0; /*关键*/
        clear: both; /*关键*/
        font-size: 32px;
        color: #fff;
        background: #fff
    }
}
.content-wrapper{
	display: flex;
	.menu-wrapper{
		width: 110px;
		flex: 0 0 110px;
		background: #f8f8f8;
	}
	.foods-wrapper{

	}
}

.menu-wrapper{
	span{
		font-size: 14px;
		display: block;
		text-align: center;
		height: 50px;
		line-height: 1.8em;
		padding: 15px 15px;
		&.active{
			background: #fff;
		}
	}
	
}

.foods-wrapper {
    flex: 1;
    .title {
        padding-left: 14px;
        height: 26px;
        line-height: 26px;
        border-left: 2px solid #d9dde1;
        font-size: 12px;
        color: rgb(147, 153, 159);
        background: #f3f5f7;
    }
    li{
    	list-style: none
    }
    .food-item {
        display: flex;
        margin: 18px;
        padding-bottom: 18px;
        border-bottom: 1px solid rgba(7, 17, 27, .1);
        .icon {
            flex: 0 0 57px;
            margin-right: 10px;
        }
        .content {
            flex: 1;
            position: relative;
            .name {
                margin: 2px 0 8px;
                line-height: 14px;
                font-size: 14px;
                color: rgb(7, 17, 27);
            }
            .desc,
            .extra {
                font-size: 10px;
                color: rgb(147, 153, 159);
            }
            .desc {
                line-height: 12px;
                margin-bottom: 8px;
            }
            .extra {
                line-height: 10px;
                .count {
                    margin-right: 12px;
                }
            }
            .price {
                font-weight: 700;
                line-height: 24px;
                .now {
                    margin-right: 8px;
                    font-size: 14px;
                    color: rgb(240, 20, 20);
                }
                .old {
                    text-decoration: line-through;
                    font-size: 10px;
                    color: rgb(147, 153, 159);
                }
            }
            .cartcontrol-wrapper {
                position: absolute;
                right: 0;
                bottom: -12px;
            }
        }
    }
}
</style>