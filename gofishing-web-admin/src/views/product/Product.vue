<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="white" v-on:click="getProducts">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleSave">新增</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="warning" @click="handleViewProperties">显示属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleSkuProperties">sku属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="success" @click="handleOnSale">商品上架</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="danger" @click="handleOffSale">商品下架</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="name" label="标题" width="180" sortable>
			</el-table-column>
			<el-table-column prop="subName" label="副标题" width="120" sortable>
			</el-table-column>
			<el-table-column prop="productType.name" label="类型" width="100" sortable>
			</el-table-column>
			<el-table-column prop="brand.name" label="商品名" width="110" sortable>
			</el-table-column>
			<el-table-column prop="onSaleTime" label="上架时间" width="115" :formatter="formatOnSaleTime" sortable>
			</el-table-column>
			<el-table-column prop="offSaleTime" label="下架时间" width="115" :formatter="formatOffSaleTime" sortable>
			</el-table-column>
			<el-table-column prop="state" label="状态" min-width="100" sortable>
				<template scope="scope">
					<span style="color:green;" v-if="scope.row.state == 1 ">上架</span>
					<span style="color:red;" v-else>下架</span>
				</template>
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
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="rows" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--新增界面-->

		<el-dialog title="新增" v-model="saveFormVisible" :close-on-click-modal="false">
			<el-form :model="saveForm" label-width="80px" :rules="saveFormRules" ref="saveForm">
				<el-form-item label="标题" prop="name">
					<el-input v-model="saveForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副标题" prop="subName">
					<el-input v-model="saveForm.subName" auto-complete="off"></el-input>
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
				<el-form-item label="品牌" prop="brandId">
					<!--<el-input v-model="saveForm.brandId" auto-complete="off">
                    </el-input>-->
					<el-select v-model="saveForm.brandId" clearable
							   placeholder="请选择品牌">
						<el-option
								v-for="item in brands"
								:label="item.name"
								:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="媒体属性">
					<el-upload
							class="upload-demo"
							action="http://localhost:8888/services/common/file"
							:on-success="handleSuccess"
							:on-remove="handleRemove"
							:file-list="fileList"
							list-type="picture">
					<el-button size="small" type="primary">点击上传</el-button>
					</el-upload>
				</el-form-item>
				<el-form-item label="商品描述">
					<el-input type="textarea" v-model="saveForm.ext.description"
							  auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品详情">
					<quill-editor
					v-model="saveForm.ext.richContent"
					ref="myQuillEditor"
					>
					</quill-editor>

				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="saveFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="saveSubmit" :loading="saveLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--编辑界面-->
<!--		<el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">-->
<!--			<el-form :model="editForm" label-width="80px" :rules="saveFormRules" ref="editForm">-->
<!--				<el-form-item label="标题" prop="name">-->
<!--					<el-input v-model="editForm.name" auto-complete="off"></el-input>-->
<!--				</el-form-item>-->
<!--				<el-form-item label="副标题" prop="subName">-->
<!--					<el-input v-model="editForm.subName" auto-complete="off"></el-input>-->
<!--				</el-form-item>-->
<!--				<el-form-item label="商品类型" prop="productTypes">-->
<!--					<el-cascader-->
<!--							v-model="editForm.productTypes"-->
<!--							expand-trigger="hover"-->
<!--							:options="options"-->
<!--							:props="defaultParams"-->
<!--							@change="handleChange">-->
<!--					</el-cascader>-->
<!--				</el-form-item>-->
<!--				<el-form-item label="品牌">-->
<!--					&lt;!&ndash;<el-input v-model="saveForm.brandId" auto-complete="off">-->
<!--                    </el-input>&ndash;&gt;-->
<!--					<el-select v-model="editForm.brandId" clearable-->
<!--							   placeholder="请选择品牌">-->
<!--						<el-option-->
<!--								v-for="item in brands"-->
<!--								:label="item.name"-->
<!--								:value="item.id">-->
<!--						</el-option>-->
<!--					</el-select>-->
<!--				</el-form-item>-->
<!--				<el-form-item label="媒体属性">-->
<!--					<el-upload-->
<!--							class="upload-demo"-->
<!--							action="http://localhost:8888/services/common/file"-->
<!--							:on-success="handleSuccess"-->
<!--							:on-remove="handleRemove"-->
<!--							:file-list="fileList"-->
<!--							list-type="picture">-->
<!--						<el-button size="small" type="primary">点击上传</el-button>-->
<!--					</el-upload>-->
<!--				</el-form-item>-->
<!--				<el-form-item label="商品描述">-->
<!--					<el-input type="textarea" v-model="editForm.ext.description"-->
<!--							  auto-complete="off"></el-input>-->
<!--				</el-form-item>-->
<!--				<el-form-item label="商品详情">-->
<!--					<quill-editor-->
<!--							ref="QuillEditor"-->
<!--							v-model="editForm.ext.richContent"></quill-editor>-->
<!--				</el-form-item>-->
<!--			</el-form>-->
<!--			<div slot="footer" class="dialog-footer">-->
<!--				<el-button @click.native="editFormVisible = false">取消</el-button>-->
<!--				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>-->
<!--			</div>-->
<!--		</el-dialog>-->


	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, saveUser, addUser } from '../../api/api';

	export default {
		data() {
			return {
				fileList: [],//用作回显
				fileListPics: [],//用作装图片遍历之后的字符串
				brands:[],
				filters: {
					keyword: ''
				},
				options:[],
				defaultParams: {
					label: 'name',
					value: 'id',
					children: 'children'
				},
				products: [],
				total: 0,
				page: 1,
				rows: 10,
				listLoading: false,
				sels: [],//列表选中列
				saveFormVisible: false,//新增界面是否显示
				saveLoading: false,

				//新增界面数据
				saveForm: {
					name: '',
					subName: '',
					productTypeId: null,
					brandId: null,
					medias:'',
					productTypes:[],//商品级联下拉框
					ext:{
						description:'',
						richContent:''
					}
				},
				saveFormRules: {
					name: [
						{ required: true, message: '请输入标题', trigger: 'blur' }
					],
					subName: [
						{ required: true, message: '请输入副标题', trigger: 'blur' }
					],
					productTypes: [
						{ type:"array",required: true, message: '请选择商品类型', trigger: 'blur' }
					]
				}
			}
		},
		methods: {
			//查询相应ext的对应的数据
			handleExt(id){
				this.$http.get("/product/productExt/productIdGetEid?id="+id)
						.then((res)=>{
							this.saveForm.ext = res.data;
						})
			},
			//Logo文件删除
			handleRemove() {
				this.$http.delete("/common/file?fileId="+this.fileId).then((res)=>{
					let {success,message,object} = res.data;
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
				})
			},
			//文件上传之后
			handleSuccess(response, file, fileList){
				let {success,message,object} = response;
				console.log("response==02",response);
				console.log("fileList==02",fileList);
				if(success){
					// this.saveForm.logo = object;
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
			//显示属性维护
			handleViewProperties(){},
			//sku属性维护
			handleSkuProperties(){},
			//商品上架
			handleOnSale(){},
			//商品下架
			handleOffSale(){},
			//上架时间格式
			formatOnSaleTime(row, column){
				return this.formatTime(row.onSaleTime);
			},
			//下架时间格式
			formatOffSaleTime(row, column){
				return this.formatTime(row.offSaleTime);
			},
			//设置时间格式
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
				+" "+this.formatTimeNum(hour)+":"+this.formatTimeNum(minute)+":"+this.formatTimeNum
				(second);
				return timeStr;
			},
			formatTimeNum(num){
				if(num>=10){
					return num;
				}
				return "0"+num;
			},
			//查询品牌
			getBrands(){
				this.$http.get("/product/brand/list")
						.then(res=>{
							this.brands = res.data;
						})
			},
			//查询商品类型的联级
			handleChange() {
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
				this.getProducts();
			},
			//获取商品列表
			getProducts() {
				let para = {
					page: this.page,
					rows: this.rows,
					keyword: this.filters.keyword
				};
				this.listLoading = true;
				this.$http.post("/product/product/json",para).then(res=>{
					this.listLoading = false;
					let {total,rows} = res.data;
					this.products = rows;
					this.total = total;
				})
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					this.$http.delete("/product/product/delete/"+row.id)
							.then((res)=>{
								this.listLoading = false;
								let {success,message}=res.data;
								if (success){
									this.$message({
										message: message,
										type: 'success'
									});
									this.getProducts();
									// this.saveForm.logo = row.logo;
									this.handleRemove();
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
			handleEdit: function (index, row) {
				this.handleExt(row.id);
				this.saveFormVisible = true;
				console.debug(row);
				this.saveForm = Object.assign({}, row);
				var a = row.productType.path.split('.');
				a = a.splice(1,a.length-2);
				for (let i = 0; i < a.length; i++) {
					a[i] = parseInt(a[i]);
				}
				this.fileListPics = row.medias.split(',');
				for (let f = 0; f < this.fileListPics.length; f++) {
					let pic={url:''};
					pic.url=this.fileListPics[f];
					this.fileList.push(pic);
				}

				console.debug("====",this.saveForm);
				this.saveForm.productTypes = a;
			},
			//显示新增界面
			handleSave: function () {
				this.saveFormVisible = true;
				this.saveForm = {
					name: '',
					subName: '',
					productTypeId: null,
					brandId: null,
					medias:'',//接收多个上传图片的值
					productTypes:[],//商品级联下拉框
					ext:{
						description:'',
						richContent:''
					}
				};
			},
			//编辑和新增
			saveSubmit: function () {
				this.$refs.saveForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.saveLoading = true;
							let para = Object.assign({}, this.saveForm);
							para.productTypeId = this.saveForm.productTypes[this.saveForm.productTypes.length-1];
							console.debug(this.fileList);
							para.medias =
									this.fileList.map(file=>file.response.object).join(",");

							console.debug("paar",para);
							// para.brandId =;
							this.$http.post("/product/product/save",para)
									.then((res)=>{
										this.saveLoading = false;
										if (res.data.success){
											this.$message({
												message: '操作成功',
												type: 'success'
											});
											this.$refs['saveForm'].resetFields();
											this.saveFormVisible = false;
											this.getBrands();
										}else {
											this.$message({
												message: res.data.message,
												type: 'error'
											});
										}
									});
							this.saveLoading = false;
							this.saveFormVisible = false;
							this.getProducts();
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
					this.$http.delete("/product/product/deleteBatch?ids="+ids).then(res=>{
						this.listLoading = false;
						let {success,message,object}=res.data;
						if (success){
							for (const sel of this.sels) {
								// this.saveForm.logo = sel.logo;获取logo中的地址
								this.handleRemove();
							}
							this.$message({
								message: message,
								type: 'success'
							});
							this.getProducts();
						}else {
							this.$message({
								message: message,
								type: 'error'
							});
						}
					})

				}).catch(() => {

				});
			}
		},
		mounted() {
			this.getProducts();
			this.handleChange();
			this.getBrands();
		}
	}

</script>

<style scoped>

</style>