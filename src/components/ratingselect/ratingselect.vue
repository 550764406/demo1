<template>
	<div class="ratingselect">
		<div class="rating-type">
			<span class="block positive" :class="{'active': selectType === 2}">全部</span>
			<span class="block positive" :class="{'active': selectType === 0}">满意</span>
			<span class="block negative" :class="{'active': selectType === 1}">吐槽</span>
		</div>
	</div>
</template>

<script>
const POSITIVE = 0;
const NEGATIVE = 1;
const ALL = 2;

export default{
	props: ['ratings','selectType','onlyContent']
}
</script>

<style lang="scss" scroped>
.ratingselect {
	.rating-type {
	    padding: 18px 0;
	    margin: 0 18px;
	    border-bottom: 1px solid rgba(7, 17, 27, .1);
	    font-size: 0;
	    .block {
	        display: inline-block;
	        padding: 8px 12px;
	        margin-right: 8px;
	        border-radius: 1px;
	        line-height: 16px;
	        color: rgb(77, 85, 93);
	        font-size: 12px;
	        &.active {
	            color: #fff;
	        }
	        &.positive {
	            background: rgba(0, 160, 220, .2);
	            &.active {
	                background: rgb(0, 160, 220);
	            }
	        }
	        &.negative {
	            background: rgba(77, 85, 93, .2);
	            &.active {
	                background: rgb(77, 85, 93);
	            }
	        }
	        .count {
	            font-size: 8px;
	            margin-left: 2px;
	        }
	    }
	}
	.switch {
	    padding: 12px 18px;
	    line-height: 24px;
	    border-bottom: 1px solid rgba(7, 17, 27, .1);
	    color: rgb(147, 153, 159);
	    font-size: 0;
	    &.on {
	        .icon-check_circle {
	            color: #00c850;
	        }
	    }
	    .icon-check_circle {
	        display: inline-block;
	        vertical-align: top;
	        margin-right: 4px;
	        font-size: 24px;
	    }
	    .text {
	        display: inline-block;
	        vertical-align: top;
	        font-size: 12px;
	    }
	}
}
</style>