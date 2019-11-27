<template>
	<view class="container">
		<view class="logo__container"><image src="../../../static/logo_h.png" mode="scaleToFill"></image></view>
		<view class="form__container">
			<form @submit="submitForm">
				<view class=""><input name="phoneNumber" type="text" value="" placeholder="请输入手机号码" /></view>
				<view class="code_container">
					<input name="verCode" type="text" value="" placeholder="请输入验证码" />
					<button type="default" class="code__button">获取验证码</button>
				</view>
				<button type="primary" form-type="submit">绑定手机</button>
			</form>
		</view>
	</view>
</template>

<script>
import * as constants from '@/common/Constant';

export default {
	data() {
		return {
			phoneNumber: '',
			openid: ''
		};
	},
	methods: {
		submitForm: function(e) {
			this.phoneNumber = e.target.value.phoneNumber;
			let openid = uni.getStorageSync('openid');
			console.log({openid});
			console.log({phoneNumber: this.phoneNumber});
			uni.request({
				url: constants.baseUrl + '/wechat/users/account/bind',
				data: {
					openid: openid,
					phoneNumber: this.phoneNumber
				},
				method: 'PUT',
				header: {
					'content-type': 'application/json'
				},
				success: res => {
					console.log({
						loginRst: res
					});
					uni.setStorage({
						key: 'phoneNumber',
						data: this.phoneNumber,
						success: () => {
							uni.reLaunch({
								url: '/pages/passport/userinfo/userinfo'
							});
						}
					});
				}
			});
		}
	},
	onLoad: function(option) {
		console.log(option.openid);
		this.$data.openid = option.openid;
	}
};
</script>

<style>
.container {
	width: 100%;
}

.logo__container {
	display: flex;
	justify-content: center;
	margin: 100rpx 0;
}

.logo__container > image {
	width: 300rpx;
	height: 150rpx;
}

.form__container {
	justify-content: flex-start;
	margin: 28rpx;
}

.input__label {
	font-size: 28rpx;
	color: rgba(0, 0, 0, 0.85);
}

.form__container input {
	font-size: 28rpx;
	margin-top: 16rpx;
	color: rgba(0, 0, 0, 0.85);
}

.code_container {
	width: 100%;
	display: inline-flex;
	justify-content: flex-start;
}

.code_container input {
	flex: 4;
}

.code_container button {
	flex: 1;
	margin-right: 28rpx;
}

.code__button {
	background: #00a95f;
	color: #fff;
	margin-top: 28rpx;
	margin-right: 28rpx;
	font-size: 24rpx;
}
button[type='primary'] {
	background: #1b9d92;
	color: #ffffff;
	margin-top: 24rpx;
}
</style>
