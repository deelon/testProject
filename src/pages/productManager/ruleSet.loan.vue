<template>
	<section>
		<el-row class="rule_box">
			<el-col :span="18">
				<strong>借款情况</strong>
			</el-col>
			<el-col :span="6" style="text-align: right;">
				<el-button type="primary" icon="plus" size="small" @click="handleSubAdd()">配置规则</el-button>
			</el-col>
		</el-row>
		<el-table :data="Data" border  style="width: 100%; margin-top: 10px;" v-loading="listLoading">
			<el-table-column prop="ruleNum" label="指标编号"  width="150"></el-table-column>
			<el-table-column prop="ruleName" label="规则名称" min-width="250"  show-overflow-tooltip></el-table-column>
			<el-table-column prop="ruleType" label="规则分类" min-width="200"  show-overflow-tooltip></el-table-column>
			<el-table-column prop="state" label="规则状态" width="100"></el-table-column>
			<el-table-column inline-template :context="_self" label="操作" width="150" >
	             <span>
					 <el-button type="primary" size="small" @click="handleEdit_loan(row)">编辑</el-button>
					 <el-button type="danger" size="small" @click="handleDel(row)">删除</el-button>
				</span>
			</el-table-column>
		</el-table>
	</section>
</template>

<script>

    export default {
        data() {
            return {
                num:'10',
                listLoading:false,
                Data:[
					{
                        ruleNum:'0001',
						ruleName:'是否有审批中的借款',
                        state:"启用",
                        ruleType:"是否有审批中的借款"

					},
                    {
                        ruleNum:'0002',
						ruleName:'贷款逾期金额判断',
                        state:"停用",
                        ruleType:"贷款逾期金额判断"

                    },
					{
						ruleNum:'0003',
						ruleName:'个贷历史逾期严重',
						state:"停用",
						ruleType:"个贷历史逾期严重"

					}
				]
            }
        },
        methods: {
			handleEdit_loan(row){
				this.$bus.$emit('ruleDialogShow','编辑规则',row);
			},
			//明细--路由跳转
			handleView(row){
				this.$router.push({ path: '/productList/scoreSet-listView', query: { name: 'scoreSet-listView',id:row.ruleNum }});
			},
			handleDel(row){
				var _this = this;
				_this.$confirm('您确定删除该项规则吗？', '提示', {}).then(() => {
					_this.$message({
					type: 'success',
					message: '删除成功！'
				});
				});
			},
			handleSubAdd(){
				this.$bus.$emit('ruleSet_settingDialog_open','规则模型配置',{},'借款情况');
			}
        }
    }
</script>

<style scoped>
	.rule_box{ margin-top: 25px;}
	.c_ff0000{ color: #ff0000;}
</style>