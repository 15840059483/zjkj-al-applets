<template>
	<view>
		<!-- loading加载动画，type默认值是原子，love爱心，mask属性是遮罩 -->
		<zero-loading v-if="loading" type="pulse" mask></zero-loading>
		<view class="zhuti">
			<div class="bg-white outpatient-department-container" v-if="keshiname!=''">
				<uni-card>
					<div class="outpatient-department" style="font-size: 16px;margin-bottom: .2rem;">
						<img src="https://s1.ax1x.com/2022/09/20/xCDS5F.png" />
						{{ keshiname }}
					</div>
					<p class="introduce" style="
			          overflow: hidden;
			          text-overflow: ellipsis;
			          display: -webkit-box;
			          -webkit-line-clamp: 6;
			          -webkit-box-orient: vertical;
			        ">
						{{ deptInfo.introduce }}
					</p>
				</uni-card>
			</div>
			<div class="bg-white outpatient-department-container" v-else>
				<uni-card>
					<div class="outpatient-department" style="font-size: 16px;margin-bottom: .2rem;">
						<img src="https://s1.ax1x.com/2022/09/20/xCDS5F.png" />
						{{ deptInfo.deptName }}
					</div>
					<p class="introduce" style="
			          overflow: hidden;
			          text-overflow: ellipsis;
			          display: -webkit-box;
			          -webkit-line-clamp: 6;
			          -webkit-box-orient: vertical;
			        ">
						{{ deptInfo.introduce }}
					</p>
				</uni-card>
			</div>

			<!-- 选择预约的日期 -->
			<view class="riqi"
				style="border-top: 1px solid rgb(212, 212, 212);border-bottom: 1px solid rgb(212, 212, 212);">
				<view class="data-container">
					<!-- 展示当前时间的年月日 -->
					<uni-row style="padding: 0.3rem 0.5rem;" class="bg-white">
						<view :span="10" class="data-change-btn"
							style="margin:5px 0 5px 0;padding:0 .2rem 0 .2rem;display: flex;"
							@click.native="showMore()">
							<view>
								{{
				              today.getFullYear() +
				              "-" +
				              (today.getMonth() + 1) +
				              "-" +
				              today.getDate()
				            }}
							</view>
							<div class="icon-caret-bor">
								<text v-if="!isShowDate" class="iconfont icon-caret-top"
									style="font-size: 12px;"></text>
								<text v-if="isShowDate" class="iconfont icon-caret-bottom"
									style="font-size: 12px;"></text>
							</div>
						</view>
					</uni-row>
					<!-- 日历内部框 -->
					<!-- 此处使用了过渡动画   <view class="show-more" :class="[isShowDate ? 'show-more-click' : '']">-->
					<view class="show-more" :class="[isShowDate ? 'show-more-click' : '']">

						<view v-show="isShowText" class="bg-white">
							<!-- 星期，展示当前日期是星期几以及今天后的6天 -->
							<uni-row type="flex" justify="space-around">
								<view v-for="week in weeks" v-bind:key="week" class="text-center date-week">
									{{ week }}
								</view>
							</uni-row>
							<uni-row type="flex" justify="space-around">
								<view v-for="(day, index) in days" v-bind:key="day.day" class="text-center"
									style=" width: 14.2%;height: 100%;">
									<view :class="selectDay == day.day ? 'select-date' : ''" class="date-day"
										@click="onSelectDayBtn(day, index)">
										<view style="line-height: 70rpx;">{{ day.day }}</view>
									</view>
								</view>
							</uni-row>
						</view>

					</view>
				</view>
			</view>

			<div class="bg-white">
				<view v-for="(item,index) in doctors" v-bind:key="index" class="doc-row" @click="clickDoctor(item)"
					style="display: flex;">
					<view style="width: 25%;">
						<div class="doc-img">
							<img v-if="item.docID" :src="baseUrl + '/hospt/doctorsImg/' + item.docID" />
							<img v-if="!item.docID" :src="baseUrl + '/hospt/doctorsImg/' + '0'" />
						</div>
					</view>
					<view style="width: 60%;padding: .1rem .2rem .1rem .2rem;">
						<view>
							<view class="doc-name">{{
			              item.docName ? item.docName : item.deptName
			            }}</view>
							<view class="doc-position" style="margin-top: 40px;">{{ item.regLevelName }}</view>
						</view>
					</view>
					<!--          <view :span="7" class="text-right doc-money">￥{{item.totalFee}}</view>-->
					<view style="width: 15%;" class="text-right doc-money">{{
			          item.noonID == "1" ? "上午" : "下午"
			        }}</view>
					<view :span="5" class="text-center">
						<!--            <span class="doc-remainder">余号：{{item.remainingNum}}</span>-->
					</view>
				</view>
			</div>


			<div class="confirm-div" v-if="isShowConfirm"></div>


			<div class="confirm-container bg-white" v-if="isShowConfirm">
				<div class="text-center confirm-doc">
					<div style="margin-bottom: 0.2rem">
						{{ doctor.deptName }}&nbsp;&nbsp;&nbsp;&nbsp;
						<span style="font-size: 0.35rem">{{ doctor.regLevelName }}</span>
					</div>
					<div>
						{{ "￥" + feeBill.DigFee }}&nbsp;&nbsp;&nbsp;&nbsp;
						{{
			              today.getMonth() + 1 + " - " + selectDay
			            }}&nbsp;&nbsp;&nbsp;&nbsp; 星期{{ weeks[selectDayIndex] }}
					</div>
				</div>
				<div class="confirm-time">
					<div>{{ doctor.noonID == "1" ? "上午" : "下午" }}</div>
				</div>
				<view class="text-center confirm-btn-row">
					<view :span="12" class="confirm-btn" @click.native="closeConfirm">取消</view>
					<view :span="12" class="confirm-btn confirm-btn-save" @click.native="saveConfirm">确认</view>
				</view>
			</div>

		</view>
	</view>
</template>

<script>
	// 引入scss文件
	import './makeAppointmentRegister.scss'
	import base from "../../request/base";
	import store from "../../store";
	export default {
		// 调用头部组件
		components: {
			// header
		},
		data() {
			return {
				title: "眼门", // 页面标题
				shouye: "no", // 是否是首页，不是首页显示返回上一层箭头  
				loading: true,
				// 这里是控制点击速度的变量
				lastTime: 0, //默认上一次点击时间为0
				checkday: "",
				baseUrl: base.base.sq,
				deptInfo: {},
				isShowDate: true,
				today: new Date(),
				weeks: [],
				days: [],
				selectDay: 0,
				selectDayInfo: {},
				selectDayIndex: 0,
				doctors: [],
				doctor: {},
				isShowConfirm: false,
				feeBill: {},
				deptbill: {
					begin: "2021/11/25 8:00:00",
					deptID: "0008",
					deptName: "眼门",
					docID: "200328",
					docName: "曲建军",
					end: "2021/11/25 10:50:00",
					iD: "216608",
					noonID: new Date().getHours() < 12 ? "1" : "2",
					regLevelID: "1",
					regLevelName: "普诊",
					seeNO: "0",
					totalFee: "8",
					testDay: "",
				},
				showSwitchPatient: true,
				deptId: "",
				isShowText: true, // 控制内容显示
				keshiname: '', //科室名称
			}
		},
		// 在uniapp中如果要使用路由传参必须使用onload(路由传参中的参数值)
		onLoad(e) {
			//console.log(e);
			this.deptId = e.id;
			//console.log(this.deptId)
			this.jiazai()
		},
		methods: {
			// 加载框
			jiazai() {
				this.loading = true;
				// 定时器，setTimeout只执行一次，setInterval执行多次
				setTimeout(() => {
					this.loading = false;
					//console.log(this.loading);
				}, 500)
			},

			// 控制动画的方法
			showMore() {
				//获取当前时间的时间戳

				let now = new Date().valueOf();

				//第一次点击
				// 控制两秒内点击多次只触发一次方法
				if (this.lastTime == 0) {

					//console.log('触发事件');
					// 控制显示
					this.isShowDate = !this.isShowDate

					// 先展开500毫秒后再显示文字,收缩200毫秒后再隐藏文字
					if (this.isShowText) {
						setTimeout(() => {
							this.isShowText = !this.isShowText
						}, 100)
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
						this.isShowDate = !this.isShowDate

						// 先展开900毫秒后再显示文字,收缩100毫秒后再隐藏文字
						if (this.isShowText) {
							setTimeout(() => {
								this.isShowText = !this.isShowText
							}, 100)
						} else {
							setTimeout(() => {
								this.isShowText = !this.isShowText
							}, 900)
						}
						this.lastTime = now;

						//console.log('间隔大于2秒，触发方法');

						//添加自己要调用的方法

					} else {
						uni.showToast({
							title: '点的太快啦！QAQ',
							icon: 'none',
							duration: 2000
						});
						//console.log('不触发');

					}

				}
			},

			// // 控制动画的方法
			// showMore() {
			// 	// 控制显示
			// 	this.isShowDate = !this.isShowDate

			// 	// 先展开500毫秒后再显示文字,收缩200毫秒后再隐藏文字
			// 	if (this.isShowText) {
			// 		setTimeout(() => {
			// 			this.isShowText = !this.isShowText
			// 		}, 100)
			// 	} else {
			// 		setTimeout(() => {
			// 			this.isShowText = !this.isShowText
			// 		}, 900)
			// 	}
			// },

			getDetDoctorInfo() {
				this.doctors = [];
				this.testDay =
					this.today.getFullYear() +
					"-" +
					(this.today.getMonth() < 10 ?
						"0" + (this.today.getMonth() + 1) :
						this.today.getMonth() + 1) +
					"-" +
					this.selectDay;
				const params = {
					beginDay: this.selectDayInfo.year +
						"-" +
						(this.selectDayInfo.month < 10 ?
							"0" + this.selectDayInfo.month :
							this.selectDayInfo.month) +
						"-" +
						this.selectDayInfo.day,
					deptId: this.deptId,
					//endDay: this.today.getFullYear() + (this.today.getMonth() < 10 ? '0'+(this.today.getMonth() + 1) : (this.today.getMonth() + 1)) + this.selectDay
				};
				this.$myRequest({
					url: "/hospt/getDetDoctorInfo",
					data: params,
				}).then(data => {
					this.deptInfo = data.data.deptInfo;
					this.doctors = data.data.doctorInfo;
					this.loading = false;
				}).catch(err => {
					this.loading = false;
				})
			},
			onSelectDayBtn(day, index) {
				this.jiazai();
				//this.checkday = day
				//console.log(day);
				//if(index>0){
				//this.$message.warning('暂不支持预约挂号');
				//return false;
				//}
				if ((this.selectDay = this.today.getDay())) {
					this.showSwitchPatient = true;
				} else {
					false;
				}
				this.selectDay = day.day;
				this.selectDayInfo = day;
				//console.log(this.selectDayInfo);
				this.selectDayIndex = index;
				this.getDetDoctorInfo();
			},
			clickDoctor(doctor) {
				//console.log(doctor)
				this.doctor = doctor;
				if (doctor.docName != null) {
					this.goToPage(doctor);
				} else {
					this.openConfirm(doctor);
				}
			},
			openConfirm(doctor) {
				this.isShowConfirm = true;
				//console.log(this.doctor);

				const params = {
					RegLevelID: doctor.regLevelID,
				};
				if (doctor.regLevelID != "1") {
					this.$myRequest({
						url: "/hospt/getDoctorFeeBill",
						data: params
					}).then(data => {
						const feeBill = data.data;
						this.feeBill = feeBill;
						this.doctor.totalFee = this.feeBill.DigFee;
						this.loading = false;
					}).catch(err => {
						this.loading = false;
					})
				}
				this.feeBill.DigFee = this.deptbill.totalFee;
			},
			closeConfirm() {
				this.isShowConfirm = false;
			},
			saveConfirm() {
				this.$router.push({
					path: "/registrationConfirmation",
					query: {
						doctorInfo: encodeURIComponent(JSON.stringify(this.doctor)),
						date: this.today.getFullYear() +
							"/" +
							(this.today.getMonth() < 10 ?
								"0" + (this.today.getMonth() + 1) :
								this.today.getMonth() + 1) +
							"/" +
							(this.selectDay < 10 ? "0" + this.selectDay : this.selectDay),
						noonName: "[" + (this.doctor.noonID == "1" ? "上午" : "下午") + "]",
						noonID: this.doctor.noonID,
					},
				});
				this.closeConfirm();
			},
			getDays(day, month) {
				let currentDay = day;

				for (let i = 0; i < 7; i++) {
					let date = new Date();
					date.setDate(date.getDate() + i);
					let info = {
						day: date.getDate(),
						month: date.getMonth() + 1,
						year: date.getFullYear(),
					};
					this.days.push(info);
					this.selectDay = this.days[0].day;
					this.selectDayInfo = this.days[0];
					//console.log(date.getDate());
				}
			},
			getWeeks(week) {
				let currentWeek = week;
				for (let i = 0; i < 7; i++) {
					this.weeks.push(this.processingWeekDisplay(currentWeek));
					currentWeek++;
					if (currentWeek === 8) {
						currentWeek = 1;
					}
				}
			},
			processingWeekDisplay(num) {
				switch (num) {
					case 1:
						return "一";
					case 2:
						return "二";
					case 3:
						return "三";
					case 4:
						return "四";
					case 5:
						return "五";
					case 6:
						return "六";
					case 7:
						return "日";
				}
			},
			goToPage(item) {
				//console.log(this.selectDayInfo);

				uni.navigateTo({
					url: '/pages/doctorScheduling/doctorScheduling?regLevelID=' +
						item.regLevelID +
						"&doctorInfo=" +
						JSON.stringify(item) +
						"&testDay=" +
						this.testDay +
						"&selectDayIndex=" +
						this.weeks[this.selectDayIndex] +
						"&selectDay=" +
						this.selectDay +
						"&selectDayInfo=" +
						JSON.stringify(this.selectDayInfo) +
						"&date=" +
						this.selectDayInfo.year +
						"/" +
						(this.selectDayInfo.month < 10 ?
							"0" + this.selectDayInfo.month :
							this.selectDayInfo.month) +
						"/" +
						(this.selectDay < 10 ? "0" + this.selectDay : this.selectDay)
				});


				// this.$router.push(
				// 	"/doctorScheduling?regLevelID=" +
				// 	item.regLevelID +
				// 	"&doctorInfo=" +
				// 	JSON.stringify(item) +
				// 	"&testDay=" +
				// 	this.testDay +
				// 	"&selectDayIndex=" +
				// 	this.weeks[this.selectDayIndex] +
				// 	"&selectDay=" +
				// 	this.selectDay +
				// 	"&selectDayInfo=" +
				// 	JSON.stringify(this.selectDayInfo) +
				// 	"&date=" +
				// 	this.selectDayInfo.year +
				// 	"/" +
				// 	(this.selectDayInfo.month < 10 ?
				// 		"0" + this.selectDayInfo.month :
				// 		this.selectDayInfo.month) +
				// 	"/" +
				// 	(this.selectDay < 10 ? "0" + this.selectDay : this.selectDay)
				// );
			},
		},
		// 这是uni的生命周期
		onShow() {
			this.jiazai()
		},
		mounted() {
			console.log(this.baseUrl)
			console.log(base)
			this.getDays(this.today.getDate(), this.today.getMonth());
			this.getWeeks(this.today.getDay());
			this.getDetDoctorInfo();
			let a = "12:00:00".replace(/-/g, "/");
			let b = new Date(a);
			//console.log(new Date().getHours());
			this.jiazai();
			// 这里让keshiname等于vuex中的keshiname
			this.keshiname = store.state.keshiname;
			//console.log(this.keshiname);
		},
	};
</script>

<style scoped>
	/* .header {
		position: fixed;
		top: 0;
		z-index: 999;
	} */

	/* .zhuti {
		margin-top: 170rpx;
	} */

	.outpatient-department-container>>>.uni-card {
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
		height: 2rem;
	}
</style>
