<template>
	<view>
		<view v-if="data.length === 0" class="nodata__box"><text>您还没有诊疗记录</text></view>
		<view class="card" v-if="data.length > 0" v-for="(item, index) in data" :key="index">
			<view class="card__title">
				<text>评估日期</text>
				<text>{{ item.date }}</text>
			</view>
			<view class="card__body">
				<view class="card__item">
					<text class="card__label">诊断类目</text>
					<text>{{ item.productType.name }}</text>
				</view>
				<view class="card__item order__item">
					<text class="card__label">消费项目</text>
					<text v-for="(order, idx) in item.orderItemSet" :key="idx">{{ order.productName }} x {{ order.quantity }}</text>
				</view>
			</view>
			<view class="card__footer"><button type="primary" class="details__btn">查看详情</button></view>
		</view>
	</view>
</template>

<script>
import format from '@/node_modules/date-fns/format';
import * as constants from '@/common/Constant';

export default {
	data() {
		return {
			data: []
		};
	},
	methods: {
		loadData: function() {
			let userInfo = uni.getStorageSync('userInfo');
			let token = uni.getStorageSync('id_token');
			uni.request({
				url: constants.baseUrl + '/diagnosis/' + userInfo.id,
				method: 'GET',
				header: {
					'content-type': 'application/json',
					Authorization: 'Bearer ' + token
				},
				success: rst => {
					this.data = rst.data;
					if (this.data) {
						this.data.forEach(item => {
							let date = format(new Date(item.createdDate), 'yyyy-MM-dd');
							item.date = date;
						});
					}
				}
			});
		}
	},
	onLoad: function() {
		this.loadData();
	}
};
</script>

<style>
.nodata__box {
	margin-top: 24rpx;
	text-align: center;
	color: rgba(0, 0, 0, 0.85);
}
.card {
	font-size: 24rpx;
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
	/* border-bottom: 4rpx solid #1B9D92; */
	padding: 24rpx 0 0 24rpx;
	font-size: 28rpx;
	font-weight: 500;
	color: rgba(0, 0, 0, 0.85);
}
.card__title > text:first-child {
	padding-right: 24rpx;
}
.card__body {
	padding: 24rpx;
	font-weight: 400;
}
.card__item {
	display: flex;
	margin: 12rpx 0;
}
.card__label {
	font-weight: 500;
}
.card__item > text:first-child {
	padding-right: 24rpx;
}
/* .card__item > text:last-child {
	color: rgba(0, 0, 0, 0.65);
} */
.order__item > text:last-child {
	padding-right: 24rpx;
}
button[type='primary'] {
	background: #1b9d92;
	color: #ffffff;
	width: 160rpx;
	height: 60rpx;
	font-size: 24rpx;
	float: right;
	margin-right: 24rpx;
}
</style>
