<template>
	<view>
		<view class="picker__container">
			<view class="picker__box">
				<picker @change="onPoiChange" :value="poiIndex" :range="pois" range-key="name" mode="selector">
					<view>{{ pois[poiIndex].name }}</view>
				</picker>
			</view>
			<view class="picker__box">
				<picker @change="onTypeChange" :value="typeIndex" :range="types" range-key="title">
					<view>{{ types[typeIndex].title }}</view>
				</picker>
			</view>
		</view>

		<view class="card">
			<swiper class="swiper" :display-multiple-items="5" :indicator-dots="false" :autoplay="false">
				<swiper-item class="swiper-item" v-for="(item, index) in listDate" :key="index" @tap="chooseDate(index)" :class="{ active: index === currentIndex }">
					<view class="date-label">{{ item.date }}</view>
					<view class="week-label">{{ item.week }}</view>
				</swiper-item>
			</swiper>
		</view>
		<view class="time__box">
			<view class="time__item" v-for="(item, index) in times" :key="index" @tap="chooseTime(item, index)" :class="{ active__time: index === currentTimeIndex }">
				<text>{{ item }}</text>
			</view>
		</view>
		<view class="kfs__box" v-if="typeIndex === 0"><view class="user__box">请选择类目</view></view>
		<view class="kfs__box" v-if="typeIndex === '1'">
			<view
				class="user__box"
				v-for="(user, index) in kfsList"
				:key="index"
				@tap="chooseKfs(user)"
				:class="{
					selected: selectedKfsData.userName === user.userName && user.active,
					inactive: !user.active
				}"
			>
				<text>{{ user.userName }}</text>
			</view>
		</view>
		<view class="kfs__box" v-if="typeIndex === '2'">
			<view
				class="user__box"
				v-for="(user, index) in tsnsList"
				:key="index"
				@tap="chooseTsns(user)"
				:class="{
					selected: selectedTsnsData.userName === user.userName && user.active,
					inactive: !user.active
				}"
			>
				<text>{{ user.userName }}</text>
			</view>
		</view>

		<view class="kfs__box">
			<view class="title">同行人数量</view>
			<input class="uni-input" type="text" @input="onKeyInput" maxlength="10" placeholder="多个人用逗号分开" />
		</view>

		<view class="btn__box"><button type="primary" class="submit__btn" @tap="submitForm()">确定预约</button></view>
	</view>
</template>

<script>
import format from '@/node_modules/date-fns/format';
import getMonth from '@/node_modules/date-fns/getMonth';
import addDays from '@/node_modules/date-fns/addDays';
import getDate from '@/node_modules/date-fns/getDate';
import getDay from '@/node_modules/date-fns/getDay';
import * as constants from '@/common/Constant';

export default {
	data() {
		return {
			listDate: [],
			pois: [{ name: '请选择门店' }],
			types: [{ title: '请选择类目' }],
			currentIndex: 0,
			currentTimeIndex: '',
			poiIndex: 0,
			typeIndex: 0,
			poiCode: '',
			token: '',
			workDate: new Date(),
			times: constants.times,
			selectedTime: '',
			tsnsList: [],
			kfsList: [],
			selectedKfsData: {},
			selectedTsnsData: {},
			peers: '',
			btnLoading: false
		};
	},
	methods: {
		onKeyInput: function(event) {
			this.peers = event.target.value;
			console.log(this.peers);
		},
		onPoiChange: function(e) {
			console.log({ e });
			this.poiIndex = e.target.value;
			const poi = this.pois[this.poiIndex];
			this.poiCode = poi.code;
		},
		onTypeChange: function(e) {
			console.log({ e });
			this.typeIndex = e.target.value;
		},
		getToken: function() {
			let that = this;
			uni.getStorage({
				key: 'id_token',
				success: function(res) {
					that.token = res.data;
					that.loadPois(res.data);
					that.loadTypes(res.data);
					that.loadAllSchedules(res.data);
				}
			});
		},
		// load pois
		loadPois: function(token) {
			let that = this;
			uni.request({
				url: constants.baseUrl + '/poi',
				method: 'GET',
				header: {
					'content-type': 'application/json',
					Authorization: 'Bearer ' + token
				},
				success: res => {
					res.data.forEach(poi => {
						that.pois = [...that.pois, poi];
					});
				}
			});
		},

		loadTypes: function(token) {
			let that = this;
			uni.request({
				url: constants.baseUrl + '/product/types',
				method: 'GET',
				header: {
					'content-type': 'application/json',
					Authorization: 'Bearer ' + token
				},
				success: res => {
					res.data.data.forEach(type => {
						that.types = [...that.types, type];
					});
				}
			});
		},

		chooseDate: function(e) {
			let that = this;
			that.currentIndex = e;
			that.workDate = addDays(new Date(), that.currentIndex);
			console.log(that.workDate);
		},

		chooseTime: function(time, index) {
			let that = this;
			console.log({ time }, { index });
			that.currentTimeIndex = index;
			that.selectedTime = time;

			that.tsnsList.forEach(tsnUser => {
				tsnUser.active = true;
				if (this.contains(tsnUser.timeSlot, time)) {
					tsnUser.timeSlot.forEach(timeSlot => {
						if (timeSlot.slot.trim() === time && timeSlot.busy === true) {
							tsnUser.active = false;
						}
					});
				} else {
					tsnUser.active = false;
				}
			});

			that.kfsList.forEach(kfsUser => {
				if (this.contains(kfsUser.timeSlot, time)) {
					kfsUser.active = true;
					kfsUser.timeSlot.forEach(timeSlot => {
						if (timeSlot.slot.trim() === time && timeSlot.busy === true) {
							kfsUser.active = false;
						}
					});
				} else {
					kfsUser.active = false;
				}
			});
		},

		chooseKfs: function(user) {
			// 每次只能选择一个专家
			this.selectedTsnsData = {};
			if (this.selectedKfsData && this.selectedKfsData === user) {
				this.selectedKfsData = {};
			} else {
				this.selectedKfsData = user;
			}
		},

		chooseTsns: function(user) {
			this.selectedKfsData = {};
			if (this.selectedTsnsData && this.selectedTsnsData === user) {
				this.selectedTsnsData = {};
			} else {
				this.selectedTsnsData = user;
			}
		},

		contains: function(timeSlot, time) {
			let isContains = false;
			timeSlot.forEach(item => {
				if (item.slot.trim() === time) {
					isContains = true;
					return isContains;
				}
			});
			return isContains;
		},

		initDate: function() {
			let today = new Date();
			for (let i = 0; i < 30; i++) {
				let dateStr;
				if (i === 0) {
					dateStr = '今天';
				} else if (i === 1) {
					dateStr = '明天';
				} else if (i === 2) {
					dateStr = '后天';
				} else {
					dateStr = getMonth(addDays(today, i)) + 1 + '.' + getDate(addDays(today, i));
				}
				const dateObj = {
					week: this.convertWeekNumber(getDay(addDays(today, i))),
					date: dateStr,
					month: getMonth(addDays(today, i)) + 1 + '月'
				};
				this.$data.listDate.push(dateObj);
			}
			console.log({ date: this.$data.listDate });
		},
		convertWeekNumber: function(weekNumber) {
			let weekStr;
			switch (weekNumber) {
				case 1:
					weekStr = '周一';
					break;
				case 2:
					weekStr = '周二';
					break;
				case 3:
					weekStr = '周三';
					break;
				case 4:
					weekStr = '周四';
					break;
				case 5:
					weekStr = '周五';
					break;
				case 6:
					weekStr = '周六';
					break;
				case 0:
					weekStr = '周日';
					break;
			}
			return weekStr;
		},
		loadAllSchedules: function(token) {
			let that = this;
			this.tsnsList = [];
			this.kfsList = [];
			const date = addDays(new Date(), that.currentIndex);
			uni.request({
				url: constants.baseUrl + '/schedules/employees',
				method: 'POST',
				header: {
					'content-type': 'application/json',
					Authorization: 'Bearer ' + token
				},
				data: {
					date
				},
				success: res => {
					res.data.forEach(item => {
						if (item.ptUsers && item.ptUsers.length > 0) {
							item.ptUsers.forEach(ptUser => {
								const kfs = {
									id: ptUser.id,
									userName: ptUser.userName,
									gender: ptUser.gender,
									timeSlot: JSON.parse(item.timeSlot),
									active: true
								};
								that.kfsList.push(kfs);
							});
						}
						if (item.tsnUsers && item.tsnUsers.length > 0) {
							item.tsnUsers.forEach(tsnUser => {
								const tsns = {
									id: tsnUser.id,
									userName: tsnUser.userName,
									gender: tsnUser.gender,
									timeSlot: JSON.parse(item.timeSlot),
									active: true
								};
								that.tsnsList.push(tsns);
							});
						}
					});
				}
			});
		},

		submitForm: function() {
			let that = this;
			this.btnLoading = true;
			
			let doctorIds;
			if (this.selectedKfsData && this.selectedKfsData.id) {
				doctorIds = [this.selectedKfsData.id];
			}
			if (this.selectedTsnsData && this.selectedTsnsData.id) {
				doctorIds = [this.selectedTsnsData.id];
			}

			if (!this.poiCode) {
				uni.showToast({
					title: '请选择门店',
					icon: 'none',
					duration: 2000
				});
				return;
			}

			if (!this.typeIndex) {
				uni.showToast({
					title: '请选择预约类目',
					icon: 'none',
					duration: 2000
				});
				return;
			}
			if (!this.selectedTime) {
				uni.showToast({
					title: '请选择预约时间',
					icon: 'none',
					duration: 2000
				});
				return;
			}
			if (!doctorIds) {
				uni.showToast({
					title: '请选择预约专家',
					icon: 'none',
					duration: 2000
				});
				return;
			}
			console.log('should not be show');
			let token = uni.getStorageSync('id_token');
			console.log({ token });
			let phoneNumber = uni.getStorageSync('phoneNumber');

			let reserveDate = format(this.workDate, 'yyyy-MM-dd');

			uni.showModal({
				title: '操作提醒',
				content: '确定预约' + reserveDate + '服务?',
				success: function(res) {
					console.log({ res });
					if (res.confirm) {
						uni.request({
							url: constants.baseUrl + '/appointments',
							method: 'POST',
							header: {
								'content-type': 'application/json',
								Authorization: 'Bearer ' + token
							},
							data: {
								phoneNumber,
								doctorIds,
								poiCode: that.poiCode,
								typeId: that.typeIndex,
								timeSlot: that.selectedTime,
								workDate: that.workDate,
								peers: that.peers
							},
							success: res => {
								that.btnLoading = false;
								uni.showToast({
									title: '预约成功',
									duration: 2000,
									success: () => {
										uni.reLaunch({
											url: '/pages/account/home/home'
										});
									}
								});
								console.log({ success: res });
							}
						});
					} else if (res.cancel) {
						console.log('用户点击取消');
					}
				}
			});
		}
	},
	onLoad: function() {
		console.log(format(new Date(), 'yyyy-MM-dd'));
		this.getToken();
		this.initDate();
	}
};
</script>

<style>
page {
	background: #f5f5f5;
}
.card {
	font-size: 24rpx;
	background: #fff;
	width: 100%;
	height: 120rpx;
	box-shadow: 0 1px 2px rgba(0, 0, 0, 0.24);
	transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
	padding-left: 0;
}
.picker__container {
	margin-top: 24rpx;
}
.picker__box {
	margin-bottom: 12px;
	font-size: 24rpx;
	background: #fff;
	padding: 24rpx;
}
.swiper {
	width: 100%;
	height: 90rpx;
	padding-top: 24rpx;
}
.swiper-item {
	height: 80rpx;
	text-align: center;
	font-weight: 500;
	font-size: 28rpx;
}
.time__box {
	display: flex;
	flex-wrap: wrap;
	width: 100%;
	padding: 24rpx;
	background: #ffffff;
	padding-left: 0;
}
.kfs__box {
	display: flex;
	width: 100%;
	padding: 24rpx;
	background: #ffffff;
	padding-left: 0;
	margin-top: 24rpx;
	font-size: 24rpx;
}
.user__box {
	margin: 0 24rpx;
	height: 50rpx;
	line-height: 50rpx;
}
.user__box text {
	padding: 0 24rpx;
}
.time__item {
	width: 150rpx;
	height: 48rpx;
	margin-bottom: 24rpx;
	text-align: center;
}
.time__item text {
	font-size: 26rpx;
	color: rgba(0,0,0,0.65);
}
.active__time {
	background: #f74e28;
	border-radius: 8rpx;
	color: #ffffff;
	line-height: 48rpx;
}
.selected {
	color: #fff;
	background-color: #f74e28;
	border-radius: 8rpx;
}

.inactive {
	color: rgba(0, 0, 0, 0.25);
	background-color: #f5f5f5;
	border-color: #d9d9d9;
	border-radius: 8rpx;
}
.active {
	border-bottom: 2px solid #f74e28;
}
.active .date-label {
	color: #f74e28;
	font-size: 1.1em;
}
.active .week-label {
	color: #f74e28;
}
.title {
	line-height: 50rpx;
	padding: 0 24rpx;
}
.submit__btn {
	margin-top: 24rpx;
}

button[type='primary'] {
	color: #ffffff;
	background-color: #1b9d92;
}
.btn__box {
	margin: 0 24rpx;
}
</style>
