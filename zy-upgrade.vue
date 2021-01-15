<template>
	<view class="zy-modal" :class="dshow?'show':''">
		<view class="zy-dialog" style="background-color: transparent;">
			<view class="padding-top text-white" :class="'zy-upgrade-topbg-'+maincolor">
				<view>
					<text class="zy-upgrade-title">
						发现新版本
					</text>
				</view>
				<text class="flex-wrap">{{version}}</text>
			</view>
			<view class="padding-xl bg-white text-left">
				<scroll-view style="max-height: 200rpx;" scroll-y="auto" v-if="!update_flag">
					<text>{{update_tips}}</text>
				</scroll-view>
				<view class="zy-progress radius striped active" v-if="update_flag">
					<view :class="'bg-'+maincolor" :style="'width: '+update_process+'%;'">
						{{update_process}}
					</view>
				</view>
			</view>
			<view class="zy-bar bg-white justify-end">
				<view class="action" v-if="!update_flag">
					<button class="zy-btn" :class="'bg-'+maincolor" @click="upgrade_checked">确认升级</button>
					<button class="zy-btn margin-left" :class="'line-'+maincolor"
					v-if="!forceupgrade"
					 @click="upgrade_cancel">取消升级</button>
				</view>
				<view class="action text-center" v-if="update_flag&&!forceupgrade">
					<button class="zy-btn" :class="'bg-'+maincolor" @click="upgrade_break">中断升级</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		name: 'ZyUpgrade',
		props: {
			width: {
				type: Number,
				default: 500
			},
			height: {
				type:Number,
				default: 800
			},
			maincolor: {
				type: String,
				default: 'green'
			},
			updateurl: {
				type:String,
				default: ''
			},
			h5preview:{	//H5界面下是否预览升级
				type: Boolean,
				default: false
			},
			oldversion: {	//如果是H5，为了方便测试，可以传入一个旧版本号进来。
				type: String,
				default: ''
			},
			appstoreflag: {	//是否启用appstore升级，如果启用，由服务端返回appstore的地址
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				update_flag: false, //点击升级按钮后，显示进度条
				dshow: false,
				update_process: 0,
				downloadTask: [],
				updated2version: '',
				version_url: '',
				update_tips: '',
				forceupgrade: false,
				currentversion: this.oldversion,
				versionname: ''
			}
		},
		mounted() {
			if(this.h5preview){
				console.log("检测升级")
				this.check_update()
			}
		},
		computed:{
			version(){
				let retversion = ''
				retversion = this.currentversion + (this.currentversion!=''&&this.updated2version!=''?'->':'')+this.updated2version
				return retversion
			}
		},
		methods:{
			//检测升级
			check_update(){
				let that = this
				// #ifdef APP-PLUS
				plus.runtime.getProperty(plus.runtime.appid, function(widgetInfo) { 
					console.log(widgetInfo.version,widgetInfo.versionCode,widgetInfo.name)
					that.currentversion = widgetInfo.version
					that.versionname = widgetInfo.name
				});  
				// #endif
				uni.showLoading({
					title: '',
					mask: false
				});
				let platform = uni.getSystemInfoSync().platform
				uni.request({  
					url: that.updateurl,  
					data: {
						method: "upgrade",
						version: that.currentversion,  
						name: that.versionname,
						ts:'123',
						transid:'123',
						sign:'123',
						platform: platform
					},  
					success: (result) => {
						uni.hideLoading()
						let data = result.data
						if(data.code == 100){
							console.log(data)
								//提示升级
							if(data.data.update_flag == 1){
								that.dshow = true
								that.update_tips = data.data.update_tips
								that.forceupgrade = data.data.forceupdate==1
								that.version_url = data.data.update_url
								//that.currentversion = widgetInfo.version
								that.updated2version = data.data.version
							}else{
								
							}
						}else{
							uni.showToast({
								title: '请求升级出错：'+data.msg,
								icon:'none'
							});
						}
					}  
				});  
			},
			//点击开始升级按钮，开始升级
			upgrade_checked:function(){
				this.update_flag = true
				this.updateversion()
			},
			//点击取消升级按钮，取消升级
			upgrade_cancel:function(){
				this.dshow = false
			},
			//升级过程中，点击中断升级按钮，中断升级
			upgrade_break: function(){
				this.downloadTask.abort()
				this.update_flag = false
			},
			//升级下载apk安装包的具体处理业务逻辑
			updateversion: function(){
				let platform = uni.getSystemInfoSync().platform
				console.log("操作系统：",platform)
				if(platform == 'ios' && this.appstoreflag){
					//如果启用ios appstore升级，则打开appstore
					that.dshow = false
					console.log("跳转至appstore")
					plus.runtime.launchApplication({
					    action: that.version_url
					}, function(e) {
					    uni.showToast({
					    	title: '打开appstore失败',
							icon:'none'
					    });
					});
				}else{
					let that = this				
					this.update_confirm = true
					this.downloadTask = uni.downloadFile({
						url: that.version_url,
						success:function(res){
							if(res.statusCode == 200){
								//开始安装
								plus.runtime.install(res.tempFilePath, {
									force: false  
								}, function() {  
									console.log('install success...');  
									plus.runtime.restart();  
								}, function(e) {  
									console.error('install fail...');  
								});  
							}else{
								uni.showToast({
									title: '下载失败，网络错误',
									icon:'none'
								});
							}
						},
						fail:function(e) {
							console.log("下载失败",e)
							uni.showToast({
								title: '下载失败:'+e.errMsg,
								icon:'none'
							});
							this.update_flag = false
						},
						complete:function(){
						}
					})
					this.downloadTask.onProgressUpdate(function(res){
						that.update_process = res.progress
					})
				}
			},
		}
	}
</script>

<style scoped>
	@import url("static/main.css");
	.zy-upgrade-topbg-green {
		background-image: url('static/images/green.png');
		background-size: 100% 100%;
		background-repeat: no-repeat;
		height: 290rpx;
	}
	.zy-upgrade-topbg-red {
		background-image: url('static/images/red.png');
		background-size: 100% 100%;
		background-repeat: no-repeat;
		height: 290rpx;
	}
	.zy-upgrade-topbg-pink {
		background-image: url('static/images/pink.png');
		background-size: 100% 100%;
		background-repeat: no-repeat;
		height: 290rpx;
	}
	.zy-upgrade-topbg-yellow {
		background-image: url('static/images/yellow.png');
		background-size: 100% 100%;
		background-repeat: no-repeat;
		height: 290rpx;
	}
	.zy-upgrade-topbg-blue {
		background-image: url('static/images/blue.png');
		background-size: 100% 100%;
		background-repeat: no-repeat;
		height: 290rpx;
	}
	.zy-upgrade-title {
		font-size: 50rpx;
		color: white;
	}
</style>
