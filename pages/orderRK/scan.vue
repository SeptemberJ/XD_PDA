<template>
	<view class="container">
		<view class="dbitem">
			<view class="itemBar">
				<view>入库单号：</view>
				<view>{{ fbillno}}</view>
			</view>
			<view class="itemBar">
				<view>入库日期：</view>
				<view>{{ fdate}}</view>
			</view>
			<!-- <view class="itemBar">
				<view>调入仓库：</view>
				<view>{{ fscstock}}</view>
			</view> -->
			<view class="itemBar">
				<view>收货仓库：</view>
				<view>{{ fdcstcok}}</view>
			</view>
			<view class="itemBar">
				<view>产品代码：</view>
				<view>{{ FNumber}}</view>
			</view>
			<view class="itemBar">
				<view>产品名称：</view>
				<view>{{ fname}}</view>
			</view>
			<view class="itemBar">
				<view>产品规格：</view>
				<view>{{ FModel}}</view>
			</view>
			<view class="itemBar">
				<view>单&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;位：</view>
				<view>{{ funit}}</view>
			</view>
			<view class="itemBar">
				<view>入库数量：</view>
				<view>{{ FAuxQty}}</view>
			</view>
			<!-- <view class="itemBar">
				<view>SN&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;码：</view>
				<view>{{ SNCode}}</view>
			</view> -->
		</view>
		<view style="clear: both;"></view>
		<button type="primary" @click="scan">扫码</button>
		<view style="margin: 0 auto 5px 20px;">SN码扫描结果</view>
		<view class="scanResultList">
			<view v-for="(code, idx) in scanResultList" class="codeBar">
				<text>{{ idx + 1}}.</text>
				<text>{{ code }} </text>
				<button type="warn" style="width: 55px;height: 30px;font-size: 12px;margin: 5px 10px 5px 0;" @click="delCode(code, idx)">删除</button>
			</view>
		</view>
		<button type="warn" @click="submit" :loading="loading">提交</button>
	</view>
</template>

<script>
	import { combineRequsetData } from '../../utils/util.js'
	import { mainUrl } from '../../utils/url.js'
	export default {
		data() {
			return {
				loading: false,
				FInterID: '',
				FEntryID: '',
				fbillno: '',
				fdate: '',
				fscstock: '',
				fdcstcok: '',
				FNumber: '',
				fname: '',
				FModel: '',
				funit: '',
				FAuxQty: '',
				SNCode: '',
				scanResultList: [],
				scanResultList2: ['01.05.11002/00000002', '01.05.11002/00000003', '01.05.11002/00000003', '01.05.11002/00000003', '01.05.11002/00000003', '01.05.11002/00000003', '01.05.11002/00000003'],
				DBList: []
			}
		},
		onLoad: function (options) {
			this.FInterID = options.FInterID
			this.FEntryID = options.FEntryID
			this.fbillno = options.fbillno
			this.fdate = options.fdate
			this.fscstock = options.fscstock
			this.fdcstcok = options.fdcstcok
			this.FNumber = options.FNumber
			this.fname = options.fname
			this.FModel = options.FModel
			this.funit = options.funit
			this.FAuxQty = options.FAuxQty
		},
		created () {
			this.getDetail()
		},
		methods: {
			scan () {
				if (this.FAuxQty > this.scanResultList.length ) {
					uni.scanCode({
					    onlyFromCamera: true,
					    success: (res) => {
					        // console.log('条码类型：' + res.scanType + '条码内容：' + res.result)
							this.saveCode(res.result)
					    },
						fail: (err) => {
							console.log(err)
						}
					})
				} else {
					uni.showModal({
						content: 'SN码行数已达到入库数量',
						showCancel: false
					});
				}
			},
			saveCode (code) {
				var tmpData = '<FInterID>' + this.FInterID + '</FInterID>'
					tmpData += '<FEntryID>' + this.FEntryID + '</FEntryID>'
					tmpData += '<FSN>' +  code + '</FSN>'
				uni.request({
					url: mainUrl,
					method: 'POST',
					data: combineRequsetData('Save_2', tmpData),
					header:{
						'Content-Type':'text/xml'
					},
					success: (res) => {
						switch (res.data[0].code) {
							case '1':
								uni.showToast({
									title: '扫码录入成功',
									icon: 'success',
									mask: true,
									duration: 1500
								})
								this.scanResultList.push(code)
							break
							case '0':
								uni.showModal({
									content: '库存中不存在该SN码',
									showCancel: false
								});
							break
						}
						
					},
					fail: (err) => {
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						});
					}
				});
			},
			delCode (code, idx) {
				var tmpData = '<FInterID>' + this.FInterID + '</FInterID>'
					tmpData += '<FEntryID>' + this.FEntryID + '</FEntryID>'
					tmpData += '<FSN>' +  code + '</FSN>'
				uni.request({
					url: mainUrl,
					method: 'POST',
					data: combineRequsetData('Delete_2', tmpData),
					header:{
						'Content-Type':'text/xml'
					},
					success: (res) => {
						switch (res.data[0].code) {
							case '1':
								uni.showToast({
									title: '删除成功',
									icon: 'success',
									mask: true,
									duration: 1500
								})
								this.scanResultList.splice(idx, 1)
							break
							case '0':
								uni.showModal({
									content: '删除成功失败',
									showCancel: false
								});
							break
						}s						
					},
					fail: (err) => {
						console.log('request fail', err);
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						});
					}
				});
			},
			submit () {
				if (this.FAuxQty == this.scanResultList.length ) {
					this.loading = true
					var tmpData = '<FInterID>' + this.FInterID + '</FInterID>'
						tmpData += '<FEntryID>' + this.FEntryID + '</FEntryID>'
					uni.request({
						url: mainUrl,
						method: 'POST',
						data: combineRequsetData('Check_2', tmpData),
						header:{
							'Content-Type':'text/xml'
						},
						success: (res) => {
							switch (res.data[0].code) {
								case '1':
									uni.showToast({
										title: '提交成功',
										icon: 'success',
										mask: true,
										duration: 1500
									})
									uni.navigateTo({
									    url: './index'
									});
								break
								default:
									uni.showModal({
										content: '提交失败',
										showCancel: false
									});
								break
							}
							
						},
						fail: (err) => {
							uni.showModal({
								content: err.errMsg,
								showCancel: false
							});
						},
						complete: () => {
							this.loading = false
						}
					});
				} else {
					uni.showModal({
						content: 'SN码行数与入库数量不一致，请先扫码',
						showCancel: false
					});
				}
			},
			getDetail () {
				var tmpData = '<FSQL>select FNumber from Z_ICSerial_LOG where FInterID=' + this.FInterID + ' and FEntryID=' + this.FEntryID + '</FSQL>'
				uni.request({
					url: mainUrl,
					method: 'POST',
					data: combineRequsetData('JA_LIST', tmpData),
					header:{
						'Content-Type':'text/xml'
					},
					success: (res) => {
						console.log(res.data)
						if (res.data.length > 0) {
							this.scanResultList = res.data.map(item => {
								return item.FNumber
							})
						}
					},
					fail: (err) => {
						console.log('request fail', err);
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						});
					}
				});
			}
		}
	}
</script>

<style>
	.container {
		padding: 20 0 20px 0;
		font-size: 14px;
		line-height: 24px;
	}
	.dbitem{
		width: 100%;
		padding: 10px 0;
		background-color: #ffffff;
		float: left;
	}
	.itemBar{
		width: calc(100% - 20px);
		padding: 0 10px;
		float: left;
	}
	.itemBar view:nth-of-type(1){
		float: left;
		width: 30%;
		color: #333333;
		font-weight: 500;
	}
	.itemBar view:nth-of-type(2){
		float: left;
		width: 70%;
		color: #999999;
		text-align: right;
	}
	button{
		width: 50%;
		margin: 30px auto;
	}
	.scanResultList{
		width: 90%;
		padding: 10px 0;
		min-height: 200px;
		margin: 0 auto 20px auto;
		background-color: #efefef;
	}
	.codeBar{
		width: 100%;
		height: 40px;
		line-height: 40px;
	}
	.codeBar text{
		padding: 0 10px;
	}
	.codeBar button{
		float: right;
	}
/* 	.codeBar text:nth-of-type(2){
		float: right;
	} */
</style>
