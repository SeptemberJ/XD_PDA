<template>
	<view class="container">
		<view v-if="loading"><uni-loading :height1="100" :loadModal="loading"></uni-loading></view>
		<view v-if="!loading && DBList.length == 0" style="text-align: center;background-color: #f5f5f5;padding-top: 40px;">
			<image src="../../static/wushuju.png" style="width: 128px;height: 84px;margin: 0 auto 20px auto;display: block;"></image>
			<text style="color: #666;">暂无数据</text>
		</view>
		<view v-if="!loading && DBList.length > 0" class="dbitem" v-for="order in DBList" @click="toScan(order)">
			<view class="itemBar" style="padding-left: 0;border-left: 4px solid #6190e8;margin-bottom: 5px;">
				<view><text style="padding-left: 6px;">出库单号：</text></view>
				<view>{{ order.FBillNo}}</view>
			</view>
			<view class="itemBar">
				<view>出库日期：</view>
				<view>{{ order.fdate}}</view>
			</view>
			<view class="itemBar">
				<view>发货仓库：</view>
				<view>{{ order.fdcstcok}}</view>
			</view>
			<view class="itemBar">
				<view>产品代码：</view>
				<view>{{ order.FNumber}}</view>
			</view>
			<view class="itemBar">
				<view>产品名称：</view>
				<view>{{ order.FName}}</view>
			</view>
			<view class="itemBar">
				<view>产品规格：</view>
				<view>{{ order.FModel}}</view>
			</view>
			<view class="itemBar">
				<view>单&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;位：</view>
				<view>{{ order.funit}}</view>
			</view>
			<view class="itemBar">
				<view>出库数量：</view>
				<view>{{ order.FAuxQty}}</view>
			</view>
		</view>
		<view style="clear: both;"></view>
	</view>
</template>

<script>
	import { combineRequsetData } from '../../utils/util.js'
	import uniLoading from '@/components/loading/loading.vue'
	import { mainUrl } from '../../utils/url.js'
	export default {
		components: {
			uniLoading
		},
		data() {
			return {
				DBList: [],
				loading: true
			}
		},
		onShow () {
			this.getList()
		},
		onPullDownRefresh() {
			this.loading = true
			this.DBList = []
			this.getList()
		},
		methods: {
			toScan (order) {
				uni.navigateTo({
				    url: './scan?FInterID=' + order.FInterID + '&FEntryID=' + order.FEntryID +'&fbillno=' + order.FBillNo + '&fdate=' + order.fdate + '&fscstock=' + order.fscstock + '&fdcstcok=' + order.fdcstcok + '&FNumber=' + order.FNumber + '&fname=' + order.FName + '&FModel=' + order.FModel + '&funit=' + order.funit  + '&FAuxQty=' + order.FAuxQty
				});
			},
			getList () {
				var tmpData = '<FSQL>SELECT * FROM z_icstockbill_21</FSQL>'
				uni.request({
					url: mainUrl,
					method: 'POST',
					data: combineRequsetData('JA_LIST', tmpData),
					header:{
						'Content-Type':'text/xml'
					},
					success: (res) => {
						this.DBList = res.data
					},
					complete: () => {
						this.loading = false;
						uni.stopPullDownRefresh()
					}
				})
			}
		}
	}
</script>

<style>
	.container {
		min-height: 100vh;
		font-size: 14px;
		line-height: 24px;
		background-color: #F5F5F5;
		text-align: center;
	}
	.dbitem{
		width: 100%;
		padding: 10px 0;
		background-color: #ffffff;
		margin-bottom: 20px;
		float: left;
	}
	.itemBar{
		width: calc(100% - 20px);
		padding: 0 10px;
		float: left;
	}
	.itemBar view:nth-of-type(1){
		float: left;
		width: 26%;
		color: #333333;
		font-weight: 500;
		text-align: left;
	}
	.itemBar view:nth-of-type(2){
		float: left;
		width: 74%;
		color: #999999;
		text-align: right;
	}
</style>
