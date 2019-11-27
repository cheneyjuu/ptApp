<template>
	<view>
		<view class="user__box">
			<image src="../../../static/account_bg.jpg" mode="" class="background__image"></image>
			<view class="account__box">
				<image :src="userInfo.imageUrl" mode="aspectFit" class="header__image"></image>
				<text>{{userInfo.userName}} 欢迎您！</text>
			</view>
		</view>
		
		<uni-grid :column="3" :show-border="false" :square="true" @change="change">
			<uni-grid-item>
				<text class="iconfont online__reserve__icon">&#xe606;</text>
				<text class="text">在线预约</text>
			</uni-grid-item>
			<uni-grid-item>
				<text class="iconfont online__reserve__icon">&#xe665;</text>
				<text class="text">我的预约</text>
			</uni-grid-item>
			<uni-grid-item>
				<text class="iconfont online__reserve__icon">&#xe60b;</text>
				<text class="text">诊疗记录</text>
			</uni-grid-item>
			<uni-grid-item>
				<text class="iconfont online__reserve__icon">&#xe604;</text>
				<text class="text">预诊病历</text>
			</uni-grid-item>
			<uni-grid-item>
				<text class="iconfont online__reserve__icon">&#xe660;</text>
				<text class="text">康复之路</text>
			</uni-grid-item>
			<uni-grid-item>
				<text class="iconfont online__reserve__icon">&#xe82d;</text>
				<text class="text">投诉评价</text>
			</uni-grid-item>
			<uni-grid-item>
				<text class="iconfont online__reserve__icon">&#xe619;</text>
				<text class="text">我的账户</text>
			</uni-grid-item>
			<uni-grid-item>
				<text class="iconfont online__reserve__icon">&#xe63c;</text>
				<text class="text">账单记录</text>
			</uni-grid-item>
			<uni-grid-item>
				<text class="iconfont online__reserve__icon">&#xe603;</text>
				<text class="text">在线客服</text>
			</uni-grid-item>
		</uni-grid>
	</view>
</template>

<script>
import uniGrid from '@/components/uni-grid/uni-grid.vue';
import uniGridItem from '@/components/uni-grid-item/uni-grid-item.vue';
import * as constants from '@/common/Constant';

export default {
	components: { uniGrid, uniGridItem },
	data() {
		return {
			userInfo: {}
		};
	},
	methods: {
		change: function(e) {
			const idx = e.detail.index;
			console.log({ idx });
			if (idx === 0) {
				uni.navigateTo({
					url: '/pages/appointment/online/online'
				});
			}
			if (idx === 1) {
				uni.navigateTo({
					url: '/pages/appointment/myself/myself'
				});
			}
			if (idx === 2) {
				uni.navigateTo({
					url: '/pages/account/diagnosis/diagnosis'
				});
			}
		},
		loadUserInfo: function() {
			let that = this;
			let token = uni.getStorageSync('id_token');
			let phoneNumber = uni.getStorageSync('phoneNumber');
			console.log({phoneNumber});
			uni.request({
				url: constants.baseUrl + '/customers/phone?phoneNumber=' + phoneNumber,
				method:'GET',
				header: {
					'content-type': 'application/json',
					Authorization: 'Bearer ' + token
				},
				success: (rst) => {
					that.userInfo = rst.data;
					uni.setStorageSync('userInfo', rst.data);
				}
			})
		}
	},
	onLoad:function(){
		let userInfo = uni.getStorageSync('userInfo');
		this.userInfo = userInfo;
		if (!userInfo) {
			this.loadUserInfo();
		}
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

.icontousu:before {
	content: '\e82d';
}

.iconzhangdan:before {
	content: '\e63c';
}

.iconyuyue:before {
	content: '\e600';
}

.iconhealth_-report-me:before {
	content: '\e60b';
}

.iconmoon:before {
	content: '\e640';
}

.iconwodehuiyuanqia:before {
	content: '\e619';
}

.iconyuyue1:before {
	content: '\e606';
}

.iconSun:before {
	content: '\e660';
}

.iconmedical:before {
	content: '\e61a';
}

.iconkefu1:before {
	content: '\e603';
}

.iconyuyue2:before {
	content: '\e665';
}

.iconhuanzheduan-zhenliaofangan:before {
	content: '\e604';
}
.online__reserve__icon {
	font-size: 48rpx;
	padding-bottom: 16rpx;
	color: #1b9d92;
}
.background__image {
	width: 750rpx;
	height: 400rpx;
	position: relative;
}
.account__box {
	position: absolute;
	top: 220rpx;
	left: 32rpx;
}
.account__box text {
	width: 400rpx;
	color: #FFF;
	margin-left: 32rpx;
	top: 30rpx;
	position: absolute;
}
.header__image {
	width: 120rpx;
	height: 120rpx;
	border-radius: 60rpx;
}
</style>
