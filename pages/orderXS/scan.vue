<template>
	<view class="container">
		<view class="dbitem">
			<view class="itemBar">
				<view>出库单号：</view>
				<view>{{ fbillno}}</view>
			</view>
			<view class="itemBar">
				<view>出库日期：</view>
				<view>{{ fdate}}</view>
			</view>
			<view class="itemBar">
				<view>发货仓库：</view>
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
				<view>出库数量：</view>
				<view>{{ FAuxQty}}</view>
			</view>
		</view>
		<view style="clear: both;"></view>
		<button type="primary" @click="scan" style="width:200px; margin:30px auto;">扫码</button>
		<view style="margin: 0 auto 5px 10px;">SN码扫描结果</view>
		<view class="scanResultList">
			<view v-for="(code, idx) in scanResultList" class="codeBar">
				<view  @click="seeDetail(idx)" style="width: calc(100% - 80px);display: inline-block;">
					<text>{{ idx + 1}}.</text>
					<text>{{ code.FNumber }} </text>
					<text style="float: right;"> {{ code.FType }}</text>
				</view>
				<button type="warn" style="border: 1px solid #e64340;background:#fff;color:#e64340;float: right;width: 55px;height: 30px;font-size: 12px;margin: 5px 10px 5px 10px;line-height: 28px;" @click="delCode(code.FNumber, idx)">删除</button>
			</view>
		</view>
		<view class="operationBar">
			<button type="secondary" @click="toPackage" style="border: 2px solid darkorange;color: darkorange;background: #fff;">打包</button>
			<button type="warn" @click="submit" :loading="loading" style="border: 2px solid darkorange;background: darkorange;color: #fff;">提交</button>
		</view>
	</view>
</template>

<script>
	import { combineRequsetData } from '../../utils/util.js'
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
			this.getDetail()
		},
		methods: {
			seeDetail (idx) {
				let info = this.scanResultList[idx]
				uni.showModal({
					title: '打包详情',
					content: 'SN码：' + info.FNumber + '\r\n' + 'FCN码：' + info.FCN + '\r\n' + '公司名称：' + info.FCompany + '\r\n' + '打包单号：' + info.FType,
					showCancel: false,
					confirmText: '关闭',
				});
			},
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
						content: 'SN码行数已达到调拨数量',
						showCancel: false
					});
				}
			},
			saveCode (code) {
				var tmpData = '<FInterID>' + this.FInterID + '</FInterID>'
					tmpData += '<FEntryID>' + this.FEntryID + '</FEntryID>'
					tmpData += '<FSN>' +  code + '</FSN>'
				uni.request({
					url: 'http://111.231.134.126:8092/Service1.asmx',
					method: 'POST',
					data: combineRequsetData('Save_21', tmpData),
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
						});
					}
				});
			},
			delCode (code, idx) {
				var tmpData = '<FInterID>' + this.FInterID + '</FInterID>'
					tmpData += '<FEntryID>' + this.FEntryID + '</FEntryID>'
					tmpData += '<FSN>' +  code + '</FSN>'
				uni.request({
					url: 'http://111.231.134.126:8092/Service1.asmx',
					method: 'POST',
					data: combineRequsetData('Delete_21', tmpData),
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
						url: 'http://111.231.134.126:8092/Service1.asmx',
						method: 'POST',
						data: combineRequsetData('Check_21', tmpData),
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
						content: 'SN码行数与调拨数量不一致，请先扫码',
						showCancel: false
					});
				}
			},
			toPackage (order) {
				let tmp = []
				this.scanResultList.map(item => {
					if (!item.FType) {
						tmp.push({FSN: item.FNumber})
					}
				})
				if (tmp.length > 0) {
					uni.navigateTo({
					    url: './package?FSN=' + JSON.stringify(tmp)
					});
				} else {
					uni.showModal({
						content: '当前没有需要打包的SN码',
						showCancel: false
					});
				}
			},
			async getDetail () {
				let packaged = await this.getPackaged()
				var tmpData = "<FSQL>select FNumber,isnull(FType,'')FType,isnull(FCompany,'')FCompany,isnull(FCN,'')FCN from Z_ICSerial_LOG where FInterID=" + this.FInterID + ' and FEntryID=' + this.FEntryID + '</FSQL>'
				uni.request({
					url: 'http://111.231.134.126:8092/Service1.asmx',
					method: 'POST',
					data: combineRequsetData('JA_LIST', tmpData),
					header:{
						'Content-Type':'text/xml'
					},
					success: (res) => {
						if (res.data.length > 0) {
							this.scanResultList = res.data.map(item => {
								return {
									FCN: item.FCN,
									FCompany: item.FCompany,
									FNumber: item.FNumber,
									FType: this.checkIfpackaged(packaged, item) || ''
								}
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
						url: 'http://111.231.134.126:8092/Service1.asmx',
						method: 'POST',
						data: combineRequsetData('JA_LIST', tmpData),
						header:{
							'Content-Type':'text/xml'
						},
						success: (res) => {
							resolve(res.data)
						},
						fail: (err) => {
							console.log('request fail', err);
							uni.showModal({
								content: err.errMsg,
								showCancel: false
							});
						}
					});
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

/* 	.codeBar text:nth-of-type(2){
		float: right;
	} */
</style>
