<template>
	<view>
		<!-- <Header title="缴费记录" isBack="yes"></Header> -->
		<!-- loading加载动画，type默认值是原子，love爱心，mask属性是遮罩 -->
		<!-- <zero-loading v-if="loading" type="pulse" mask></zero-loading> -->
		<view shadow="never" v-if="currentPatient.cardNumber"
			style="padding: .2rem;margin-bottom: .2rem;background-color: #fff;">
			<view class="card-row">
				<view class="patient-name">
					<span>{{ currentPatient.patientName | processingName }}</span>
				</view>
				<view class="change-patient-name">
					<button class="change-patient-name-btn" @click="switchPatient">切换就诊人</button>
				</view>
			</view>
			<view class="card-row">
				<view class="visit-number">就诊号：{{ currentPatient.cardNumber | processingcardNumber }}</view>
			</view>
		</view>
		<view class="record-container">
			<view class="record-item" v-for="item in listData" @click="goToDetail(item)">
				<img class="icon-success" src="../../static/icon-success.png">
				<view class="record-right">
					<view>
						<view>{{ item.deptName || '-'}}</view>
						<view class="item-bot">{{ item.patientName || '-' }}</view>
					</view>
					<view style="text-align: right">
						<view style="color: #EB7C00">￥{{ item.totalFee || 0 }}</view>
						<view class="item-bot">{{item.regDate | dateStr}}</view>
					</view>
				</view>
			</view>
			<view v-if="listData.length === 0" style="text-align: center;color: #999;">暂无记录</view>
		</view>
		<view v-if="showSwitchPatient" class="switch-patient-bg">
			<view class="switch-patient-container bg-white">
				<view class="switch-patient-title text-center border-bottom">
					切换就诊人
					<!-- <i class="el-icon-error" @click="showSwitchPatient = false"></i> -->
					<text class="iconfont icon-guanbi" @click="showSwitchPatient = false"></text>
				</view>
				<view class="border-bottom switch-patient-list" v-for="item in switchPatientList"
					v-bind:key="item.patientId" @click="onSwitchPatientBtn(item)">
					<view class="patient-name">{{ item.patientName|processingName }}</view>
					<view class="visit-number">就诊号：{{ item.cardNumber | processingcardNumber }}</view>
					<!-- <i class="el-icon-check" v-if="currentPatient.patientId === item.patientId"
						style="color: #008cfe"></i> -->
					<text class="iconfont icon-duihao" v-if="currentPatient.patientId === item.patientId"
						style="color: #008cfe"></text>
				</view>
				<view style="display: flex;padding: 10px 0;">
					<view style="width: 50%;" class="switch-patient-btn switch-patient-btn-l"
						@click.native="addPatient">添加就诊人
					</view>
					<view style="width: 50%;" class="switch-patient-btn" @click.native="managePatient">管理就诊人</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import '@/pages/outpatientPayment/outpatientPayment.scss'
	export default {
		name: "registration-record",
		data() {
			return {
				listData: [],
				dfltPatientInfo: [],
				switchPatientList: [],
				currentPatient: {},
				showSwitchPatient: false,
				isSelectAll: false,
				loading: false,
			}
		},
		filters: {
			dateStr(val) {
				if (!val) {
					return '-'
				}
				return val.slice(0, 4) + '-' + val.slice(4, 6) + '-' + val.slice(6, 8)
			},
			processingName(str) {
				if (!str) {
					return '-';
				}
				if (null != str && str != undefined) {
					let star = '' //存放名字中间的*
					//名字是两位的就取姓名首位+*
					if (str.length <= 2) {
						return str.substring(0, 1) + "*";
					} else {
						// 长度减1是因为后面要保留1位
						for (var i = 0; i < str.length - 1; i++) {
							star = star + '*'
						}
						// substring()截取字符串， 第一个参数是开始截取的下标，第二个是结束的下标，第二个参数不填就从下标开始截取到最后一位
						return str.substring(0, 0) + star + str.substring(str.length - 1, str.length);
					}
				}

			},
			processingcardNumber(str) {
				if (!str) {
					return '-';
				}
				let star = '' //存放就诊号中间的*
				// 长度减2是因为后面要保留两位
				for (var i = 0; i < str.length - 2; i++) {
					star = star + '*'
				}
				// substring()截取字符串， 第一个参数是开始截取的下标，第二个是结束的下标，第二个参数不填就从下标开始截取到最后一位
				return str.substring(0, 3) + star + str.substring(str.length - 2, str.length)

			},
		},
		mounted() {
			this.getPatientInfo()
		},
		methods: {
			// 切换就诊人
			onSwitchPatientBtn(item) {
				this.currentPatient = item;
				this.isSelectAll = false;
				this.listData = [];
				this.getList();
			},
			switchPatient() {
				this.showSwitchPatient = true;
			},
			getPatientInfo() {
				this.loading = true
				this.$myRequest({
					url: "/wechat/user/patientcard/info"
				}).then(data => {
					if (data && data.data && data.data && data.data.length > 0) {
						this.switchPatientList = data.data;
						this.currentPatient = data.data[0];
						this.getList();
					}
					this.loading = false;
				}).catch(err => {
					this.loading = false;
				})
			},
			getList() {
				const params = {
					// patientNo: this.currentPatient.cardNumber
					patientNo: '0000000001'
				}
				this.loading = true
				this.$myRequest({
					url: "/hospt/getRegList",
					data: params
				}).then(data => {
					this.listData = data.data
					this.loading = false;
				}).catch(err => {
					this.loading = false;
				})
			},
			goToDetail(item) {
				const params = {
					regNo: item.regNo,
					patientNo: item.outpatientId,
				}

				this.loading = true
				this.$myRequest({
					url: "/hospt/getRegList",
					data: params
				}).then(res => {
					uni.navigateTo({
						url: '/pages/registration-record/registration-details?orderDetail=' + JSON
							.stringify(res.data[0])
					})
					this.loading = false;
				}).catch(err => {
					this.loading = false;
				})
			}
		}
	}
</script>

<style scoped lang="scss">
	.card-row {
		display: flex;
		justify-content: space-between;
		padding: .1rem;
		background-color: #FFFFFF;
	}

	.change-patient-name-btn {
		width: 100%;
		height: 50rpx;
		line-height: 50rpx;
		font-size: 20rpx;
		border-radius: .4rem;
		background: no-repeat;
		border: 1px solid rgb(0, 142, 254);
		padding: 0 .4rem;
		color: rgb(0, 142, 254);
		outline: none;
		margin: 0;
	}

	.record-container {
		.record-item {
			background: #FFFFFF;
			border-bottom: 1px solid #E2E3E4;
			padding: .3rem .5rem;
			display: flex;
			align-items: center;

			.icon-success {
				width: 1rem;
				height: 1rem;
				margin-right: .5rem;
			}

			.record-right {
				flex: 1;
				display: flex;
				align-items: center;
				justify-content: space-between;
				font-size: .25rem;

				.item-bot {
					color: #DFE0E1;
					margin-top: .2rem;
				}
			}
		}
	}
</style>
