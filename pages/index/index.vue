<template>
	<view class="container">
		<button type="primary" plain="true" @click="toOrderRK">待处理入库单</button>
		<button type="primary" plain="true" @click="toOrderDB" style="margin-top: 30px;">待处理调拨单</button>
		<button type="primary" plain="true" @click="toOrderXS" style="margin-top: 30px;">待处理销售出库单</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
			}
		},
		methods: {
			toOrderDB () {
				uni.navigateTo({
				    url: '../orderDB/index'
				});
			},
			 toOrderRK () {
				 uni.navigateTo({
				     url: '../orderRK/index'
				 });
			 },
			 toOrderXS () {
				 uni.navigateTo({
				     url: '../orderXS/index'
				 });
			 }
		}
	}
</script>

<style>
	uni-page{
		background: #f5f5f5 !important;
	}
	.container {
		padding: 20px;
		font-size: 14px;
		line-height: 24px;
	}
</style>
