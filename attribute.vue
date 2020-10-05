<!-- 
开发者：耿哥贡献
邮箱：2515677459@qq.com
注意：该代码用于动态生成电商平台的sku，如果需要
运行，请把该代码放在vue脚手架里，并需要下载element组件，
new this.mytitle....该代码删除，可自定义提示
 -->

<template>
	<div>
		<div class="information">销售属性</div>
		<div class="home-list">
			<div class="home-title">输入尺码(中国码)</div>
			<div class="home-sku">
				<el-input v-model="skusize" placeholder="请输入尺码"></el-input>
				<el-button type="primary" class="button-ing" @click="skuFun()">确定</el-button>
			</div>
			<!-- 添加 -->
			<div class="add-to">
				<el-tag
				class="eltab"
				  v-for="(tag,index) in skusizedata"
				  :key="index"
				  type="success"
				  effect="plain"
				  closable
				  @close="handleSize(index,0)">
				  {{tag}}
				</el-tag>
			</div>
		</div>
		<!-- 颜色 -->
		<div class="home-list">
			<div class="home-title">输入颜色</div>
			<div class="home-sku">
				<el-input  v-model="skucolour" placeholder="请输入颜色"></el-input>
				<!-- 图片上传 -->
				<el-upload
				  class="upload-demo"
				  action="http://localhost:9000/api/potimg"
				  name="file"
				  :show-file-list="false"
				  :on-success="onSuccess"
				  :on-error="onErr"
				  >
				<img v-if="imageup == ''" src="../../../images/icon/upimg.svg" class="upimges"/>
				<img v-else :src="imageup" class="upimges"/>
				</el-upload>
				<el-button type="primary" @click="colourfun()">确定</el-button>
			</div>
			<!-- 输入的颜色 -->
			<div class="add-to">
				<el-tag
				class="eltab"
				  v-for="(tag,index) in skucolourdata"
				  :key="index"
				  type="success"
				  effect="plain"
				  closable
				  @close="handleSize(index,1)">
				  {{tag.color}}
				  <img :src="tag.img" class="tagimges"/>
				</el-tag>
			</div>
		</div>
		<!-- 匹配库存尺码 -->
		<div class="home-list">
			<div class="home-title">价格与库存</div>
			<div class="add-to">销售属性自动为你组合成SKU，请选择需要的SKU</div>
			<div v-for="(item, index) in skudata" :key="index">
				<div class="checklist">
					<el-checkbox v-model="item.checked" class="checkman"></el-checkbox>
					<div>{{item.size}}——{{item.color}}——</div>
					<img :src="item.img" v-show="item.img != ''"/>
				</div>
				<div class="home-sku sku-men">
					<div><el-input  v-model="item.Price" placeholder="请输入价格"></el-input></div>
					<div><el-input  v-model="item.stock" placeholder="请输入库存"></el-input></div>
				</div>
			</div>
		</div>
		<!-- 提交 -->
		<div class="home-list">
		  <el-row>
		    <el-button type="success" @click="btNs()">提交</el-button>
		  </el-row>
		</div>
	</div>
</template>

<script>
const {log} = console
// const upimg = require('../../../images/upimges.svg')
export default{
	data() {
		return {
			// 尺码
			skusize:'',
			skusizedata:[],
			// 颜色
			skucolour:'',
			// 上传的图片
			imageup:'',
			skucolourdata:[],
			imgindex:'',
			checked: false,
			// 存储待整合sku数据1
			skulist: [
			{ name: '尺码', data: [] },
			{ name: '颜色', data: [] }
			],
			// 存储待整合sku数据1
			skumen:[],
			// sku组合
			skudata:[],
			skuIng:[]
		}
	},
	methods:{
		// 删除某个尺码,颜色
		handleSize(tag,inx){
			log(tag)
			inx == 0 ? this.skusizedata.splice(tag,1) : this.skucolourdata.splice(tag,1)
			this.skulist[inx].data.splice(tag,1)
			this.stArtsku()
		},
		
		// 添加尺寸
		skuFun(){
			if(this.skusize == '')return false
			this.skusizedata.push(this.skusize)
			this.skulist[0].data.push(this.skusize)
			this.skusize = ''
			this.stArtsku()
		},
		
		// 添加颜色
		colourfun(){
			if(this.skucolour == ''){
				new this.mytitle(this.$message,'info','请添加颜色').funtitle()
			}else if(this.imageup == ''){
				new this.mytitle(this.$message,'info','请上传图片').funtitle()
			}else{
				this.skucolourdata.push({'color':this.skucolour,'img':this.imageup})
				this.skulist[1].data.push({'color':this.skucolour,'img':this.imageup})
				this.skucolour = ''
				this.imageup = ''
				this.stArtsku()
			}
		},
		
		// 生成sku:重组一个多维数组
		stArtsku(){
			this.skumen = []
			this.skuIng = []
			log(this.skulist)
			this.skulist.forEach(item => {
			item.data.length > 0 ? this.skumen.push(item.data) : ''
			})
			log(this.skumen)
			// 数组为空就不再调用
			if (this.skumen.length > 0){
			this.generAte([], 0, this.skumen)	
			}
		},
		
		// 生成sku
		generAte(mensku = [], i, list){
			for (let j = 0; j < list[i].length; j++) {
			if (i < list.length - 1) {
				 mensku[i] = list[i][j]
				 this.generAte(mensku, i + 1, list) // 递归轮询
			}
			else {
				  // 递归还会走到这里
				  this.skuIng.push({...mensku, ...list[i][j]}) // 合并对象
				  let vbsku = this.skuIng.map(item=>{
					  let cvb = {
						  'size':item[Object.keys(item)[0]] == item.color ? '' : item[Object.keys(item)[0]],
						  'color':item.color == undefined ? '' : item.color,
						  'img':item.img == undefined ? '' : item.img,
						  'checked':false,
						  'Price':'',
						  'stock':''
					  }
					  return cvb
				  })
				  log(vbsku)
				  this.skudata = vbsku
			}
			}
		},
		
		// 上传成功
		onSuccess(res){
			log(res)
			this.imageup = res.data
			// this.$set(this.skucolour[this.imgindex],'img',res.data)
		},
		// 上传图片
		indeFun(index){
			// log(index)
			this.imgindex = index
		},
		// 上传失败
		onErr(){
			new this.mytitle(this.$message,'info','上传失败,请重试').funtitle()
		},
		
		
		// 提交
		btNs(){
			log(this.skudata)
		},
	},
	
	
}	
</script>

<style scoped="scoped">
@import '../../../../style/table.css';
a：hover{cursor:pointer}
.information{font-weight: bold; font-size: 20px;
margin-bottom: 30px;
}
.upimges{width: 40px !important; height: 40px !important;}
/deep/ .el-upload--picture-card{
  width: 100px;
  height: 100px;
}
/deep/ .el-upload{
  width: 50px;
  height: 40px;
  line-height: 40px;
}
/deep/ .el-upload-list--picture-card .el-upload-list__item{
  width: 100px;
  height: 100px;
  line-height: 100px;
}
/deep/ .el-upload-list--picture-card .el-upload-list__item-thumbnail{
  width: 100px;
  height: 100px;
  line-height: 100px;
}
/deep/ .avatar{
  width: 100px;
  height: 100px;
}
.home-sku img{width: 20px; height: 20px;}
.home-sku{display: flex; align-items: center;
margin-bottom: 10px;
}
.checkman{margin-right: 10px;}
.checklist{display: flex; align-items: center; margin-bottom: 10px;}
.checklist img{width: 30px; height: 30px;}
.add-to{color: #0086B3; font-size: 14px; margin: 20px 0;}
.add-width{width: 50px;}
.sku-men div:nth-child(1){margin-right: 10px;}
.button-ing{margin-left: 10px;}
.eltab{margin: 0 7px 7px 0;}
.tagimges{width: 20px; height: 20px;}
</style>
