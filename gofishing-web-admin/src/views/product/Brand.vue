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
			<el-table-column prop="firstLetter" label="首字母" width="120" sortable>
			</el-table-column>
			<el-table-column prop="productType.name" label="商品类型" width="120" sortable>
			</el-table-column>
			<el-table-column prop="logo" label="品牌logo" width="130" sortable>
				<template scope="scope">
					<img :src="'http://10.11.185.56'+scope.row.logo" width="50px" height="50px">
				</template>
			</el-table-column>
			<el-table-column prop="createTime" label="创建时间" width="118"  :formatter="createTime" sortable>
			</el-table-column>
			<el-table-column prop="updateTime" label="修改时间" min-width="118" :formatter="updateTime"  sortable>
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
				<el-form-item label="商品类型" prop="productTypes">
					<el-cascader
							v-model="saveForm.productTypes"
							expand-trigger="hover"
							:options="options"
							:props="defaultParams"
							@change="handleChange">
					</el-cascader>
				</el-form-item>
				<el-form-item label="公司Logo" prop="logo">
					<el-upload
							class="upload-demo"
							action="http://localhost:8888/services/common/file"
							:on-success="handleSuccess"
							:before-upload="handleBefore"
							:on-remove="handleRemove"
							:file-list="fileList"
							list-type="picture">
						<el-button size="small" type="primary">点击上传</el-button>
					</el-upload>
				</el-form-item>
				<el-form-item label="品牌描述">
					<el-input type="textarea" v-model="saveForm.description"></el-input>
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
				options:[],
				filters: {
					keyword: ''
				},
				defaultParams: {
					label: 'name',
					value: 'id',
					children: 'children'
				},
				fileList:[],
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
					],
					englishName: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					],
					productTypes: [
						{type:'array', required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				saveForm: {
					name: '',
					englishName: '',
					productTypes: [],
					description: '',
					productTypeId: 0,
					logo: '',
				},

			}
		},
		methods: {
			//文件上传之前
			handleBefore(file){
				if(this.fileList.length>0){
					this.$message({
						message: '只能上传一张logo图片',
						type: 'warning'
					});
					return false;//停止上传
				}
			},
			//文件上传之后
			handleSuccess(response, file, fileList){
				let {success,message,object} = response;
				console.log("response==02",response);
				console.log("file==02",file);
				console.log("fileList==02",fileList);
				if(success){
					this.saveForm.logo = object;
					this.$message({
						message: '上传成功',
						type: 'success'
					});
				}else{
					this.$message({
						message: message,
						type: 'error'
					});
				}
				this.fileList = fileList;
			},
			//Logo文件删除
			handleRemove() {
				this.fileId = this.saveForm.logo;
				this.$http.delete("/common/file?fileId="+this.fileId).then((res)=>{
					let {success,message} = res.data;
					if (success){
						this.$message({
							message: 'Logo删除成功',
							type: 'success'
						});
					}else{
						this.$message({
							message: message,
							type: 'error'
						});
					}
				});
				this.fileList=[]
			},

			//创建时间
			createTime(row){
				return this.formatTime(row.createTime);
			},
			//修改时间
			updateTime(row){
				return this.formatTime(row.updateTime);

			},
			formatTime(time){
				if(!time){
					return null;
				}
				let date = new Date(time);
				let year = date.getFullYear();
				let month = date.getMonth()+1;
				let day = date.getDate();//获取一个月中的第几天
				let hour = date.getHours();
				let minute = date.getMinutes();
				let second = date.getSeconds();
				let timeStr = year+"-"+this.formatTimeNum(month)+"-"+this.formatTimeNum(day)
						+" "+this.formatTimeNum(hour)+":"+this.formatTimeNum(minute)+":"+this.formatTimeNum(second);
				return timeStr;
			},
			formatTimeNum(num){
				if(num>=10){
					return num;
				}
				return "0"+num;
			},
			//查询商品类型的联级
			handleChange(value) {
				this.$http.get("/product/productType/loadProductTypeTree")
						.then((res)=>{
							this.options = this.getTreeData(res.data);
						})
			},
			// 递归判断列表，把最后的children设为undefined
			getTreeData(data){
				for(var i = 0;i<data.length;i++){
					if(data[i].children.length<1){
						// children若为空数组，则将children设为undefined
						data[i].children = undefined;
					}else {
						// children若不为空数组，则继续 递归调用 本方法
						this.getTreeData(data[i].children);
					}
				}
				return data;
			},
			//分页
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
					this.saveForm.logo = row.logo;
					this.$http.delete("/product/brand/delete/"+row.id)
							.then((res)=>{
								this.listLoading = false;
								let {success,message,object}=res.data;
								if (success){
									this.$message({
										message: message,
										type: 'success'
									});
									this.handleRemove();
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
				console.debug(row);
				var a = row.productType.path.split('.');
				a = a.splice(1,a.length-2);
				for (let i = 0; i < a.length; i++) {
					a[i] = parseInt(a[i]);
				}
				this.saveForm.logo = "http://10.11.185.56" + row.logo;
				this.saveFormVisible = true;
				this.saveForm = Object.assign({}, row);
				this.saveForm.productTypes = a;
			},
			//显示新增界面
			handleAdd: function () {
				this.saveFormVisible = true;
				this.saveForm.logo = '';
				this.saveForm = {};
			},
			//编辑和新增
			saveSubmit: function () {
				this.$refs.saveForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.saveLoading = true;
							this.saveForm.productTypeId = this.saveForm.productTypes[this.saveForm.productTypes.length-1];
							let para = Object.assign({}, this.saveForm);
							console.debug(para);
							// 传数据到后台
							this.$http.post("/product/brand/save", para).then(result=>{
								if (result.data.success) {
									this.$message({
										message: '提交成功',
										type: 'success'
									});
								} else {
									this.$message({
										message: result.data.message,
										type: 'error'
									});
								}
								this.saveLoading = false;
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
								let {success,message}=res.data;
								if (success){
									this.$message({
										message: message,
										type: 'success'
									});
									for(let i = 0; i < this.sels.length; i++) {
										this.saveForm.logo = this.sels[i].logo;
										this.handleRemove();
									}
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
			this.handleChange();
		}
	}

</script>

<style scoped>

</style>