<template>
	<section>
		<!--工具条-->
		<el-col :span="24"
		    class="toolbar"
		    style="padding-bottom: 0px">
			<el-form :inline="true"
			    :model="filters">

				<el-form-item>
					<el-input v-model="filters.name"
					    placeholder="投诉类型名称"></el-input>
				</el-form-item>

				<el-form-item>
					<el-button type="primary"
					    v-on:click="getData">查询</el-button>
				</el-form-item>

				<el-form-item>
					<el-button type="primary"
					    @click="handleAdd"
						v-show="checkBtn('sys:complaintType:save')">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="data"
		    highlight-current-row
		    v-loading="listLoading"
		    @selection-change="selsChange"
		    style="width: 100%">
			<el-table-column type="selection"
			    width="55">
			</el-table-column>
			<el-table-column prop="id"
			    label="编号"
			    width="120"
			    sortable>
			</el-table-column>
			<el-table-column prop="name"
			    label="投诉类型名称"
			    width="200"
			    sortable>
			</el-table-column>

			<el-table-column label="操作"
			    width="150">
				<template slot-scope="scope">
					<el-button size="small"
					    @click="handleEdit(scope.$index, scope.row)"
						v-show="checkBtn('sys:complaintType:update')">编辑</el-button>
					<el-button type="danger"
					    size="small"
					    @click="handleDel(scope.$index, scope.row)"
						v-show="checkBtn('sys:complaintType:delete')">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24"
		    class="toolbar">
			<!-- <el-button type="danger"
			    @click="batchRemove"
			    :disabled="this.sels.length===0">批量删除</el-button> -->
			<el-pagination layout="prev, pager, next"
			    @current-change="handleCurrentChange"
			    :page-size="pageSize"
			    :total="total"
			    style="float:right">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="编辑"
		    :visible.sync="editFormVisible"
		    :close-on-click-modal="false">
			<el-form :model="editForm"
			    label-width="120px"
			    :rules="editFormRules"
			    ref="editForm">
				<el-form-item label="投诉类型名称"
				    prop="name">
					<el-input v-model="editForm.name"
					    auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer"
			    class="dialog-footer">
				<el-button @click.native="editFormVisible = false">取消</el-button>
				<el-button type="primary"
				    @click.native="editSubmit"
				    :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--新增界面-->
		<el-dialog title="新增"
		    :visible.sync="addFormVisible"
		    :close-on-click-modal="false">
			<el-form :model="addForm"
			    label-width="120px"
			    :rules="addFormRules"
			    ref="addForm">
				<el-form-item label="投诉类型名称"
				    prop="name">
					<el-input v-model="addForm.name"
					    auto-complete="off"></el-input>
				</el-form-item>

			</el-form>
			<div slot="footer"
			    class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary"
				    @click.native="addSubmit"
				    :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
// import util from '../../common/js/util'
import NProgress from 'nprogress'
import { getComplaintTypeListPage, findComplaintTypeByNameLike,deleteComplaintType, batchRemoveUser, editComplaintType, addComplaintType } from '../../../api/api'
import { checkBtns } from '../../../common/user';
export default {
	data() {
		return {
			filters: {
				name: ''
			},
			data: [],
			total: 0,
			page: 1,
			pageSize:20,
			listLoading: false,
			sels: [],//列表选中列

			editFormVisible: false,//编辑界面是否显示
			editLoading: false,
			editFormRules: {
				name: [
					{ required: true, message: '请输入投诉类型名称', trigger: 'blur' }
				]
			},
			//编辑界面数据
			editForm: {
				id: 0,
				name: ''
			},

			addFormVisible: false,//新增界面是否显示
			addLoading: false,
			addFormRules: {
				name: [
					{ required: true, message: '请输入投诉类型名称', trigger: 'blur' }
				]
			},
			//新增界面数据
			addForm: {
				name: ''
			}

		}
	},
	methods: {
		checkBtn: function (code) {
			// `this` 指向 vm 实例
			return checkBtns('complaintType-list', code)
		},
		handleCurrentChange(val) {
			this.page = val
			this.getData()
		},
		//获取用户列表
		getData() {
			let para = {
				page: this.page-1,
				size: this.pageSize
			}
			if (this.filters.name) {
				para = Object.assign({
					name: this.filters.name
				}, para)
			}
			this.listLoading = true
			NProgress.start()
			getComplaintTypeListPage(para).then((res) => {
				this.total = res.data.totalElements
				this.data = res.data.content
				this.listLoading = false
				NProgress.done()
			})
		},
		//删除
		handleDel: function (index, row) {
			this.$confirm('确认删除该记录吗?', '提示', {
				type: 'warning'
			}).then(() => {
				this.listLoading = true
				//NProgress.start()
				let para = { id: row.id }
				deleteComplaintType(para).then((res) => {
					this.listLoading = false
					//NProgress.done()
					this.$message({
						message: '删除成功',
						type: 'success'
					})
					this.getData()
				})
			}, () => { })
		},
		//显示编辑界面
		handleEdit: function (index, row) {
			this.editFormVisible = true
			this.editForm = Object.assign({}, row)
		},
		//显示新增界面
		handleAdd: function () {
			this.addFormVisible = true
			this.addForm = {
				name: ''
			}
		},
		//编辑
		editSubmit: function () {
			this.$refs.editForm.validate((valid) => {
				if (valid) {
					this.$confirm('确认提交吗？', '提示', {}).then(() => {
						this.editLoading = true
						NProgress.start()
						let para = Object.assign({}, this.editForm)
						editComplaintType(para).then((res) => {
							this.editLoading = false
							// NProgress.done()
							this.$message({
								message: '提交成功',
								type: 'success'
							})
							this.$refs['editForm'].resetFields()
							this.editFormVisible = false
							this.getData()
						})
					}, () => { })
				}
			})
		},
		//新增
		addSubmit: function () {
			this.$refs.addForm.validate((valid) => {
				if (valid) {
					this.$confirm('确认提交吗？', '提示', {}).then(() => {
						this.addLoading = true
						NProgress.start()
						let para = Object.assign({}, this.addForm)
						addComplaintType(para).then((res) => {
							this.addLoading = false
							NProgress.done()
							this.$message({
								message: '提交成功',
								type: 'success'
							})
							this.$refs['addForm'].resetFields()
							this.addFormVisible = false
							this.getData()
						})
					}, () => {

					})
				}
			})
		},
		selsChange: function (sels) {
			this.sels = sels
		},
		//批量删除
		batchRemove: function () {
			var ids = this.sels.map(item => item.id).toString()
			this.$confirm('确认删除选中记录吗？', '提示', {
				type: 'warning'
			}).then(() => {
				this.listLoading = true
				//NProgress.start()
				let para = { ids: ids }
				batchRemoveUser(para).then((res) => {
					this.listLoading = false
					//NProgress.done()
					this.$message({
						message: '删除成功',
						type: 'success'
					})
					this.getData()
				})
			}).catch(() => {

			})
		}
	},
	mounted() {
		this.getData()
	}
}

</script>

<style scoped>
</style>