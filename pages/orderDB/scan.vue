<template>
	<view class="container">
		<view class="dbitem">
			<view class="itemBar">
				<view>调拨单号：</view>
				<view>{{ fbillno}}</view>
			</view>
			<view class="itemBar">
				<view>调拨日期：</view>
				<view>{{ fdate}}</view>
			</view>
			<view class="itemBar">
				<view>调入仓库：</view>
				<view>{{ fscstock}}</view>
			</view>
			<view class="itemBar">
				<view>调出仓库：</view>
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
				<view>调拨数量：</view>
				<view>{{ FAuxQty}}</view>
			</view>
		</view>
		<view style="clear: both;"></view>
<!-- 		<button type="primary" @click="scan" style="width:200px; margin:30px auto;">扫码</button>-->		
		<view style="margin: 0 auto 5px 10px;">SN码扫描结果</view>
		<view class="scanResultList">
			<view v-for="(code, idx) in scanResultList" class="codeBar">
				<text>{{ idx + 1}}.</text>
				<text>{{ code.FNumber }} </text>
				<button type="warn" style="border: 1px solid #e64340;background:#fff;color:#e64340;float: right;width: 55px;height: 30px;font-size: 12px;margin: 5px 10px 5px 10px;line-height: 28px;" @click="delCode(code.FNumber, idx)">删除</button>
				<text style="float: right;"> {{ code.FType }}</text>
			</view>
		</view>
		<view class="operationBar">
			<button type="secondary" @click="package" :loading="loadingPackage" style="border: 2px solid darkorange;color: darkorange;background: #fff;">打包</button>
			<button type="warn" @click="submit" :loading="loading" style="border: 2px solid darkorange;background: darkorange;color: #fff;">提交</button>
		</view>
		<scan-code></scan-code>
	</view>
</template>

<script>
	import { combineRequsetData } from '../../utils/util.js'
	import { mainUrl } from '../../utils/url.js'
	import scanCode from "@/components/scan-code/scan-code.vue"

	export default {
		data() {
			return {
				loading: false,
				loadingPackage: false,
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
				DBList: []
			}
		},
		components: {
			scanCode
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
		onShow () {
			var _this = this
			uni.$off('scancodedate') // 每次进来先 移除全局自定义事件监听器  
			uni.$on('scancodedate',(data) => {  
				_this.result = data.code
				_this.broadcastBackInfo(data.code)
			})
		},
		created () {
			this.getDetail()
		},
		methods: {
			broadcastBackInfo (result) {
				if (this.FAuxQty > this.scanResultList.length ) {
					this.saveCode(result)
				} else {
					uni.showModal({
						content: 'SN码行数已达到调拨数量',
						showCancel: false
					})
				}
			},
			scan () {
				if (this.FAuxQty > this.scanResultList.length ) {
					uni.scanCode({
					    onlyFromCamera: true,
					    success: (res) => {
							this.saveCode(res.result)
					    },
						fail: (err) => {
							console.log(err)
						}
					})
				} else {
					uni.showModal({
						content: 'SN码行数已达到调拨数量',
						showCancel: false
					})
				}
			},
			saveCode (code) {
				var tmpData = '<FInterID>' + this.FInterID + '</FInterID>'
					tmpData += '<FEntryID>' + this.FEntryID + '</FEntryID>'
					tmpData += '<FSN>' +  code + '</FSN>'
				uni.request({
					url: mainUrl,
					method: 'POST',
					data: combineRequsetData('Save', tmpData),
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
								this.scanResultList.push({FNumber: code, FType: null})
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
						})
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
					data: combineRequsetData('Delete', tmpData),
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
									content: '删除失败',
									showCancel: false
								});
							break
						}s						
					},
					fail: (err) => {
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						})
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
						data: combineRequsetData('Check', tmpData),
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
									uni.navigateBack()
								break
								default:
									uni.showModal({
										content: '提交失败',
										showCancel: false
									})
								break
							}
							
						},
						fail: (err) => {
							uni.showModal({
								content: err.errMsg,
								showCancel: false
							})
						},
						complete: () => {
							this.loading = false
						}
					});
				} else {
					uni.showModal({
						content: 'SN码行数与调拨数量不一致，请先扫码',
						showCancel: false
					})
				}
			},
			package () {
				let tmp = {items:[]}
				this.scanResultList.map(item => {
					if (!item.FType) {
						tmp.items.push({FSN: item.FNumber})
					}
				})
				if (tmp.items.length == 0) {
					uni.showModal({
						content: '当前没有需要打包的SN码',
						showCancel: false
					});
				} else {
					this.loadingPackage = true
					var tmpData = '<FJson>' + JSON.stringify(tmp) + '</FJson>'
					uni.request({
						url: mainUrl,
						method: 'POST',
						data: combineRequsetData('pack', tmpData),
						header:{
							'Content-Type':'text/xml'
						},
						success: (res) => {
							switch (res.data[0].code) {
								case '1':
									uni.showToast({
										title: '打包成功',
										icon: 'success',
										mask: true,
										duration: 1500
									})
									this.loadingPackage = false
									this.getDetail()
								break
								default:
									uni.showModal({
										content: '打包失败',
										showCancel: false
									})
								break
							}
							
						},
						fail: (err) => {
							uni.showModal({
								content: err.errMsg,
								showCancel: false
							})
						},
						complete: () => {
							this.loading = false
						}
					});
				}
			},
			async getDetail () {
				let packaged = await this.getPackaged()
				var tmpData = '<FSQL>select FNumber from Z_ICSerial_LOG where FInterID=' + this.FInterID + ' and FEntryID=' + this.FEntryID + '</FSQL>'
				uni.request({
					url: mainUrl,
					method: 'POST',
					data: combineRequsetData('JA_LIST', tmpData),
					header:{
						'Content-Type':'text/xml'
					},
					success: (res) => {
						if (res.data.length > 0) {
							this.scanResultList = res.data.map(item => {
								return {
									FNumber: item.FNumber,
									FType: this.checkIfpackaged(packaged, item) || null
								}
							})
						}
					},
					fail: (err) => {
						console.log('request fail', err);
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						})
					}
				});
			},
			checkIfpackaged (packaged, curItem) {
				for (let i = 0; i < packaged.length; i++) {
					if (curItem.FNumber == packaged[i].FNumber) {
						return packaged[i].FType
					}
				}
			},
			getPackaged () {
				return new Promise((resolve, reject) => {
					var tmpData = "<FSQL>select FNumber,isnull(FType,'')FType from Z_ICSerial_LOG where FInterID=" + this.FInterID + ' and FEntryID=' + this.FEntryID + '</FSQL>'
					uni.request({
						url: mainUrl,
						method: 'POST',
						data: combineRequsetData('JA_LIST', tmpData),
						header:{
							'Content-Type':'text/xml'
						},
						success: (res) => {
							resolve(res.data)
						},
						fail: (err) => {
							uni.showModal({
								content: err.errMsg,
								showCancel: false
							})
						}
					})
				})
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
	.operationBar{
		width: 100%;
		margin: 30px 0;
		text-align: center;
		
	}
	.operationBar button{
		width: 150px;
		margin: 0 20px;
		display: inline-block;
	}
	.scanResultList{
		width: calc(100% - 20px);
		padding: 10px;
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
</style>
