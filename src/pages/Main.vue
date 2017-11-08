<template>
	<div id="h-home">
		<!--待办事项h-needDealt-->
		<div class="h-homeBox">
			<div class="h-border">
				<h2 class="h-title">用户看板<span class="refresh" @click="getUserBoard">刷新</span></h2>
				<el-row :gutter="20" v-loading="todoLoading">
					<el-col :span="6">
						<div class="h-iconItem bg-purple1">
							<p class="pName">白名单用户数累计：人</p>
							<div class="name">{{userBoardData.whitePersonNum}}</div>
						</div>
					</el-col>
					<el-col :span="6">
						<div class="h-iconItem bg-purple2">
							<p class="pName">黑名单用户数累计：人</p>
							<div class="name">{{userBoardData.blackPersonNum}}</div>
						</div>
					</el-col>
					<el-col :span="6">
						<div class="h-iconItem bg-purple3">
							<p class="pName">授信普通用户累计：人</p>
							<div class="name">{{userBoardData.normalPersonNum}}</div>
						</div>
					</el-col>
					<el-col :span="6">
						<div class="h-iconItem bg-purple4">
							<p class="pName">总授信额度：万</p>
							<div class="name">{{userBoardData.creditAmount}}</div>
						</div>
					</el-col>
				</el-row>
			</div>
		</div>



		<!--报表-->
		<div class="h-homeBox" style="position: relative;">
			<div class="h-border clearfix" style="min-width: 1000px;">
				<div style="float: left; width: 65%; position: relative;">
					<div style="margin-right: 150px;">
						<h2 class="h-title">用户信用评分分布</h2>
						<!--查询条件-->
						<div id="mainChartFilters">
							<div  id="mainChartTime">
								<el-radio-group v-model="filters.countWay" size="mini" @change="getCountWay">
									<el-radio-button label="1">近7天</el-radio-button>
									<el-radio-button label="2">近30天</el-radio-button>
									<el-radio-button label="3">近一年</el-radio-button>
									<el-radio-button label="4">全部</el-radio-button>
								</el-radio-group>
							</div>
							<div class="data">
								<el-date-picker
										v-model="filters.dataRange"
										type="daterange"
										:picker-options="pickerOptions"
										placeholder="选择日期范围"
										:editable = false
										@change="getData"
										size="mini"
										align="right">
								</el-date-picker>
							</div>
						</div>
						<!--图表-->
						<div id="mainChart" style="width:100%; height:450px; margin-top: 5px; z-index: 0;"></div>
					</div>
					<div id="h-mainChart-bar">
						<ul>
							<!--<li class="h-home-list"><span class="h-bg bg0" data-color="red"></span>员工贷</li>
							<li class="h-home-list"><span class="h-bg bg1"></span>消费贷</li>
							<li class="h-home-list"><span class="h-bg bg2"></span>商圈贷</li>-->
							<li class="h-home-list" v-for="(item,index) in chartProductData" v-bind:title="item.name" @click="clickProduct(item,index)"><span class="h-bg" v-bind:class="{ active: item.isActive}"></span>{{item.name}}</li>
						</ul>
					</div>
				</div>

				<!--右侧 start-->
				<div style="float: left; width: 35%; position: relative;">
					<h2 class="h-title">风险规则命中排行</h2>
					<!--查询条件-->
					<div id="mainChartFilters2" style=" position: absolute; top: 0; right: 10px;  z-index: 999;">
						<div class="data">
							<el-date-picker
									v-model="filters2.dataRange"
									type="daterange"
									:picker-options="pickerOptions2"
									placeholder="选择日期范围"
									:editable = false
									@change="getData2"
									size="mini"
									align="right">
							</el-date-picker>
						</div>
					</div>
					<!--图表-->
					<div id="mainChart2" style="width:100%; height:450px; margin-top: 5px; z-index: 0;"></div>
					<div id="rightTips" style="color: #999; display: none; position: absolute; top: 120px; left: 40%;">
						<img src="../../static/images/noData.png" alt="">
						<div style="padding-left: 32px;">暂无数据</div>
					</div>
				</div>
				<!--右侧 end-->
			</div>
		</div>
		<!--产品风向标-->
		<div class="h-homeBox">
			<div class="h-border">
				<h2 class="h-title">产品风向标 <span class="refresh" @click="getVane">刷新</span></h2>
				<el-table
						:data="tableData"
						border
						style="width: 100%" v-loading="tableLoading">
					<el-table-column prop="name" label="产品名称" min-width="100" show-overflow-tooltip></el-table-column>
					<el-table-column prop="createTime" label="产品上架时间" width="180"></el-table-column>
					<el-table-column prop="creditPersonNum" label="总授信人数" width="120"></el-table-column>
					<el-table-column prop="refuseNum" label="拒绝总人数" width="120"></el-table-column>
					<el-table-column prop="creditAmount" label="授信总额度（元）" width="150"></el-table-column>
					<el-table-column prop="minCreditAmount" label="已授信最小额度（元）" width="150"></el-table-column>
					<el-table-column prop="maxCreditAmount" label="已授信最大额度（元）" width="150"></el-table-column>
					<el-table-column prop="passingRate" label="通过率" width="100"></el-table-column>

				</el-table>
			</div>
		</div>

	</div>
</template>

<script>
	import homeApi from '../api/home'
	import echartsFn from '../api/echartsFn'
	import echarts from 'echarts'

	export default {
		data() {
			return {
				pickerOptions: {
					shortcuts: [{
						text: '最近一周',
						onClick(picker) {
							const end = new Date();
							const start = new Date();
							start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
							picker.$emit('pick', [start, end]);
						}
					}, {
						text: '最近一个月',
						onClick(picker) {
							const end = new Date();
							const start = new Date();
							start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
							picker.$emit('pick', [start, end]);
						}
					}, {
						text: '最近三个月',
						onClick(picker) {
							const end = new Date();
							const start = new Date();
							start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
							picker.$emit('pick', [start, end]);
						}
					}]
				},
				pickerOptions2: {
					shortcuts: [{
						text: '最近一周',
						onClick(picker) {
							const end = new Date();
							const start = new Date();
							start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
							picker.$emit('pick', [start, end]);
						}
					}, {
						text: '最近一个月',
						onClick(picker) {
							const end = new Date();
							const start = new Date();
							start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
							picker.$emit('pick', [start, end]);
						}
					}, {
						text: '最近三个月',
						onClick(picker) {
							const end = new Date();
							const start = new Date();
							start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
							picker.$emit('pick', [start, end]);
						}
					}]
				},
				//看板数据
				userBoardData:{
					"whitePersonNum": 0,
					"blackPersonNum": 0,
					"normalPersonNum": 0,
					"creditAmount": 0
				},
				//产品
				tableData: [],
				//过滤条件
				filters:{
					productCode:'', //产品编号
					countWay:'4',       //统计方式(1:近7天,2:近30天,3:近1年,4:全部,5:范围统计)
					dataRange:''
				},
				formatterData:'', //返回格式日期
				todoLoading:false,
				tableLoading:false,
				objLoading:false,

				//产品背景颜色集合
				bgColor:["#199ed8","#65cea7","#fc8675","#ebc85e","#ba3b28","#2e4652","#619faa","#bea296","#707173","#98c3b0"],
				//当前背景颜色
				currentColor:'#199ed8',
				currentName:'',//当前产品名称
				//当前产品编号
				currentProductCode:'',
				//定义产品数据
				chartProductData:[],

				//右侧图表
				formatterData2:'', //返回格式日期
				//过滤条件
				filters2:{
					dataRange:''
				}
			};
		},
		watch: {
			/*'filters.countWay': function (val, oldVal) {
			 setTimeout(() => {
			 this.getChart();
			 },500);
			 }*/
		},
		methods: {
			//单击选择产品
			clickProduct(row,index){
				this.currentColor = row.bgColor;
				this.filters.productCode = row.code;
				for(var i= 0,len= this.chartProductData.length; i<len; i++){
					this.chartProductData[i].isActive = false;
				}
				this.chartProductData[index].isActive = true;
				this.getChart();
			},
			//用户看板数据
			getUserBoard(){
				var _this = this;
				_this.todoLoading = true;
				homeApi.getUserBoard({}).then(res =>{
					_this.userBoardData = res;
					if(_this.userBoardData.creditAmount){
						//_this.userBoardData.creditAmount =  _this.userBoardData.creditAmount / 10000; //转为万单位
						_this.userBoardData.creditAmount = Math.round((_this.userBoardData.creditAmount /10000) * 100) / 100 ;  //接近约数
					}
				setTimeout(()=>{
					_this.todoLoading = false;
			},500)
			}).catch(error=>{
					_this.$Error(_this,error);
				_this.todoLoading = false;
			});
			},
			//获取产品风向标数据
			getVane(){
				var _this = this;
				_this.tableLoading = true;
				homeApi.getVane({}).then(res =>{
					_this.tableData = res;
					var _len = _this.tableData.length;
				    if(_len > 0){
						for(var i=0; i<_len; i++){
							_this.tableData[i].creditAmount = _this.$Utils.addThousandSign(_this.tableData[i].creditAmount,2);
							_this.tableData[i].minCreditAmount = _this.$Utils.addThousandSign(_this.tableData[i].minCreditAmount,2);
							_this.tableData[i].maxCreditAmount = _this.$Utils.addThousandSign(_this.tableData[i].maxCreditAmount,2);
						}
					}
				setTimeout(()=>{
					_this.tableLoading = false;
			},500)
			}).catch(error=>{
					_this.$Error(_this,error);
				_this.tableLoading = false;
			});
			},

			//获取产品
			getDicProduct(){
				var  _this = this;
				homeApi.getDicProduct().then(res =>{
					//console.log(res);
					var content = res.content;
				    if(content.length){
						for(var i=0; i<content.length; i++){
							this.chartProductData.push({
								name:content[i].descp, //描述值为产品名称
								code:content[i].resName, //编号为产品code
								//bgColor:_this.bgColor[i]
								isActive:false
							})
						}
						_this.filters.productCode = this.chartProductData[0].code; //设置默认为第一个
						this.chartProductData[0].isActive = true;
						//调用图表
						_this.getChart();
					}

				}).catch(error =>{
					_this.$Error(_this,error);
				})
			},
			//获取用户信用评分分布接口
			getChart(){
				var _this = this;
				if(_this.formatterData){
					var params = {
						productCode:_this.filters.productCode,
						countWay:_this.filters.countWay,
						beginDate:_this.formatterData.slice(0,10),
						endDate:_this.formatterData.substr(_this.formatterData.length-10),
					};
				}else{
					var params = {
						productCode:_this.filters.productCode,
						countWay:_this.filters.countWay
					};
				}
				//console.log(JSON.stringify(params));
				_this.chartColumn.showLoading();
				homeApi.getCreditScore(params).then(res =>{
					if(res.xAxisData && res.xAxisTitle){
					    echartsFn.barOnlyX('mainChart',res.xAxisTitle,res.xAxisData,{name:'用户数量'});
					}

				_this.chartColumn.hideLoading();
			}).catch(error =>{
					_this.$Error(_this,error);
				_this.chartColumn.hideLoading();
			});
			},

			getCountWay(){
				//alert(this.filters.countWay)
				if(this.filters.countWay !='5'){
					this.formatterData = '';
					this.filters.dataRange = '';
				}
				this.getChart();
			},

			//异常规则命中接口
			abnormalRuleHit(){
				var _this = this;
				if(_this.formatterData2){
					var params = {
						beginDate:_this.formatterData2.slice(0,10),
						endDate:_this.formatterData2.substr(_this.formatterData2.length-10),
					};
				}else{
					var params = {
						beginDate:'',
						endDate:''
					};
				}
				//console.log(JSON.stringify(params));
				_this.chartColumn2.showLoading();
				homeApi.abnormalRuleHit(params).then(res =>{
					if(res.yAxisData && res.yAxisTitle){
						echartsFn.barOnlyY('mainChart2',res.yAxisTitle,res.yAxisData,{name:'命中数量'});
						if(res.yAxisData.length == 0){
							document.querySelector('#mainChart2 canvas').style.display = 'none';
							document.querySelector('#rightTips').style.display = 'block';
						}else{
							document.querySelector('#mainChart2 canvas').style.display = 'block';
							document.querySelector('#rightTips').style.display = 'none';
						}
					}
				_this.chartColumn2.hideLoading();
			}).catch(error =>{
					_this.$Error(_this,error);
				_this.chartColumn2.hideLoading();
			});
			},

			//返回格式化后的值
			getData(val){
				this.formatterData = val;
				if(val){
					this.filters.countWay = '5';
				}else{
					this.filters.countWay = '4'; //设为全部
				}
				this.getChart();
			},
			//返回格式化后的值
			getData2(val){
				this.formatterData2 = val;
				this.abnormalRuleHit();
			}

		},
		mounted() {
			this.chartColumn = echarts.init(document.getElementById('mainChart'));
			this.chartColumn2 = echarts.init(document.getElementById('mainChart2'));
			this.getUserBoard(); //获取用户看板
			this.getVane(); //获取产品风向标
			this.getDicProduct(); //获取数字字典产品列表

			this.abnormalRuleHit(); //获取异常规则命中接口
		}
	}
</script>
<style scoped>
	#mainChartTime .el-radio-group .el-radio-button__inner{padding: 5px 6px;}
	#mainChart,#mainChart2{margin-top: -50px!important; z-index: 0; position: relative;}
</style>