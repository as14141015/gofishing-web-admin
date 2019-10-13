<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getBrands">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="brands" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55"></el-table-column>
			<el-table-column type="index" width="60"></el-table-column>
			<el-table-column prop="name" label="商品名称" width="120" sortable>
			</el-table-column>
			<el-table-column prop="englishName" label="英文名" width="120"  sortable>
			</el-table-column>
			<el-table-column prop="firstLetter" label="字母头" width="100" sortable>
			</el-table-column>
			<el-table-column prop="productType.name" label="商品类型" width="120" sortable>
			</el-table-column>
			<el-table-column prop="logo" label="品牌logo" min-width="160" sortable>
				<template scope="scope">
					<img src="scope.row.logo" width="50px" height="50px">
				</template>
			</el-table-column>
			<el-table-column prop="createTime" label="创建时间" min-width="118" sortable>
			</el-table-column>
			<el-table-column prop="updateTime" label="修改时间" min-width="118" sortable>
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleSave(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="rows" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑和新增界面-->
		<el-dialog title="编辑/新增" v-model="saveFormVisible" :close-on-click-modal="false">
			<el-form :model="saveForm" label-width="80px" :rules="saveFormRules" ref="saveForm">
				<el-form-item label="商品名称" prop="name">
					<el-input v-model="saveForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="英文名称" prop="englishName">
					<el-input v-model="saveForm.englishName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="首字母" prop="firstLetter">
					<el-input v-model="saveForm.firstLetter" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label=".." prop="description">
					<el-input v-model="saveForm.description" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="类型" prop="product_type_id">
					<el-radio-group v-model="saveForm.product_type_id">
						<el-radio class="radio" :label="1">男</el-radio>
						<el-radio class="radio" :label="0">女</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="sortIndex" prop="sortIndex">
					<el-input v-model="saveForm.sortIndex" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="公司Logo" prop="logo">
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="saveFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="saveSubmit" :loading="saveLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, saveUser, addUser } from '../../api/api';

	export default {
		data() {
			return {
				filters: {
					keyword: ''
				},
				brands: [],
				total: 0,
				page: 1,
				rows: 10,
				listLoading: false,
				sels: [],//列表选中列

				saveFormVisible: false,//编辑界面是否显示
				saveLoading: false,
				saveFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				saveForm: {
					name: '',
					englishName: '',
					firstLetter: 0,
					description: '',
					product_type_id: '',
					sortIndex: '',
					logo: '',
				},

			}
		},
		methods: {
			handleCurrentChange(val) {
				this.page = val;
				this.getBrands();
			},
			//获取商品列表
			getBrands() {
				let para = {
					page: this.page,
					rows: this.rows,
					keyword: this.filters.keyword
				};
				this.listLoading = true;
				//NProgress.start();
				this.$http.post("/product/brand/json",para).then((res)=>{
					this.listLoading = false;
					let {total,rows} = res.data;
					this.brands = rows;
					this.total = total;
				})
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					this.$http.delete("/product/brand/delete/"+row.id)
							.then((res)=>{
								this.listLoading = false;
								let {success,message,object}=res.data;
								if (success){
									this.$message({
										message: message,
										type: 'success'
									});
									this.getBrands();
								}else {
									this.$message({
										message: message,
										type: 'error'
									});
								}
							})
				}).catch(() => {
					this.$message({
						message: "系统繁忙..",
						type: 'error'
					});
				});
			},
			//显示编辑界面
			handleSave: function (index, row) {
				this.saveFormVisible = true;
				this.saveForm = Object.assign({}, row);
			},
			//显示新增界面
			handleAdd: function () {
				this.saveFormVisible = true;
				this.saveForm = {};
			},
			//编辑和新增
			saveSubmit: function () {
				this.$refs.saveForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.saveLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.saveForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							saveUser(para).then((res) => {
								this.saveLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['saveForm'].resetFields();
								this.saveFormVisible = false;
								this.getBrands();
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
					this.$http.delete("/product/brand/deleteBatch?ids="+ids)
							.then((res)=>{
								this.listLoading = false;
								let {success,message,object}=res.data;
								if (success){
									this.$message({
										message: message,
										type: 'success'
									});
									this.getBrands();
								}else {
									this.$message({
										message: message,
										type: 'error'
									});
								}
							})
				}).catch(() => {
					this.$message({
						message: "系统繁忙...",
						type: 'error'
					});
				});
			}
		},
		mounted() {
			this.getBrands();
		}
	}

</script>

<style scoped>

</style>