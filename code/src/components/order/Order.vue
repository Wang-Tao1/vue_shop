<template>
 <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>

        <el-row>
            <el-col :span="8">
                <el-input placeholder="请输入内容">
                <el-button slot="append" icon="el-icon-search"></el-button>
                </el-input>
            </el-col>
        </el-row>

        <el-table :data="orderlist" style="width: 100%" border stripe>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="order_number" label="订单编号"></el-table-column>
            <el-table-column prop="order_price" label="订单价格"></el-table-column>
            <el-table-column label="是否付款" prop="pay_status">
                <template scope="scope">
                    <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
                    <el-tag type="danger" v-else>未付款</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="is_send" label="是否发货"></el-table-column>
            <el-table-column label="下单时间" prop="create_time">
                <template scope="scope">
                    {{scope.row.create_time | dateFormat}}
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template>
                    <el-button size="mini" type="primary" icon="el-icon-edit" @click="showBox"></el-button>
                    <el-button size="mini" type="success" icon="el-icon-location" @click="showProgressBox"></el-button>
                </template>
            </el-table-column>
    </el-table>


        <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>

    </el-card>

    <el-dialog title="修改地址" :visible.sync="addressVisible" width="50%" @close="addressDialogClosed">
  <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader :options="cityData" v-model="addressForm.address1"  :props="{ expandTrigger: 'hover' }"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addressVisible  = false">取 消</el-button>
    <el-button type="primary" @click="addressVisible  = false">确 定</el-button>
  </span>
</el-dialog>

 <el-dialog title="物流进度" :visible.sync="progressVisible" width="50%">
     <el-timeline>
        <el-timeline-item v-for="(activity, index) in progressInfo" :key="index" :timestamp="activity.time">
          {{activity.context}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>

 </div>
</template>
 
<script>
import cityData from './citydata.js'
export default {
 name: '',
 data () {
 return {
     queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderlist: [],
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      progressVisible: false,
      progressInfo: []
 }
 },
 methods:{
    async getOrderList(){
        const {data : res } = await this.$http.get('orders',{
            params:this.queryInfo
        })
        if(res.meta.status!==200){
            return this.$message.error('获取订单列表失败!')
        }
        this.total = res.data.total
        this.orderlist = res.data.goods
    },
    handleSizeChange(newSize){
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPage){
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    showBox(){
        this.addressVisible = true
    },
    addressDialogClosed(){
        this.$refs.addressFormRef.resetFields()
    },
    showProgressBox(){
         this.$http.get('http://localhost:8080/data/index.json').then(res=>{
         this.progressInfo = res.data.data
         this.progressVisible = true
        }).catch(err=>{
            return this.$message.error('获取物流进度失败！')
        })      
    }
 },
 created() {
      this.getOrderList()
 }
}
</script>
 
<style scoped>
 .el-cascader {
  width: 100%;
}
</style>