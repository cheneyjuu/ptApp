<template>
	<view class="container">
		<view class="card" v-for="(item, index) in data" :key="index">
			<view class="card__title">
				<text class="iconfont">&#xe606;</text>
				<text>{{item.date}} {{item.timeSlot}}</text>
				<text>{{item.address}}</text>
			</view>
			<view class="card__body">
				<view class="body__item">
					<text>预约单号: </text>
					<text>{{item.tradeNo}}</text>
				</view>
				<view class="body__item">
					<text>预约类目: </text>
					<text>{{item.typeName}}</text>
				</view>
			</view>
			<view class="card__footer">
				<!-- <button type="primary" plain="true" class="details__btn">查看详情</button> -->
				<button type="warn" plain="true" class="details__btn">取消预约</button>
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
		loadData: function() {
			let token = uni.getStorageSync('id_token');
			uni.request({
				url: constants.baseUrl + '/appointments/patient',
				method: 'GET',
				header: {
					'content-type': 'application/json',
					Authorization: 'Bearer ' + token
				},
				success: (rst) => {
					this.data = rst.data;
					console.log({rst: this.data});
				}
			})
		}
	},
	onLoad: function() {
		this.loadData();
	}
};
</script>

<style>
@font-face {
	font-family: 'iconfont';
	src: url('~@/static/iconfonts/iconfont.ttf?t=1572612544444') format('truetype'); /* iOS 4.1- */
}

.iconfont {
	font-family: 'iconfont' !important;
	font-size: 16px;
	font-style: normal;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
}
.card {
	background: #fff;
	border-radius: 2px;
	display: inline-block;
	height: 300rpx;
	margin: 24rpx;
	position: relative;
	width: 702rpx;
	box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
	transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
}
.card__title {
	display: flex;
	font-size: 28rpx;
	font-weight: 500;
	padding: 24rpx;
	align-items: flex-end;
}
.card__title > text:nth-child(1) {
	color: #1b9d92;
	padding-right: 16rpx;
	align-self: center;
	flex: 1;
}
.card__title > text:nth-child(2) {
	color: rgba(0, 0, 0, 0.85);
	padding-right: 24rpx;
	flex: 4;
}
.card__title > text:nth-child(3) {
	color: rgba(0, 0, 0, 0.45);
	font-size: 22rpx;
	flex: 5;
}
.card__body {
	padding: 0 0 24rpx 24rpx;
	font-size: 24rpx;
	color: rgba(0, 0, 0, 0.65);
}
.card__body>.body__item {
	display: flex;
	justify-content: flex-start;
	margin-bottom: 16rpx;
}
.body__item>text:nth-child(1) {
	padding-right: 24rpx;
}
button[type=primary][plain] {
	border: 1px solid #1b9d92;
	color: #1b9d92;
	height: 60rpx;
	font-size: 24rpx;
}
button[type=warn][plain] {
	width: 160rpx;
	height: 60rpx;
	font-size: 24rpx;
	float: right;
	margin-right: 24rpx;
}
</style>
