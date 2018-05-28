<template>
	<div class="AdminPcLogin">
		<PcHeader></PcHeader>
		<div class="container">
			<Table
				:data="devList"
				border
				style="width: 100%">
				<TableColumn
				prop="id"
				label="编号"
				width="60">
				</TableColumn>
				<TableColumn
				prop="username"
				label="姓名"
				width="100">
					<template slot-scope="scope">
						<Popover trigger="hover" placement="top">
							<p>姓名: {{ scope.row.username }}</p>
							<p>电话: {{ scope.row.tel }}</p>
							<p>设备: {{ scope.row.dev_name }}</p>
							<p>维修员: {{ scope.row.repairtor }}</p>
							<p>学院: {{ scope.row.xueyuan }}</p>
							<p>专业: {{ scope.row.zhuanye }}</p>
							<p>开始时间: {{ scope.row.time_s }}</p>
							<p style="max-width: 200px;">备注: {{ scope.row.content }}</p>
							<img :src="'http://repaire.dnpuzi.com/' + scope.row.qrcode_url" alt="" style="width: 120px;">
							<div slot="reference" class="username-wrapper">
								<Tag size="medium">{{ scope.row.username }}</Tag>
							</div>
						</Popover>
					</template>
				</TableColumn>
				<TableColumn
				prop="tel"
				label="电话"
				width="120">
				</TableColumn>
				<TableColumn
				prop="dev_name"
				label="设备"
				width="100">
				</TableColumn>
				<TableColumn
				prop="repairtor"
				label="维修人员"
				width="100">
				</TableColumn>
				<TableColumn
				prop="price"
				label="价格"
				width="100">
				</TableColumn>
				<TableColumn
				prop="status"
				label="状态"
				width="99">
					<template slot-scope="scope">
						{{['待店维修', '售后', '送修', '已完成'][Number(scope.row.status)-1]}}
					</template>
				</TableColumn>
				<TableColumn
				prop="time_s"
				sortable
				label="开始时间"
				width="160">
				</TableColumn>
				<TableColumn
				prop="content"
				label="描述"
				width="280">
				</TableColumn>
				<!-- <TableColumn
				prop="dev_name"
				label="二维码"
				width="150">
					<template slot-scope="scope">
						<img :src="'http://repaire.dnpuzi.com/' + scope.row.qrcode_url" alt="" style="width: 120px;">
					</template>
				</TableColumn> -->
				<TableColumn
				label="操作"
				width="80">
					<template slot-scope="scope">
						<Button @click="_PrintPreview(scope.row)" type="primary" size="small" style="margin-bottom: 10px;">打印</Button>
						<br>
						<Button v-show="scope.row.status != 4" @click="modify(scope.row)" type="primary" size="small" style="margin-bottom: 10px;">编辑</Button>
						<br>
						<Button v-show="scope.row.status != 4" type="primary" size="small" @click="inputPrice(scope.row)">完成</Button>
						<br>
						<Tag v-show="scope.row.status == 4" type="success">已完成</Tag>
					</template>
				</TableColumn>
			</Table>
		</div>
		<Dialog title="登录" :visible.sync="loginVisible">
			<Form :model="loginUserInfo" :rules="rules" ref="loginUserInfo">
				<FormItem label="店名" :label-width="formLabelWidth" prop="username">
					<Input v-model="loginUserInfo.username" auto-complete="off"></Input>
				</FormItem>
				<FormItem label="密码" :label-width="formLabelWidth" prop="password">
					<Input v-model="loginUserInfo.password" auto-complete="off"></Input>
				</FormItem>
			</Form>
			<div slot="footer" class="dialog-footer">
				<Button @click="loginVisible = false">取 消</Button>
				<Button type="primary"  @click="submitForm('loginUserInfo')">确 定</Button>
			</div>
		</Dialog>
		<Dialog title="编辑信息" :visible.sync="devInfoVisible" width="30%">
			<Form>
				<FormItem label="姓名" :label-width="devFormLabelWidth">
					<Input v-model="devInfo.username"></Input>
				</FormItem>
				<FormItem label="设备型号" :label-width="devFormLabelWidth">
					<Input v-model="devInfo.dev_name"></Input>
				</FormItem>
				<FormItem label="维修人员" :label-width="devFormLabelWidth">
					<Select v-model="devInfo.repairtor" placeholder="请选择" @change="changeDevInfo">
						<Option
						v-for="item in repairtorList"
						:key="item.username"
						:label="item.username"
						:value="item.username">
						</Option>
					</Select>
				</FormItem>
				<FormItem label="备注信息" :label-width="devFormLabelWidth">
					<Input v-model="devInfo.content"></Input>
				</FormItem>
				<FormItem label="电话" :label-width="devFormLabelWidth">
					<Input v-model="devInfo.tel"></Input>
				</FormItem>
				<FormItem label="学院" :label-width="devFormLabelWidth">
					<Select v-model="devInfo.xueyuan" placeholder="请选择" @change="changeDevInfo">
						<Option
						v-for="item in this.$store.state.xueyuan"
						:key="item.value"
						:label="item.label"
						:value="item.value">
						</Option>
					</Select>
				</FormItem>
				<FormItem label="专业" :label-width="devFormLabelWidth">
					<Input v-model="devInfo.zhuanye"></Input>
				</FormItem>
				<FormItem label="状态" :label-width="devFormLabelWidth">
					<Select v-model="devInfo.status" placeholder="请选择" @change="changeDevInfo">
						<Option
						v-for="item in this.$store.state.status"
						:key="item.value"
						:label="item.label"
						:value="item.value">
						</Option>
					</Select>
				</FormItem>
				<FormItem label="二维码" :label-width="devFormLabelWidth">
					<img :src="'http://repaire.dnpuzi.com/' + devInfo.qrcode_url" alt="" style="width: 120px;">
				</FormItem>
			</Form>
			<div slot="footer" class="dialog-footer">
				<Button @click="devInfoVisible = false">取 消</Button>
				<Button type="primary" @click="handleModify">确 定</Button>
			</div>
		</Dialog>
	</div>
</template>
<script>
import qs from 'qs';
import { Dialog, Form, FormItem, Input, Button, Select, Option, Table, TableColumn, Popover, Tag, Col } from 'element-ui';
import PcHeader from '@/components/pc/PcHeader';
export default {
	data() {
		return {
			loginVisible: false,
			devInfoVisible: false,
			loginUserInfo: {
				username: '',
				password: ''
			},
			repairtorList: [],
			devInfo: {},
			formLabelWidth: '60px',
			devFormLabelWidth: '100px',
			rules: {
				username: [
					{ required: true, message: '请输入店名',  trigger: 'blur' }
				],
				password: [
					{ required: true, message: '请输入密码',  trigger: 'blur' }
				]
			},
			devList: []
		}
	},
	name: 'AdminPcLogin',
	methods: {
		_SetprintParameters() {
	      var LODOP = getCLodop();
	      // console.log(LODOP);
	      let name = this.devInfo.username;
	      let tel = this.devInfo.tel;
	      let dev_name = this.devInfo.dev_name;
	      let status = this.devInfo.status == 3 ? '已完成' : '正在维修';
	      LODOP.SET_PRINT_PAGESIZE(1, 500, 750, ""); //规定纸张大小;
	      LODOP.SET_PRINT_STYLE("FontSize", 14); //字体大小  
	      LODOP.ADD_PRINT_TEXT(20, 20, 500, 39, "学长的电脑铺子"); //内容，参数（上边距，左边距，内容显示宽度，内容显示高度）
	      LODOP.SET_PRINT_STYLE("FontSize", 10); //更换大小  
	      LODOP.ADD_PRINT_TEXT(50, 12, 360, 39, "姓名：");
	      LODOP.ADD_PRINT_TEXT(50, 50, 360, 39, name);
	      LODOP.ADD_PRINT_TEXT(70, 12, 360, 39, "电话：");
	      LODOP.ADD_PRINT_TEXT(70, 50, 360, 39, tel);
	      LODOP.ADD_PRINT_TEXT(90, 12, 360, 39, "机型：");
	      LODOP.ADD_PRINT_TEXT(90, 50, 360, 39, dev_name);
	      LODOP.ADD_PRINT_TEXT(110, 12, 360, 39, "维修：");
	      LODOP.ADD_PRINT_TEXT(110, 50, 360, 39, status);
	      LODOP.ADD_PRINT_IMAGE(130,30,100,100,"<img border='0' src='http://repaire.dnpuzi.com/" + this.devInfo.qrcode_url + "' />");
	      LODOP.SET_PRINT_STYLEA(0,"Stretch",2);
	    },
	    _PrintPreview(scope) {
	      // this._SetprintParameters();
	      // LODOP.PREVIEW();//打印预览  
	      console.log(scope);
	    },
		copy(obj){
			var newobj = {};
			for ( var attr in obj) {
			newobj[attr] = obj[attr];
			}
			return newobj;
		},
		changeDevInfo(key) {
			if(key) {
				this.repairtorList.forEach((item) => {
					if(item.username == key) {
						this.devInfo.repairtor_id = item.id
					}
				})
			}
			this.devInfo = this.copy(this.devInfo);
		},
		handleModify() { //提交修改的信息
			// console.log(this.devInfo);
			this.$axios.post('http://repaire.dnpuzi.com/Admin/index/modifyDevInfo', qs.stringify(this.devInfo))
			.then(res => {
				if(res.data == 1) {
					this.devInfo = {};
					this.devInfoVisible = false;
					location.reload();
				}
			})
		},
		modify(row) {
		 	this.devInfoVisible = true;
			this.devInfo.id = row.id;
			this.devInfo.dev_name = row.dev_name;
			this.devInfo.repairtor = row.repairtor;
			this.devInfo.content = row.content;
			this.devInfo.username = row.username;
			this.devInfo.tel = row.tel;
			this.devInfo.xueyuan = row.xueyuan;
			this.devInfo.zhuanye = row.zhuanye;
			// this.devInfo.price = row.price;
			// this.devInfo.time_s = row.time_s;
			this.devInfo.status = row.status;
			this.devInfo.qrcode_url = row.qrcode_url;
			console.log(this.devInfo);
		},
		inputPrice(row) {
			// console.log(row);
			this.$prompt('请输入价格', '', {
				confirmButtonText: '确定',
				cancelButtonText: '取消',
				inputType: 'Number',
			}).then(({ value }) => {
				if(value) {
					this.$alert('维修金额：￥' + value, '确认完成？', {
						confirmButtonText: '确定',
						cancelButtonText: '取消',
						callback: action => {
							if(action == 'confirm') {
								let data = {
									id: row.id,
									price: value
								};
								this.$axios.post('http://repaire.dnpuzi.com/Admin/index/devOrderDone', qs.stringify(data))
								.then(res => {
									if(res.data.code == 1 && res.data.msgReturn.msg == '成功') {
										this.$notify({
											title: '完成提示',
											message: '已确认',
											type: 'success',
											duration: 2000
										});
									} else {
										this.$notify({
											title: '短信提示',
											message: '发送失败',
											type: 'warning',
											duration: 2000
										});
									}
								})
							}
						}
					})
				} else {
					this.$message({
						type: 'info',
						message: '取消输入'
					});
				}
			}).catch(() => {
				this.$message({
					type: 'info',
					message: '取消输入'
				});       
			});
		},
		submitForm(formName) {
			this.$refs[formName].validate((valid) => {
				if (valid) {
					let data = {
						username: this.loginUserInfo.username,
						password: this.loginUserInfo.password
					}
					// console.log(data);
					this.$axios.post('http://repaire.dnpuzi.com/Admin/index/Login', qs.stringify(data))
					.then(res => {
						if(res.data.code == 0) {
							this.$notify({
								title: '登录提示',
								message: '找不到该用户',
								type: 'warning',
								duration: 2000
							});
						} else if(res.data.code == 1) {
							this.$notify({
								title: '登录提示',
								message: '密码错误',
								type: 'warning',
								duration: 2000
							});
						} else {
							this.loginUserInfo.username = '';
							this.loginUserInfo.password = '';
							this.loginVisible = false;
							let userInfo = {
								id: res.data.data.id,
								username: res.data.data.username
				            };
				            localStorage.setItem('userInfo', JSON.stringify(userInfo));
							this.$notify({
								title: '登录提示',
								message: '登录成功',
								type: 'success',
								duration: 2000
							});
							location.reload();
						}
					})
				}
			});
		}
	},
	created() {
		if(localStorage.getItem('userInfo')) {
			let userInfo = JSON.parse(localStorage.getItem('userInfo'));
        	this.$store.dispatch("changeUserInfo", userInfo);
        	this.$axios('http://repaire.dnpuzi.com/Admin/index/adminShowDevList')
        	.then(res => {
        		this.devList = res.data.data;
        	})
		} else {
			this.loginVisible = true;
		}
	},
	mounted() {
		this.$axios('http://repaire.dnpuzi.com/Admin/index/showRepairtorList')
    	.then(res => {
    		this.repairtorList = res.data;
    	})
	},
	components: {
		PcHeader,
		Dialog, Form, FormItem, Input, Button, Select, Option, Table, TableColumn, Popover, Tag, Col
	}
}
</script>
<style scoped lang="scss">
	.AdminPcLogin {
		.container {
			width: 1200px;
			margin: 0 auto;
		}
	}
</style>