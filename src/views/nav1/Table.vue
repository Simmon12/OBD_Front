<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.fenceId" placeholder="围栏ID"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getFences">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="fences" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="fenceId" label="围栏ID" width="120" >
			</el-table-column>
			<el-table-column prop="fenceName" label="围栏名字" width="100"  >
			</el-table-column>
			<el-table-column prop="fenceMonitoredPerson" label="监控对象" width="100" >
			</el-table-column>
			<el-table-column prop="fenceKeyword" label="行政区关键字" width="120" >
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="围栏ID" prop="fenceId">
					<el-input v-model="editForm.fenceId" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="围栏名字">
					<el-input v-model="editForm.fenceName" ></el-input>
				</el-form-item>
				<el-form-item label="监控对象">
					<el-input  v-model="editForm.fenceMonitoredPerson"></el-input>
				</el-form-item>
				<el-form-item label="行政区关键字">
					<el-input  v-model="editForm.fenceKeyword"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="editFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--新增界面-->
		<el-dialog title="新增" v-model="addFormVisible" :close-on-click-modal="false">
			<el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
				<el-form-item label="围栏ID" prop="fenceId">
					<el-input v-model="addForm.fenceId" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="围栏名字">
					<el-input v-model="addForm.fenceName"></el-input>
				</el-form-item>
				<el-form-item label="监控对象">
					<el-input v-model="addForm.fenceMonitoredPerson"></el-input>
				</el-form-item>
				<el-form-item label="行政区关键字">
					<el-input  v-model="addForm.fenceKeyword"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getlistFencePage } from '../../api/api';

	export default {
		data() {
			return {
				filters: {
					fenceId: -1
				},
				fences: [],
				total: 0,
				page: 1,
				pageSize: 1,
				listLoading: false,
				sels: [],//列表选中列

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					// name: [
					// 	{ required: true, message: '请输入姓名', trigger: 'blur' }
					// ]
				},
				//编辑界面数据
				editForm: {
					id: 0,
					fenceId: -1,
					fenceName: '',
					fenceMonitoredPerson: '',
					fenceKeyword: '',
				},

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
					fenceId: -1,
					fenceName: '',
					fenceMonitoredPerson: '',
					fenceKeyword: '',
				}

			}
		},
		methods: {
			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getFences();
			},
			//获取用户列表
			getFences() {
				let para = {
					pageIndex: this.page,
					pageSize: this.pageSize
				};
				this.listLoading = true;
				//NProgress.start();
				getlistFencePage(para).then((res) => {
					console.log("ddd", res)
					// this.total = res.data.total;
					this.total = 2;
					this.fences = res.data.data.content;
					console.log("fec:", this.fences)
					this.listLoading = false;
					//NProgress.done();
				}).catch(err=>{
					console.log("错误信息:", err)
				});
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { id: row.id };
					removeUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getFences();
					});
				}).catch(() => {

				});
			},
			//显示编辑界面
			handleEdit: function (index, row) {
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
			},
			//显示新增界面
			handleAdd: function () {
				this.addFormVisible = true;
				this.addForm = {
					name: '',
					sex: -1,
					age: 0,
					birth: '',
					addr: ''
				};
			},
			//编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.editForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							editUser(para).then((res) => {
								this.editLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['editForm'].resetFields();
								this.editFormVisible = false;
								this.getFences();
							});
						});
					}
				});
			},
			//新增
			addSubmit: function () {
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.addForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							addUser(para).then((res) => {
								this.addLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['addForm'].resetFields();
								this.addFormVisible = false;
								this.getFences();
							});
						});
					}
				});
			},
			selsChange: function (sels) {
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { ids: ids };
					batchRemoveUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getFences();
					});
				}).catch(() => {

				});
			}
		},
		mounted() {
			this.getFences();
		}
	}

</script>

<style scoped>

</style>
