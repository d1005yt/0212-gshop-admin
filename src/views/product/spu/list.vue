<template>
  <div>
    <el-card style="margin-bottom: 20px" v-show="!isShowSkuForm">
      <CategorySelector @categoryChange="handleCategoryChange"></CategorySelector>
    </el-card>
    <el-card>
      <div v-show="!isShowSpuForm&&!isShowSkuForm">
        <el-button type="primary" icon="el-icon-plus" @click="showSpuAdd">添加SPU</el-button>

        <el-table border :data="spuList">
          <el-table-column label="序号" type="index" width="80" align="center"></el-table-column>
          <el-table-column label="SPU名称" prop="spuName"></el-table-column>
          <el-table-column label="SPU描述" prop="description"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="{row, $index}">
              <hint-button title="添加SKU" type="primary" icon="el-icon-plus" size="mini"
              @click="showSkuAdd(row)"/>
              <hint-button title="修改SPU" type="primary" icon="el-icon-edit" size="mini"
              @click="showSpuUpdate(row)" />
              <hint-button title="查看所有SKU" type="info" icon="el-icon-info" size="mini"
              @click="showSkuList(row)"/>

              <el-popconfirm  :title="`确定删除${row.spuName}吗？`" @onConfirm="deleteSpu(row.id)">
                 <hint-button title="删除" type="danger" icon="el-icon-delete" size="mini"
                 slot="reference" />
              </el-popconfirm>

            </template>
          </el-table-column>
        </el-table>

        <el-pagination
          style="text-align: center"
          :current-page="page"
          :page-sizes="[5, 10, 15]"
          :page-size="limit"
          :total="total"
          layout="prev, pager, next, jumper, ->, sizes, total"
          @current-change="getSpuList"
          @size-change="handleSizeChange" />
      </div>

      <SpuForm :visible.sync="isShowSpuForm" ref="spuForm"
      @success="handleSuccess" @cancel="handleCancel"></SpuForm>
      <!-- <SpuForm :visible="isShowSpuForm" @update:visible="isShowSpuForm=$event"></SpuForm> -->
      <SkuForm v-show="isShowSkuForm" ref="spuForm" @success="isShowSkuForm=false" :cancel="()=>isShowSkuForm=false"></SkuForm>
    </el-card>

     <el-dialog :title="`${spu.spuName} 的SKU列表`" :visible.sync="isShowDialog">
      <el-table :data="skuList" v-loading="isLoading">
        <el-table-column prop="skuName" label="名称"></el-table-column>
        <el-table-column property="price" label="价格(元)"></el-table-column>
        <el-table-column property="weight" label="重量(KG)"></el-table-column>
        <el-table-column label="默认图片">
          <template slot-scope="{row}">
            <img :src="row.skuDefaultImg" alt="" style="width:100px;height:100px;">
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
import SpuForm from '../components/SpuForm'
import SkuForm from '../components/SkuForm'
export default {
  name: 'SpuList',

  data(){
    return{
      category1Id:'',
      category2Id:'',
      category3Id:'',
      spuList:[],
      total:0,
      page:1,
      limit:5,
      isShowSpuForm:false,
      isShowSkuForm:false,

      isShowDialog:false,
      spu:{},
      skuList:[],
      isLoading:false,
    }
  },

  async mounted () {
    // const result = await this.$API.spu.getList(1, 3, 61)
    // console.log('result---', result)
    this.category1Id = 2
    this.category2Id = 13
    this.category3Id = 61
    this.getSpuList(1)
  },

  methods:{
    //显示sku的添加界面
    showSkuAdd(spu){
      this.isShowSkuForm=true
      spu={...spu}
      spu.category1Id=this.category1Id
      spu.category2Id=this.category2Id
      this.$refs.spuForm.initLoadAddData(spu)
    },

    //显示指定spu下的sku列表
    async showSkuList(spu){
      this.isShowDialog=true
      this.spu=spu
      this.skuList=[]
      this.isLoading=true

      const result = await this.$API.sku.getListBySpuId(spu.id)
      this.isLoading=false
      const skuList = result.data
      this.skuList=skuList
    },

    //删除指定spu
    async deleteSpu(spuId){
     const result= await this.$API.spu.remove(spuId)
      this.$message.success('删除成功')
      this.getSpuList()
    },

    // 保存添加（更新）成功的事件回调
    handleSuccess(){
      this.getSpuList(this.spuId?this.page:1)
      this.spuId=''
    },

    // 取消保存的事件回调
    handleCancel(){
      this.spuId=''
    },

    //显示spu详细信息的修改
    showSpuUpdate(spu){
       this.spuId = spu.id // 保存更新的标识
      this.isShowSpuForm = true
      // 让SpuForm组件去加载更新的初始数据
      this.$refs.spuForm.initUpdateData(spu.id)
    },

     //显示spu详细信息的添加
    showSpuAdd(){
     // 显示SpuForm界面
      this.isShowSpuForm = true
      // 初始化加载添加SPU的数据
      this.$refs.spuForm.initAddData(this.category3Id)
    },


    //分类发生改变的回调
    handleCategoryChange({categoryId,level}){
      if(level===1){
        this.category2Id=''
        this.category3Id=''
        this.spuList=[]
        this.total=0
        this.category1Id = categoryId
      }else if(level===2){
        this.category3Id=''
        this.spuList=[]
        this.total=0
        this.category2Id = categoryId
      }else{
        this.category3Id=categoryId
        this.getSpuList(1)
      }
    },

    //异步获取指定页码SPU列表的显示
    async getSpuList(page=1){
      this.page=page
      const result=await this.$API.spu.getList(page,this.limit,this.category3Id)
      const {records,total} = result.data
      this.spuList=records
      this.total=total
    },

    //煤业数量改变的监听回调
    handleSizeChange(limit){
      this.limit=limit
      this.getSpuList()
    }
  },

  components:{
    SpuForm,
    SkuForm
  }
}
</script>
