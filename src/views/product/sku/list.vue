<template>
  <div>
    <el-table v-loading="loading" style="margin: 20px 0" :data="skuList" border>
      <el-table-column label="序号" type="index" width="50" align="center" />

      <el-table-column prop="skuName" label="名称" />

      <el-table-column prop="skuDesc" label="描述" />

      <el-table-column label="默认图片"  width="150" align="center">
        <template slot-scope="scope">
          <img :src="scope.row.skuDefaultImg" style="width: 100px; height:80px;" />
        </template>
      </el-table-column>

      <el-table-column prop="weight" label="重量(KG)" />

      <el-table-column prop="price" label="价格(元)" />

      <el-table-column label="操作"  width="300" align="center">
        <template slot-scope="{row, $index}" >
          <el-button v-if="row.isSale===1" type="warning" size="mini" icon="el-icon-bottom" @click="cancelSale(row)" title="下架"></el-button>
          <el-button v-else type="success" size="mini" icon="el-icon-top" @click="onSale(row)" title="上架"></el-button>
          <el-button type="primary" size="mini" icon="el-icon-edit" @click="showUpdate(row)" title="修改"></el-button>
           <el-button type="primary" size="mini" icon="el-icon-info" @click="get(row)" title="查看详情"></el-button>
          <el-button type="danger" size="mini" icon="el-icon-delete" @click="remove(row)" title="删除"></el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination style="text-align: center" :current-page="page" :page-sizes="[3, 6, 9]" :page-size="limit"
      :total="total" layout="prev, pager, next, jumper, ->, sizes, total" @current-change="getSkuList"
      @size-change="handleSizeChange" />
  </div>
</template>

<script>
export default {
  name: 'SkuList',

  data(){
    return{
      skuList:[],
      total: 0, // 总数量
      page: 1, // 当前页码
      limit: 3, // 每页数量
      loading: false, // 是否正在请求中,
    }
  },

  mounted(){
    this.getSkuList()
  },

  methods:{
    //删除指定id的sku  remove (skuId)
    remove (sku) {
      this.$confirm(`确认删除${sku.skuName}嘛？`,'提示',{
        type:'warning'
      }).then(async ()=>{
        const result = await this.$API.sku.remove(sku.id)
        if(result.code===200){
          this.$message.success('删除成功！')
          this.getSkuList(this.skuList.length===1&&this.page>1?this.page-1:this.page)
        }else{
          this.$message.error('删除失败！')
        }
      }).catch(()=>{
        this.$message.info('已取消删除')
      })
    },

    //下架  cancelSale (skuId)
     async cancelSale(sku){
      const result = await this.$API.sku.cancelSale(sku.id)
      if(result.code===200){
        this.$message.success('下架成功！')
         sku.isSale===1
        this.getSkuList(this.skuList.length===1&&this.page>1?this.page-1:this.page)
      }else{
        this.$message.error('下架失败！')
      }
    },

    //上架  onSale (skuId)
    async onSale(sku){
      const result = await this.$API.sku.onSale(sku.id)
      if(result.code===200){
        this.$message.success('上架成功！')
        console.log(sku.isSale)
        sku.isSale===0
        this.getSkuList(this.skuList.length===1&&this.page>1?this.page-1:this.page)
      }else{
        this.$message.error('上架失败！')
      }
    },

    //修改指定id的sku
    showUpdate(skuId){
      this.$message.warning('待完成')
    },

    //查看当前id的sku的详情
    get(skuId){

    },


    //修改每页数量的监听回调
    handleSizeChange(pageSize) {
        // 更新limit
        this.limit = pageSize
        // 重新获取第1页显示
        this.getSkuList(1) // 也可不传1
    },

    //异步获取指定页的列表数据显示
     async getSkuList(page = 1) {
        // 更新当前页码
        this.page = page
        this.loading = true // 显示loading
        // 调用接口请求函数发获取列表数据的请求
        const result = await this.$API.sku.getList(page, this.limit)
        this.loading = false // 隐藏loading
        // 如果成功了, 更新数据显示
        if (result.code === 200) {
          const {
            records,
            total
          } = result.data
          this.skuList = records
          this.total = total
        } else { // 如果失败, 提示
          /*
          this.$message({
            type: 'error',
            message: '获取分页列表失败'
          }) */
          this.$message.error('获取分页列表失败')
        }
      }
  }

}
</script>
