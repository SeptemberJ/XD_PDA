<template>
	<view class="container">
		<view class="itemBar">
			<view>快递单号：</view>
			<view>
				<text style="margin-right: 10px;">{{ FCN }}</text>
				<button type="primary" @click="scan" style="width:100px;height: 40px;line-height: 40px; float: right;background: #fff; border: 1px solid #007aff;color: #007aff;margin-top: 5px;">扫码</button>
				</view>
		</view>
		<view class="itemBar">
			<view>公司名称：</view>
			<view>
				<picker @change="bindPickerChange" :value="index" :range="companyList">
					{{companyList[index]}}
				</picker>
			</view>
		</view>
		<view class="itemBar" style="margin-top: 30px;">
			<button type="secondary" @click="package" :loading="loadingPackage" style="width: 200px;border: 2px solid darkorange;color: #fff;background: darkorange;">提交</button>
		</view>
	</view>
</template>

<script>
	import { combineRequsetData } from '../../utils/util.js'
	import { mainUrl } from '../../utils/url.js'
	export default {
		data() {
			return {
				FSNList: [],
				companyIDList: [],
				companyList: [],
				FCN: '',
				FCompany: '',
				index: 0,
				loadingPackage: false
			}
		},
		onLoad: function (options) {
			this.FSNList = JSON.parse(options.FSN)
			console.log(JSON.parse(options.FSN))
		},
		onShow () {
			this.getCompany()
		},
		methods: {
			scan () {
				uni.scanCode({
				    onlyFromCamera: true,
				    success: (res) => {
						this.FCN = res.result
				    },
					fail: (err) => {
						console.log(err)
					}
				})
			},
			bindPickerChange (e) {
				this.index = e.target.value
				this.FCompany = this.companyIDList[e.target.value]
			},
			package () {
				if (!this.FCN || !this.FCompany) {
					uni.showModal({
						content: '快递单号和公司为必填项',
						showCancel: false
					});
					return false
				}
				let tmp = {items: this.FSNList}
				this.loadingPackage = true
				var tmpData = '<FJson>' + JSON.stringify(tmp) + '</FJson>'
					tmpData += '<FCompany>' + this.FCompany + '</FCompany>'
					tmpData += '<FCN>' + this.FCN + '</FCN>'
				uni.request({
					url: mainUrl,
					method: 'POST',
					data: combineRequsetData('pack_21', tmpData),
					header:{
						'Content-Type':'text/xml;charset=utf-8'
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
								uni.navigateBack();
							break
							default:
								uni.showModal({
									content: '打包失败',
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
			},
			getCompany () {
				var tmpData = '<FSQL>select FName,FItemID from t_Item where FItemClassID=3005</FSQL>'
				uni.request({
					url: mainUrl,
					method: 'POST',
					data: combineRequsetData('JA_LIST', tmpData),
					header:{
						'Content-Type':'text/xml'
					},
					success: (res) => {
						this.companyList = res.data.map(item => {
							return item.FName
						})
						this.companyIDList = res.data.map(item => {
							return item.FItemID
						})
						this.FCompany = res.data[0].FItemID
					},
					fail: (err) => {
						console.log('request fail', err);
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						});
					}
				})
			}
		}
	}
</script>

<style>
	.container {
		padding: 20px 10px;
		font-size: 14px;
	}
	.itemBar{
		width: calc(100% - 20px);
		height: 50px;
		line-height: 50px;
		padding: 0px 10px;
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
</style>
