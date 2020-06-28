<template>
  <el-form label-width="100px">
    <el-form-item label="SPU名称">
      <span>{{spu.spuName}}</span>
    </el-form-item>

    <el-form-item label="SKU名称">
      <el-input type="text" placeholder="SKU名称" v-model="skuInfo.skuName"></el-input>
    </el-form-item>

    <el-form-item label="价格">
      <el-input type="number" placeholder="SKU价格" v-model="skuInfo.price"></el-input>
    </el-form-item>

    <el-form-item label="重量(kg)">
      <el-input type="number" placeholder="SKU重量" v-model="skuInfo.weight"></el-input>
    </el-form-item>

    <el-form-item label="规格描述">
      <el-input type="textarea" placeholder="SKU规格描述"  v-model="skuInfo.skuDesc"></el-input>
    </el-form-item>

    <el-form-item label="平台属性">
      <el-form inline lable-width="100px" >
        <el-form-item :label="attr.attrName" style="margin:5px 0" v-for="attr in attrList" :key="attr.id">
          <el-select placeholder="请输入" v-model="attr.attrIdValueId">
           <el-option  :label="va.valueName" :value="`${attr.id}:${va.id}`" v-for="va in attr.attrValueList" :key="va.id"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
    </el-form-item>

    <el-form-item label="销售属性">
      <el-form inline lable-width="100px">
        <el-form-item :label="sp.saleAttrName" style="margin:5px 0" v-for="sp in spuSaleAttrList" :key="sp.id">
          <el-select placeholder="请输入" v-model="sp.saleAttrValueId">
            <el-option :label="list.saleAttrValueName" :value="list.id" v-for="list in sp.spuSaleAttrValueList" :key="list.id"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
    </el-form-item>

    <el-form-item label="图片列表">
      <el-table border :data="spuImageList"  @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column label="图片">
          <template slot-scope="{row}">
            <img :src="row.imgUrl" alt="" style="width:100px;height:100px">
          </template>
        </el-table-column>
        <el-table-column label="名称" prop="imgName"></el-table-column>
        <el-table-column label="操作">
           <template slot-scope="{row,$inndex}">
             <el-tag type="successs" v-if="row.isDefault==='1'">默认</el-tag>
           <el-button type="primary" size="small" @click="setDefault(row)" v-else>设为默认</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-form-item>

    <el-form-item>
      <el-button type="primary" @click="save">保存</el-button>
      <el-button @click="back">取消</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
import { category } from '../../../api'
export default {
  name: 'skuForm',

  props:{
    cancel:Function
  },

  data(){
    return{
      spu:{},
      skuInfo:{
        category3Id: null, // 3级分类ID
        spuId:'',
        tmId: null, // spu的品牌id

        skuName: '', // sku名称
        skuDesc: '',
        price:'',
        weight:'',

        skuDefaultImg:null,
        skuAttrValueList:[],
        skuSaleAttrValueList: [], // sku的销售属性列表
        skuImageList: [], // sku图片列表
      },
      attrList:[],
      spuSaleAttrList:[],
      spuImageList:[],
      selectedSpuImageList:[]
    }
  },

  methods:{

    //重置数据
    resetDate(){
       this.spu={}
      this.skuInfo={
        category3Id: null, // 3级分类ID
        spuId:'',
        tmId: null, // spu的品牌id

        skuName: '', // sku名称
        skuDesc: '',
        price:'',
        weight:'',

        skuDefaultImg:null,
        skuAttrValueList:[],
        skuSaleAttrValueList: [], // sku的销售属性列表
        skuImageList: [], // sku图片列表
      }
      this.attrList=[]
      this.spuSaleAttrList=[]
      this.spuImageList=[]
      this.selectedSpuImageList=[]
    },

    back(){
      this.resetDate()
      this.cancel()
    },

    //保存当前的Spu信息
    async save(){
      const {skuInfo,attrList,spuSaleAttrList,selectedSpuImageList}=this

      attrList.forEach(attr=>{
        if(attr.attrIdValueId){
          const [attrId,ValueId] = attr.attrIdValueId.split(':')
          skuInfo.skuAttrValueList.push({
            attrId,ValueId
          })
        }
      })

      skuInfo.skuSaleAttrValueList=spuSaleAttrList.reduce((pre,attr)=>{
        if(attr.saleAttrValueId){
          pre.push({saleAttrValueId:attr.saleAttrValueId})
        }
        return pre
      },[])

      skuInfo.skuImageList=selectedSpuImageList.map(item=>({
        imgName:item.imgName,
        imgUrl:item.imgUrl,
        spuImgId:item.id,
        isDefault:item.isDefault
      }))


      await this.$API.sku.addUpdate(skuInfo)
      this.$message.success('保存sku成功')
      //重置数据
      this.resetDate()
      this.$emit('success')
    },

    //勾选项的状态发生改变的事件监听回调
    handleSelectionChange(val){
      this.selectedSpuImageList=val
    },

    //将指定图片设置成默认的
    setDefault(image){
      this.spuImageList.forEach(item=>item.isDefault='0')
      image.isDefault='1'
      //收集数据
      this.skuInfo.skuDefaultImg=image.imgUrl
    },

    initLoadAddData(spu){
      this.spu=spu

      this.skuInfo.category3Id=spu.category3Id
      this.skuInfo.spuId=spu.id
      this.skuInfo.tmId=spu.tmId

      this.getAddData()
    },

    getAddData(){
      const {category1Id,category2Id,category3Id,id}=this.spu

      const promise1=this.$API.attr.getList(category1Id,category2Id,category3Id)
      const promise3=this.$API.sku.getSpuImageList(id)
      const promise2= this.$API.sku.getSpuSaleAttrList(id)
      Promise.all([promise1,promise2,promise3]).then(results=>{
        const attrList=results[0].data
        const spuSaleAttrList=results[1].data
        const spuImageList=results[2].data.map(item=>({...item,isDefault:'0'}))

        this.attrList=attrList
        this.spuSaleAttrList=spuSaleAttrList
        this.spuImageList=spuImageList
      })
    },

  }
}
</script>

<style lang="less" scoped>

</style>
