<template>
	<view>
		<view class="package__box" v-for="(item, index) in data" :key="index">
			<view class="title">{{ item.name }}</view>
			<view class="line"></view>
			<view class="price__box">
				<view class="rmb">￥</view>
				<view class="item__price">{{ item.price }}</view>
			</view>
			<view class="package__item" v-for="(charge, idx) in item.packageCharges" :key="idx">{{ charge.chargeName }} x {{ charge.count }}</view>
			<view class="btn__box">
				<button type="primary" form-type="submit">购买套餐</button>
			</view>
		</view>
	</view>
</template>

<script>
import * as constants from '@/common/Constant';
export default {
	data() {
		return {
			data: []
		};
	},
	methods: {
		loadPackages() {
			let that = this;
			const token = uni.getStorageSync('id_token');
			console.log({ token });
			uni.request({
				url: constants.baseUrl + '/packages',
				method: 'GET',
				header: {
					'content-type': 'application/json',
					Authorization: 'Bearer ' + token
				},
				success: res => {
					this.data = res.data;
					console.log(res);
				}
			});
		}
	},
	onLoad() {
		//默认加载
		this.loadPackages();
	}
};
</script>

<style>
.package__box {
	text-align: center;
	background: #fff;
	border-radius: 2px;
	display: inline-block;
	height: 600rpx;
	position: relative;
	width: 702rpx;
	box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
	margin: 24rpx;
}
.line {
	width: 691rpx;
	border: 6rpx solid #efefef;
	margin-bottom: 62rpx;
}
.title {
	height: 200rpx;
	line-height: 140rpx;
	font-weight: 500;
	font-size: 48rpx;
	color: #282c34;
}
.price__box {
	width: 120rpx;
	height: 120rpx;
	background: #26a69a;
	border-radius: 50%;
	text-align: center;
	/* line-height: 120rpx; */
	color: #fff;
	position: absolute;
	left: 271rpx;
	top: 140rpx;
	font-size: 24rpx;
}
.rmb {
	position: absolute;
	font-size: 32rpx;
	left: 36%;
}
.item__price {
	position: absolute;
	font-size: 44rpx;
	font-weight: 500;
	left: 20%;
	top: 34%;
}
.package__item {
	font-size: 24rpx;
	color: rgba(0, 0, 0, 0.65);
	margin: 12rpx;
}
.btn__box {
	position: relative;
}
button[type='primary'] {
	background: #1b9d92;
	color: #ffffff;
	margin-top: 24rpx;
	position: absolute;
	right: 24rpx;
	width: 180rpx;
	height: 60rpx;
	font-size: 24rpx;
}
</style>
