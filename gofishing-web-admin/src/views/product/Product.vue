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

		<!-- 显示属性模态窗 -->
		<el-dialog size="tiny" title="显示属性" v-model="viewPropertiesShow" :close-on-click-modal="false">
			<el-form label-width="80px">
				<el-form-item v-for="viewProperty in viewProperties"
							  :label="viewProperty.specName">
					<el-input v-model="viewProperty.value" auto-complete="off">
					</el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="viewPropertiesShow = false">取消</el-button>
				<el-button type="primary" @click.native="viewSubmit" :loading="saveLoading">提交</el-button>
			</div>
		</el-dialog>
		<!-- sku属性模态窗 -->
		<el-dialog title="sku属性" v-model="skuPropertiesShow" :close-on-click-modal="false">
			<!--外层循环展示sku属性名-->
			<el-card class="box-card" v-for="(skuProperty,ind) in skuProperties">
				<div slot="header" class="clearfix">
					<span style="line-height: 36px;">{{skuProperty.specName}}</span>
				</div>
				<!--内层循环展示sku属性的选项-->
				<div v-for="index in skuProperty.options.length+1" class="text item">
					<el-row>
						<el-col :span="18">
							<el-input v-model="skuProperty.options[index-1]" autocomplete="off"></el-input>
						</el-col>
						<el-col :span="6">
							<el-button :plain="true" type="danger" @click="removeProperty(ind,index-1)">&nbsp;-&nbsp;</el-button>
						</el-col>
					</el-row>
				</div>
			</el-card>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="skuPropertiesShow = false">取消</el-button>
				<el-button type="primary" @click.native="skuSubmit" :loading="saveLoading">提交</el-button>
			</div>
			<!--根据上面的属性动态生成表格-->
			<el-table :data="skus" highlight-current-row style="width: 100%;">
				<el-table-column v-if="key!='price'&&key!='availableStock' && key!='indexs'" v-for="(value,key) in skus[0]" :label="key" :prop="key">
				</el-table-column>
				<el-table-column v-if="key=='price'||key=='availableStock'" v-for="(value,key) in skus[0]" :label="key" :prop="key">
					<template scope="scope">
						<el-input v-model="scope.row[key]" auto-complete="off"/>
					</template>
				</el-table-column>
			</el-table>
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
				viewPropertiesShow:false,//显示属性界面是否显示
				viewProperties:[],//获取查询来显示属性的数据
				skuPropertiesShow:false,//sku属性界面是否显示
				skuProperties:[],
				skus:[],//用来装经过算法返回的值
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
			handleViewProperties(){
				if(this.sels[0].state == 1){
					this.$message({
						message: '该商品上架中，请先下架再维护！！',
						type: 'warning'
					});
					return;
				}
				//只能选中一行数据
				if(this.sels.length==0){
					this.$message({
						message: '请选中一行数据',
						type: 'warning'
					});
					return;
				}
				if(this.sels.length>1){
					this.$message({
						message: '只能选中一行数据',
						type: 'warning'
					});
					return;
				}
				let productId = this.sels[0].id;
				//查询要维护商品的显示属性
				this.$http.get("/product/product/viewProperties/"+productId)
						.then(res=>{
							this.viewProperties = res.data;
						});
				this.viewPropertiesShow = false;
			},
			//显示属性提交
			viewSubmit(){
				let productId = this.sels[0].id;
				this.$confirm('确认保存吗?', '提示', {
					type: 'warning'
				}).then(() => {
						this.$http.post("/product/product/updateViewProperties?productId="+productId,this.viewProperties)
							.then(res=>{
								let {success,message} = res.data;
								if(success){
									this.$message({
										message: '保存成功!',
										type: 'success'
									});
									this.viewPropertiesDialogVisible =false;
								}else{
									this.$message({
										message: message,
										type: 'error'
									});
								}
							})
				}).catch(() => {
				});
			},
			//sku属性维护
			handleSkuProperties(){
				if(this.sels[0].state == 1){
					this.$message({
						message: '该商品上架中，请先下架再维护！！',
						type: 'warning'
					});
					return;
				}
				//只能选中一行数据
				if(this.sels.length==0){
					this.$message({
						message: '请选中一行数据',
						type: 'warning'
					});
					return;
				}
				if(this.sels.length>1){
					this.$message({
						message: '只能选中一行数据',
						type: 'warning'
					});
					return;
				}
				let productId = this.sels[0].id;
				//查询要维护商品的显示属性
				this.$http.get("/product/product/skuProperties/"+productId)
						.then(res=>{
							this.skuProperties = res.data.skuProperties;
							this.skus =  res.data.skus;
						});
				this.skuPropertiesShow = true;
			},
			//sku属性提交
			skuSubmit(){
				let productId = this.sels[0].id;
				let parameter = {};
				parameter.skuProperties = this.skuProperties;
				console.debug("skuProperties",this.skuProperties);
				parameter.skus = this.skus;
				this.$confirm('确认保存吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.$http.post("/product/product/updateSkuProperties?productId="+productId,parameter)
							.then(res=>{
								let {success,message} = res.data;
								if(success){
									this.$message({
										message: '保存成功!',
										type: 'success'
									});
									this.skuPropertiesShow =false;
								}else{
									this.$message({
										message: message,
										type: 'error'
									});
								}
							})
				}).catch(() => {
				});
			},
			//sku属性删除选项
			removeProperty(index1,index2){
				//index1 第几个sku属性 index2 当前属性的第几个选项
				this.skuProperties[index1].options.splice(index2,1);
			},
			//商品上架
			handleOnSale(){
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认上架选中商品吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					this.$http.post("/product/product/onSale?ids="+ids).then(res=>{
						this.listLoading = false;
						let {success,message}=res.data;
						if (success){
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
			},
			//商品下架
			handleOffSale(){
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认上架选中商品吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					this.$http.post("/product/product/offSale?ids="+ids).then(res=>{
						this.listLoading = false;
						let {success,message}=res.data;
						if (success){
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
			},
			//上架时间格式
			formatOnSaleTime(row){
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
				if(row.state == 1){
					this.$message({
						message: '该商品上架中，请先下架再删除！！',
						type: 'warning'
					});
					return;
				}
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
				if(row.state == 1){
					this.$message({
						message: '该商品上架中，请先下架再修改！！',
						type: 'warning'
					});
					return;
				}
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
				for(let i=0;i<this.sels.length;i++){
					if (this.sels[i].state == 1){
						this.$message({
							message: '删除商品中存在已上架商品，请先下架！！',
							type: 'error'
						});
						return;
					}
				}
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
		//核心方法，监听skuProperties属性值的变化
		watch:{
			skuProperties:{
				handler(){
					//过滤掉options为空数组的sku属性
					let skuPropertiesArr = this.skuProperties.filter(e=>e.options.length>0);
					let result = skuPropertiesArr.reduce((pre,cur,currentIndex)=>{
						//创建一个空数组来获取值
						let temp = [];
						let i = 0;
						pre.forEach(e1=>{ //e1 {} 第一次是为初始的空，从第二次开始就是上次循环的结果值
								cur.options.forEach((e2,index)=>{ //e2 从数组的第一个参数开始
									let obj = Object.assign({},e1);
									obj[cur.specName] = e2; //获取每个specName的值
									let lastIndexs = obj.indexs;//获取上一次的indexs,后面拼接这一次的索引
									//如果没有值就设置为一个字符串
									if(!lastIndexs) lastIndexs = "";
									//判断是否是最后一次循环 如果是最后一次，拼接价格和库存
									if(currentIndex==skuPropertiesArr.length-1){
										if (i<this.skus.length){
											obj.price = this.skus[i].price;
											obj.availableStock = this.skus[i].availableStock;
											i++;
										}else {
											obj.price = 0;
											obj.availableStock = 0;
										}
										lastIndexs = lastIndexs+index;
									}else {
										lastIndexs = lastIndexs+index+"_";
									}
									obj.indexs = lastIndexs;
									temp.push(obj);
								})
							});
							return temp;
					},[{}]);
					this.skus = result;//赋值给data中的数组

				},
				deep:true
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