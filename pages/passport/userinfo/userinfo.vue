<template>
	<view class="container">
		<form @submit="formSubmit">
			<view class="uni-form-item uni-column">
				<view class="title input-title">姓名</view>
				<input class="uni-input" name="userName" placeholder="姓名!" />
			</view>
			<view class="uni-form-item uni-column">
				<view class="title input-title">性别</view>
				<view class="uni-input">
					<picker @change="bindPickerChange" :value="index" :range="array" range-key="label">
						<view>{{ array[index].label }}</view>
					</picker>
				</view>
			</view>
			<view class="uni-form-item uni-column">
				<view class="title input-title">生日</view>
				<view style="flex: 8; margin-left: -12rpx;">
					<picker mode="date" :value="date" @change="bindDateChange">
						<view class="uni-input">{{ date }}</view>
					</picker>
				</view>
			</view>
			<view class="uni-form-item uni-column">
				<view class="title">个人症状</view>
				<textarea class="uni-textarea" name="symptom" placeholder="个人症状!" />
			</view>
			<view class="uni-form-item uni-column">
				<view class="title">病史</view>
				<textarea class="uni-textarea" name="pastMedicalHistory" placeholder="病史!" />
			</view>
			<view class="uni-form-item uni-column">
				<view class="title">诉求</view>
				<textarea class="uni-textarea" name="appeal" placeholder="诉求" />
			</view>
			<view class="uni-form-item uni-column">
				<view class="title">禁忌</view>
				<textarea class="uni-textarea" name="taboo" placeholder="禁忌" />
			</view>
			<view><button type="primary" class="submit__btn" form-type="submit">完善信息</button></view>
		</form>
	</view>
</template>

<script>
import * as constants from '@/common/Constant';
export default {
	data() {
		const currentDate = this.getDate({
			format: true
		});
		return {
			array: [{ label: '男', value: 1 }, { label: '女', value: 0 }],
			index: 0,
			gender: 1,
			date: currentDate
		};
	},
	methods: {
		getDate() {
			const date = new Date();
			let year = date.getFullYear();
			let month = date.getMonth() + 1;
			let day = date.getDate();
			year = year - 30;
			month = month > 9 ? month : '0' + month;
			day = day > 9 ? day : '0' + day;
			return `${year}-${month}-${day}`;
		},
		bindPickerChange: function(e) {
			console.log('picker发送选择改变，携带值为', e.target.value);
			this.index = e.target.value;
			this.gender = this.array[this.index].value;
			console.log({ gender: this.gender });
		},
		bindDateChange: function(e) {
			this.date = e.target.value;
			console.log({ date: this.date });
		},
		formSubmit: function(e) {
			var formdata = e.detail.value;
			console.log('form发生了submit事件，携带数据为：' + JSON.stringify(formdata));
			let token = uni.getStorageSync('id_token');
			formdata.customerType = 'MEMBER';
			formdata.openId = uni.getStorageSync('openid');
			formdata.phoneNumber = uni.getStorageSync('phoneNumber');
			formdata.fromOrigin = '微信';
			formdata.gender = this.gender;
			formdata.birthday = this.date;
			uni.request({
				url: constants.baseUrl + '/wechat/user/improve',
				method: 'PUT',
				header: {
					'content-type': 'application/json'
				},
				data: formdata,
				success: rst => {
					// TODO: token
					uni.request({
						url: constants.baseUrl + '/authenticate',
						data: {
							username: formdata.phoneNumber,
							password: formdata.openId
						},
						method: 'POST',
						header: {
							'content-type': 'application/json'
						},
						success: res => {
							console.log({
								tokenRst: res
							});
							uni.setStorage({
								key: 'id_token',
								data: res.data.id_token,
								success: () => {
									uni.showToast({
										title: '个人信息更新成功',
										duration: 2000,
										success: () => {
											uni.reLaunch({
												url: '/pages/account/home/home'
											});
										}
									});
								}
							});
						}
					});
				}
			});
		}
	}
};
</script>

<style>
.container {
	font-size: 24rpx;
	padding: 24rpx;
}
.uni-form-item {
	display: flex;
	align-items: center;
	margin-bottom: 24rpx;
}
.title {
	font-size: 1.1em;
	flex: 2;
}
.input-title {
	line-height: 48rpx;
}
.uni-input {
	margin-left: 24rpx;
	flex: 8;
	border: 1px solid #f5f5f5;
	border-radius: 8rpx;
	padding: 16rpx 0 16rpx 24rpx;
}
.uni-textarea {
	margin-left: 24rpx;
	flex: 8;
	height: 3em;
	border: 1px solid #f5f5f5;
	border-radius: 8rpx;
	padding: 24rpx;
}
button[type='primary'] {
	color: #ffffff;
	background-color: #1b9d92;
}
</style>
