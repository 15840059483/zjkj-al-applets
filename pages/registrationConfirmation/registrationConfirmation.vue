<template>
	<view>
		<!-- loading加载动画，type默认值是原子，love爱心，mask属性是遮罩 -->
		<zero-loading v-if="loading" type="pulse" mask></zero-loading>
		<!-- 使用组件的时候首字母要大写！！！！ -->
		<!-- <view class="header" style="width: 100%;height: 150rpx;">
			<Header :title="title" :shouye="shouye"></Header>
		</view> -->
		<view class="zhuti">
			<uni-card>
				<div class="bg-white patient-container" style="margin-bottom:0;">
					<view style="display: flex;padding:10px .3rem 10px .3rem;" class="patient-container-row">
						<view style="width: 40%;text-align: left;">挂号类别</view>
						<view style="width: 60%;text-align: right;" class="text-right">{{doctorInfo.regLevelName}}
						</view>
					</view>
					<view style="display: flex;padding:10px .3rem 10px .3rem;" class="patient-container-row">
						<view style="width: 40%;text-align: left;">就诊科室</view>
						<!-- 根据vuex中的科室名称是否为空限制显示 -->
						<view style="width: 60%;text-align: right;" class="text-right" v-if="keshiname != ''">
							{{keshiname}}
						</view>
						<!-- 原有的，如果需要改回原有的将v-eles与上面v-if中所有内容删掉 -->
						<view style="width: 60%;text-align: right;" class="text-right" v-else>{{doctorInfo.deptName}}
						</view>
					</view>
					<view style="display: flex;padding:10px .3rem 10px .3rem;" class="patient-container-row"
						v-if="doctorInfo.docName&&doctorInfo.regLevelID!='1'">
						<view style="width: 40%;text-align: left;">就诊医生</view>
						<view style="width: 60%;text-align: right;" class="text-right">{{doctorInfo.docName}}</view>
					</view>
					<view style="display: flex;padding:10px .3rem 10px .3rem;" class="patient-container-row">
						<view style="width: 40%;text-align: left;">就诊时间</view>
						<view style="width: 60%;text-align: right;" class="text-right red-font">{{seeDateInfo.seeTime}}
						</view>
					</view>
				</div>
			</uni-card>
			<uni-card>
				<div class="bg-white patient-container" style="margin-bottom:0;">
					<view style="display: flex;padding: 0 0.3rem;" class="patient-container-row">
						<view style="width: 40%;text-align: left;">诊查费</view>
						<view style="width: 60%;text-align: right;" class="text-right red-font">￥{{doctorInfo.totalFee}}
						</view>
					</view>
					<!--      <view class="patient-container-row">-->
					<!--        <view :span="10">工本费</view>-->
					<!--        <view :span="14" class="text-right red-font">￥1.00</view>-->
					<!--      </view>-->
					<!--      <view class="patient-container-row">-->
					<!--        <view :span="10">合计金额</view>-->
					<!--        <view :span="14" class="text-right red-font">￥20.00</view>-->
					<!--      </view>-->
				</div>
			</uni-card>
			<uni-card>
				<div style="margin-bottom:0;" class="bg-white patient-container">
					<!--  此处代码作废，仅供参考
					<view style="display: flex;" class="patient-container-row" @click.native="showMore()">
						<view style="width: 40%;text-align: left;">就诊人</view>
						<view class="text-right" style="position: relative;width: 60%;">
							<div class="select-patient-show">
								<text class="red-font">{{selectPatient.patientName}}</text>
								<text v-if="!showOtherPatient" class="iconfont icon-youjiantou"
									style="font-size: 24px;"></text>
								<text v-if="showOtherPatient" class="iconfont icon-xiajiantou"
									style="font-size: 24px;"></text>
							</div>
						</view>
					</view> -->
					<!-- 这里使用了uniapp中的uni-collapse组件替代了动画效果 -->
					<uni-collapse ref="collapse">
						<uni-collapse-item titleBorder="none" :open="true">
							<template style="border-bottom: 1px solid #e1e1e1;" v-slot:title>
								<view style="display: flex;" class="patient-container-row">
									<view style="width: 40%;text-align: left;">就诊人</view>

									<view class="select-patient-show"
										style="position: relative;width: 60%;text-align:right;">
										<text class="red-font"
											style="margin-right: .2rem;">{{processingName(selectPatient.patientName)}}</text>
									</view>

								</view>
							</template>
							<view class="content">
								<view v-show="isShowText">
									<div v-if="showOtherPatient" class="zoom-patient-container">
										<view style="display: flex;padding:.3rem .5rem .3rem .5rem;"
											class="patient-container-row"
											:class="item.patientId === selectPatient.patientId ? 'select-bg' : ''"
											v-for="(item,index) in patients" v-bind:key="index"
											@click.native="selectPatient = item">
											<view style="width: 50%;text-align: left;">
												{{processingName(item.patientName)}}</view>
											<view style="width: 50%;text-align: right;" class="text-right">
												{{processingcardNumber(item.cardNumber)}}
											</view>
										</view>
									</div>
								</view>
							</view>
						</uni-collapse-item>
					</uni-collapse>
					<view style="display: flex;padding:10px .3rem 10px .3rem;" class="patient-container-row">
						<view style="width: 50%;text-align: left;">就诊号</view>
						<view style="width: 50%;text-align: right;" class="text-right red-font">
							{{processingcardNumber(selectPatient.cardNumber)}}
						</view>
					</view>
					<view style="display: flex;" class="patient-container-row">
						<view style="font-size:12px;color:red;width: 10%;">注意</view>
						<view class="text-right red-font" style="font-size:12px;color:red;width: 90%;">
							如需退号请在挂号时间点前半个小时进行退号，过期不退！
						</view>
					</view>
				</div>
			</uni-card>
			<div class="anniu">
				<button type="primary" style="width: 100%;height: 60%;" @click="dialog()">提 交</button>
				<!-- <zg-button type="primary" @click="dialog()" class="tijiaoanniu" block size="base">
					提 交
				</zg-button> -->
				<!-- <el-button class="submit-btn" @click="dialog()">提 交</el-button> -->
			</div>

			<!-- 挂号提示书弹出框，设置点击空白不会退出 is_mask="true" maskFun="false"，无关闭按钮 is_close="false"-->
			<pop ref="popo" direction="center" :width="80" height="fit-content">
				<view class="biaoti">
					提示
				</view>
				<view class="neirong-wai">
					<view class="neirong" style="padding: .5rem;">
						<text>挂号成功后请在{{seeDateInfo.seeTime}}前,前往院内自助机打印凭条方便就诊</text>
					</view>
				</view>
				<view class="zizhuanniu">
					<view @click="payRegister()">确 定</view>
				</view>
			</pop>


			<!-- <el-dialog title="提示" :visible.sync="centerDialogVisible" width="70%" center>
				<span>挂号成功后请在{{seeDateInfo.seeTime}}前,前往院内自助机<br>打印凭条方便就诊</span>
				<span slot="footer" class="dialog-footer">
					<el-button type="primary" @click="payRegister">确 定</el-button>
				</span>
			</el-dialog> -->

		</view>
	</view>
</template>

<script>
	// 引入导航栏组件
	// import header from '@/components/header/header.vue'
	import '../patient-management/patient-management.scss';
	import './registrationConfirmation.scss';
	import store from "../../store";
	// 引入弹窗组件
	import pop from "@/components/ming-pop/ming-pop.vue";
	export default {
		// 调用头部组件
		components: {
			// header,
			pop
		},
		computed: {
			processingName() {
				return function(str) {
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
				}
			},
			processingcardNumber() {
				return function(str) {
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
				}
			}
		},
		data() {
			return {
				title: "挂号确认", // 页面标题
				shouye: "no", // 是否是首页，不是首页显示返回上一层箭头
				loading: true,
				isShowText: true, // 控制内容显示
				// 这里是控制点击速度的变量
				lastTime: 0, //默认上一次点击时间为0

				doctorInfo: {},
				registrationDate: '',
				patients: [],
				selectPatient: {},
				showOtherPatient: true,
				centerDialogVisible: false,
				noonID: '',
				noonName: '',
				seeDateInfo: {},
				schemaId: '',
				keshiname: '', //获取vuex中的科室名称
			}
		},
		methods: {
			// 开启弹窗的方法
			dialog() {
				this.$refs.popo.show();
			},
			/* 此处代码已作废
			// 控制动画的方法
						showMore() {
							//获取当前时间的时间戳

							let now = new Date().valueOf();

							//第一次点击
							// 控制两秒内点击多次只触发一次方法
							if (this.lastTime == 0) {

								console.log('触发事件');
								// 控制显示
								this.showOtherPatient = !this.showOtherPatient

								// 先展开500毫秒后再显示文字,收缩200毫秒后再隐藏文字
								if (this.isShowText) {
									setTimeout(() => {
										this.isShowText = !this.isShowText
									}, 500)
								} else {
									setTimeout(() => {
										this.isShowText = !this.isShowText
									}, 900)
								}
								this.lastTime = now;

							} else {

								if ((now - this.lastTime) > 2000) {

									//重置上一次点击时间，2000是我自己设置的2秒间隔，根据自己的需要更改
									// 控制显示
									this.showOtherPatient = !this.showOtherPatient

									// 先展开500毫秒后再显示文字,收缩200毫秒后再隐藏文字
									if (this.isShowText) {
										setTimeout(() => {
											this.isShowText = !this.isShowText
										}, 500)
									} else {
										setTimeout(() => {
											this.isShowText = !this.isShowText
										}, 900)
									}
									this.lastTime = now;

									console.log('间隔大于2秒，触发方法');

									//添加自己要调用的方法

								} else {
									uni.showToast({
										title: '点的太快啦！QAQ',
										icon: 'none',
										duration: 2000
									});
									console.log('不触发');

								}

							}
						},
			*/


			// 加载框
			jiazai() {
				this.loading = true;
				// 定时器，setTimeout只执行一次，setInterval执行多次
				setTimeout(() => {
					this.loading = false;
					console.log(this.loading);
				}, 500)
			},
			//获取就诊人
			getPatientInfo() {
				this.$myRequest({
					url: "/wechat/user/patientcard/info",
				}).then(data => {
					this.patients = data.data;
					this.selectPatient = data.data[0];
					this.loading = false;
					this.$nextTick(() => {
						this.$refs.collapse.resize()
					})
				}).catch(err => {
					this.loading = false;
				})
			},
			payRegister() {
				const params = {
					schemaId: this.schemaId,
					dateBegin: this.seeDateInfo.seeTime,
					dateEnd: this.seeDateInfo.seeTime,
					deptId: this.doctorInfo.deptID,
					deptName: this.doctorInfo.deptName,
					doctorName: this.doctorInfo.docName,
					doctorId: this.doctorInfo.docID,
					doctorTitle: this.doctorInfo.regLevelName,
					doctorTitleId: this.doctorInfo.regLevelID,
					idEnNo: this.selectPatient.patientCardId,
					iPhone: this.selectPatient.patientPhone,
					patientName: this.selectPatient.patientName,
					patientNo: this.selectPatient.cardNumber,
					payMount: this.doctorInfo.totalFee,
					noonId: this.doctorInfo.noonID,
					regPeriod: this.noonName,
					'regSeeData.ID': this.seeDateInfo.iD,
					'regSeeData.queue': this.seeDateInfo.queue,
					'regSeeData.seeTime': this.seeDateInfo.seeTime,
					'regSeeData.state': this.seeDateInfo.state,
					pay_Type: 'AL'
				}

				this.$myRequest({
					url: "/wechat/pay/reg",
					data: params
				}).then(data => {
					if (data.code == 0) {
						my.tradePay({
							// 调用统一收单交易创建接口（alipay.trade.create），获得返回字段支付宝交易号trade_no
							tradeNO: data.data.tradeNO,
							success: (res) => {
								// 关闭弹窗
								if (!res.resultCode == '9000') {
									this.$refs.popo.close();
									uni.navigateTo({
										url: '/pages/paymentPage/paymentPage?orderNo=' + data
											.data.orderNo
									});
								} else {
									uni.showToast({
										title: '支付失败',
										icon: 'none',
										duration: 2000
									});
								}
							},
							fail: (res) => {
								my.alert({
									content: '已取消支付',
								});
							}
						});
					}
				}).catch(err => {
					this.loading = false;
				})

				// _this.$router.push('/paymentPage?orderNo=' + 1);

			}
		},
		// 这是uni的生命周期
		// 在uniapp中如果要使用路由传参必须使用onload(路由传参中的参数值)
		onLoad(e) {
			console.log(e);
			let doctorInfo = decodeURIComponent(e.doctorInfo);
			this.doctorInfo = JSON.parse(doctorInfo);
			this.registrationDate = e.date;
			this.noonName = e.noonName;
			this.seeDateInfo = JSON.parse(decodeURIComponent(e.seeInfo));
			this.schemaId = e.schemaId;
			this.jiazai()
		},
		onShow() {
			this.jiazai()
		},
		mounted() {
			this.getPatientInfo();
			//获取vuex中的科室名称
			this.keshiname = store.state.keshiname;
			this.jiazai()
			console.log(this.doctorInfo, this.registrationDate, this.noonName, this.seeDateInfo)
		},
	};
</script>

<style scoped>
	/* 	.header {
		position: fixed;
		top: 0;
		z-index: 999;
	} */
	/* 
	.zhuti {
		margin-top: 190rpx;
	} */

	.zhuti>>>.uni-card {
		padding: 0 !important;
	}

	/* 动画样式 */
	/* 点击更多的样式 */
	.show-more {
		width: 100%;
		height: 1px;
		transition: height 1s;
		-moz-transition: height 1s;
		/* Firefox 4 */
		-webkit-transition: height 1s;
		/* Safari and Chrome */
		-o-transition: height 1s;
		/* Opera */
	}

	/* 动画样式 */
	.show-more-click {
		height: 2.5rem;
	}

	.biaoti {
		font-size: 18px;
		font-weight: 800;
		text-align: center;
		margin-bottom: 10px;

	}

	.neirong-wai {
		border-top: 1px solid rgb(212, 212, 212);
		border-bottom: 1px solid rgb(212, 212, 212);
	}

	.zizhuanniu {
		color: rgb(0, 147, 255);
		padding: 0;
		margin-top: 10px;
		text-align: center;
		width: 100%;
		font-size: 16px;
		font-weight: 800;
	}

	.anniu {
		width: 65%;
		height: 80px;
		margin: 0 auto;
		display: flex;
	}
</style>
