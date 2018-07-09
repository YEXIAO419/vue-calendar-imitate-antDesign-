<template>
	<section class="orderDownload">
		<div class="header">
			<h2 class="title">仿ant-design日历</h2>
		</div>
		<div class="main">
			<div class="carlendar-header">
				<el-radio-group v-model="calendar_type" size="mini">
					<el-radio-button label="日账单"></el-radio-button>
					<el-radio-button label="月账单"></el-radio-button>
				</el-radio-group>
				<el-date-picker v-model="selected_date" size="mini" type="month" placeholder="选择日期" v-if="calendar_type == '日账单'" />
				<el-select v-model="selected_year" placeholder="请选择" size="mini" v-if="calendar_type == '月账单'" @change="getYearCalendar">
					<el-option v-for="item in year_list" :key="item" :label="item + '年'" :value="item" />
				</el-select>
			</div>
			<full-calendar v-on:changeMonth="changeMonth" :choseDate="selected_date" :markDate="arr" v-if="calendar_type == '日账单'"></full-calendar>
			<div class="year" v-if="calendar_type == '月账单'">
				<el-row class="year-title">
					<el-col :span="6">一季度</el-col>
					<el-col :span="6">二季度</el-col>
					<el-col :span="6">三季度</el-col>
					<el-col :span="6">四季度</el-col>
				</el-row>
				<div class="year-con">
					<el-row>
						<el-col @click.native="setY(selected_year + '-' + item)" :span="6" :class="{wh_isMark: monthObj[selected_year + '-' + item]}" v-for="item in months">
							<div>
								{{item}}月
								<span class="wh_date-mark" v-if="!monthObj[selected_year + '-' + item]">账单未生成</span>
								<span class="wh_download" v-if="monthObj[selected_year + '-' + item] && monthObj[selected_year + '-' + item] == 'Y'"><i class="el-icon-download" /></span>
								<div class="wh_sum" :class="{wh_sum_postion: item%4 == 0}" v-if="monthObj[selected_year + '-' + item] && monthObj[selected_year + '-' + item] == 'Y'">
									<el-row class="wh_sum-header">
										<el-col :span="8">余额账户</el-col>
										<el-col :span="8">收入余额账户</el-col>
										<el-col :span="8">余额变化</el-col>
									</el-row>
									<el-row class="wh_sum-item">
										<el-col :span="8" class="wh_sum-green">0.00 元</el-col>
										<el-col :span="8" class="wh_sum-red">0.00元</el-col>
										<el-col :span="8">0 笔</el-col>
									</el-row>
									<el-row class="wh_sum-item">
										<el-col :span="8">0 笔</el-col>
										<el-col :span="8">0 笔</el-col>
										<el-col :span="8">期末  000.</el-col>
									</el-row>
								</div>
							</div>
						</el-col>
					</el-row>
				</div>
			</div>
		</div>
	</section>
</template>

<script type="text/javascript">
  import FullCalendar from './components/Calendar/Calendar.vue'

  export default {
	data() {
		return {
			calendar_type: '日账单',
			selected_date: new Date(),
			selected_year: (new Date()).getFullYear(),
			year_list: [],
			months: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
			arr: ['2018-8-1', '2018-8-3', '2018-8-4'], //日账单-已生成账单
			monthObj: {'2018-1': 'N', '2018-2': 'N', '2018-3': 'N', '2018-4': 'N', '2018-5': 'N', '2018-6': 'N', '2018-7': 'N'} //月账单-已生成账单 N:没有点击过
		}
	},
	components: { FullCalendar },
	mounted() {
		this.getYearList();
	},
	methods: {
		//月账单获取近10年年份
		getYearList(){
			let curYear = (new Date()).getFullYear();
			for(var i = 0; i < 11; i++){
				this.year_list.push(curYear - i);
			}
		},

		//改变前后月份——触发编辑框改变
		changeMonth(v){
			if(this.calendar_type == '日账单') {
				this.selected_date = new Date(v);
				return;
			}else {
				let selYear = (v.split('-'))[0];
				if(this.selected_year != selYear) {
					this.selected_year = selYear;
				}
			}
		},

		getYearCalendar(){
			let option = {
					url: this.selected_year, //获取选中年份账单情况
					type: 'GET',
					success: (result, status, xhr) => {
						//this.monthArr = result； 
					}
				}		
		},

		setY(k){
			this.monthObj[k] = 'Y';
		}
	}
 }
</script>

<style lang="less">
	.orderDownload {
		min-height: 745px;

		.header {
			padding: 8px 0 0 24px;
			height: 51px;
			position: relative;
			border-bottom: 1px solid #e9e9e9;

			.title {
				display: inline-block;
				font-size: 18px;
				color: #1d1d1d;
				line-height: 1;
				padding-top: 10px;
				margin: 0;
			}
		}

		.main {
			position: relative;
			padding: 10px 20px 40px 40px;
			.el-date-editor.el-input, .el-date-editor.el-input__inner {
				width: 174px;
			}
			.year {
				margin-top: 15px;
				.year-title {
					height: 40px;
					line-height: 40px;		
					text-align: center;
					background-color: #f6f6f6;
  					border-bottom: 1px solid #e6e6e6;
				}
				.year-con {
  					border-right: 1px solid #e6e6e6;
					.el-col-6 {	
						background-color: #f6f6f6;
						height: 118px;
						text-align: center;
  						border-left: 1px solid #e6e6e6;
						border-bottom: 1px solid #e6e6e6;
						position: relative;
						cursor: pointer;
    					padding: 50px;
						&.wh_isMark {
							background-color: #fff;
						}
						.wh_download {
							position: absolute;
							bottom: 0;
							left: 8px;
							font-size: 20px;
							color: #000;
							z-index: 1;
						}

						&.wh_isMark:hover {					
  							background: #50E3C2;
							color: #fff;
						}

						&.wh_isMark:hover .wh_download {
							color: #fff;
						}

						.wh_date-mark {
							position: absolute;
							font-size: 12px;
							bottom: 5px;
							color: #999;
							left: 10px;
							display: none;
						}

						&:hover .wh_date-mark {
							display: block;
						}

						.wh_sum {
							position: absolute;
							font-size: 12px;
							color: #666;
							width: 360px;
							z-index: 1;
							background-color: #fff;
							border: solid 1px #ddd;
							box-shadow: 1px 1px 2px #ddd;
							border-radius: 5px;
							padding: 0 20px 5px 20px;
							text-align: left;
							bottom: -74px;
							display: none;				
							.wh_sum-header {
								border-bottom: solid 1px #ddd;
								padding: 8px 0 2px 0;
								margin-bottom: 5px;
							}
							.wh_sum-green {
  								color: #03be67;
							}

							.wh_sum-red {
							color: #e01515;
							}

							&.wh_sum_postion {
								right: 0;

							}
						}

						&:hover .wh_sum {
							display: block;
						}
					}
				}
			}
		}
	}
</style>