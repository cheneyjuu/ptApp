<template>
	<view>
		<!-- #ifdef MP-WEIXIN -->
		<view v-if="isCanUse">
			<view>
				<view class="header"><image src="../../../static/logo_h.png"></image></view>
				<view class="content">
					<view>申请获取以下权限</view>
					<text>获得你的公开信息(昵称，头像、地区等)</text>
				</view>

				<button class="bottom" type="primary" open-type="getUserInfo" withCredentials="true" lang="zh_CN" @getuserinfo="wxGetUserInfo">授权登录</button>
			</view>
		</view>
		<!-- #endif -->
	</view>
</template>

<script>
import * as constants from '@/common/Constant';

export default {
	data() {
		return {
			SessionKey: '',
			OpenId: '',
			nickName: null,
			avatarUrl: null,
			isCanUse: uni.getStorageSync('isCanUse') || true, //默认为true
			userInfo: {
				openid: '',
				nickname: '',
				gender: '',
				country: '',
				province: '',
				city: '',
				avatarUrl: ''
			},
			phoneNumber: ''
		};
	},
	methods: {
		//第一授权获取用户信息===》按钮触发
		wxGetUserInfo() {
			let _this = this;
			uni.getUserInfo({
				provider: 'weixin',
				success: function(infoRes) {
					console.log({
						userinfo: infoRes
					});
					_this.nickName = infoRes.userInfo.nickName;
					_this.avatarUrl = infoRes.userInfo.avatarUrl;
					try {
						uni.setStorageSync('isCanUse', false); //记录是否第一次授权  false:表示不是第一次授权
						_this.login();
					} catch (e) {}
				},
				fail(res) {}
			});
		},

		//登录
		login() {
			let _this = this;
			const isCanUse = uni.getStorageSync('isCanUse');
			console.log({ isCanUse });
			if (isCanUse === false) {
				uni.showLoading({
					title: '登录中...'
				});
				// 1.wx获取登录用户code
				uni.login({
					provider: 'weixin',
					success: function(loginRes) {
						let code = loginRes.code;
						if (!_this.isCanUse) {
							//非第一次授权获取用户信息
							uni.getUserInfo({
								provider: 'weixin',
								success: function(infoRes) {
									console.log({
										getUserInfo: infoRes
									});
									//获取用户信息后向调用信息更新方法
									// _this.$data.userInfo.nickname = infoRes.userInfo.nickName;
									// _this.$data.userInfo.avatarUrl = infoRes.userInfo.avatarUrl;
									_this.gender = infoRes.userInfo.gender;
									_this.country = infoRes.userInfo.country;
									_this.province = infoRes.userInfo.province;
									_this.city = infoRes.userInfo.city;
									// _this.updateUserInfo(); //调用更新信息方法
								}
							});
						}

						//2.将用户登录code传递到后台置换用户SessionKey、OpenId等信息
						uni.request({
							url: constants.baseUrl + '/login/app',
							data: {
								code: code
							},
							method: 'GET',
							header: {
								'content-type': 'application/json'
							},
							success: res => {
								console.log({
									loginRst: res
								});
								uni.setStorageSync('openid', res.data.openid);
								_this.openid = res.data.openid;
								//openId、或SessionKdy存储//隐藏loading
								let phoneNumber = uni.getStorageSync('phoneNumber');
								console.log({ phoneNumber });
								if (phoneNumber) {
									uni.reLaunch({
										url: '/pages/account/home/home'
									});
								} else {
									_this.bindAccount();
								}
								uni.hideLoading();
							}
						});
					}
				});
			}
		},
		bindAccount: function() {
			uni.reLaunch({
				url: '/pages/passport/register/register?openid=' + this.userInfo.openid
			});
		},
		//向后台更新信息
		updateUserInfo() {
			let _this = this;
			uni.request({
				url: constants.baseUrl + '/wechat/users', //服务器端地址
				data: {
					openid: _this.openid,
					nickname: _this.nickname,
					gender: _this.gender,
					country: _this.country,
					province: _this.province,
					city: _this.city,
					avatarUrl: _this.avatarUrl
				},
				method: 'PUT',
				header: {
					'content-type': 'application/json'
				},
				success: res => {
					console.log({ result: res });
					if (res.statusCode == 200) {
						if (res.data.hasPhoneNumber) {
							uni.setStorage({
								key: 'phoneNumber',
								data: res.data.phoneNumber
							});
						} else {
							uni.reLaunch({
								//信息更新成功后跳转到小程序首页
								url: '/pages/passport/register/register?openid=' + _this.$data.userInfo.openid
							});
						}
					}
				}
			});
		}
	},
	onLoad() {
		//默认加载
		this.login();
	}
};
</script>

<style>
.header {
	margin: 90rpx 0 90rpx 50rpx;
	border-bottom: 1px solid #ccc;
	text-align: center;
	width: 650rpx;
	height: 300rpx;
	line-height: 450rpx;
}

.header image {
	width: 300rpx;
	height: 150rpx;
}

.content {
	margin-left: 50rpx;
	margin-bottom: 90rpx;
	font-size: 32rpx;
}

.content text {
	display: block;
	color: #9d9d9d;
	margin-top: 40rpx;
	font-size: 28rpx;
}

.bottom {
	border-radius: 80rpx;
	margin: 70rpx 50rpx;
	font-size: 35rpx;
}
button[type='primary'] {
	background: #1b9d92;
	color: #ffffff;
	margin-top: 24rpx;
	font-size: 32rpx;
}
</style>
