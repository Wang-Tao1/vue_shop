<template>
 <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false"></el-alert>

        <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader
                v-model="selectedCateKeys"
                :options="catelist"
                :props="cateProps"
                @change="handleChange">
          </el-cascader>
        </el-col>
      </el-row>

    <el-tabs v-model="activeName" @tab-click="handleTabClick">
    <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
           <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template scope="scope">
                <el-tag v-for="(item, index) in scope.row.attr_vals" :key="index" closable @close="handleClose(index, scope.row)">{{item}}</el-tag>
                <el-input
  class="input-new-tag"
  v-if="scope.row.inputVisible"
  v-model="scope.row.inputValue"
  ref="saveTagInput"
  size="small"
  @keyup.enter.native="handleInputConfirm(scope.row)"
  @blur="handleInputConfirm(scope.row)"
>
</el-input>
<el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
    </el-tab-pane>
    <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加属性</el-button>
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
               <template slot-scope="scope">
                <el-tag v-for="(item, index) in scope.row.attr_vals" :key="index" closable @close="handleClose(index, scope.row)">{{item}}</el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
    </el-tab-pane>
  </el-tabs>
    </el-card>

   <el-dialog :title="'添加' + titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>


 <el-dialog :title="'修改' + titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
 <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>

 </div>
</template>
 
<script>
export default {

 name: '',
 data () {
 return {
   catelist: [],
   selectedCateKeys:[],
   cateProps:{
      expandTrigger: 'hover',
      value: 'cat_id',
      label: 'cat_name',
      children:'children'
   },
   activeName: 'many',
   manyTableData: [],
   onlyTableData: [],
   addDialogVisible: false,
   addForm: {
      attr_name: ''
   },
   addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
  },
  editDialogVisible:false,
  editForm: {},
  editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
  }
 }
 },
 methods:{
    async getCateList(){
    const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) 
     return this.$message.error('获取商品分类失败！')
      this.catelist = res.data
     },
    handleChange(){
      this.getParamsData()
    },
    handleTabClick(){
        this.getParamsData()
    }, 
    async getParamsData() {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败！')
      }
      res.data.forEach(item=>{
        item.attr_vals = item.attr_vals ?item.attr_vals.split(','):[]
        item.inputVisible = false
        item.inputValue = ''
      })
       console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    addDialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    async addParams(){
      this.$refs.addFormRef.validate(async valid => {
        if(!valid) return
        const { data : res } = await this.$http.post(`categories/${this.cateId}/attributes`,{
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
         if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！')
        }
          this.$message.success('添加参数成功！')
          this.addDialogVisible = false
          this.getParamsData()
      })
    },
    editDialogClosed(){
      this.$refs.editFormRef.resetFields()
    },
    editParams(){
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
         const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
        )
          if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败！')
        }
        this.$message.success('修改参数成功！')
         this.getParamsData()
        this.editDialogVisible = false     
      })
    },
    async showEditDialog(attrId){
        const { data : res} = await this.$http.get(`categories/${this.cateId}/attributes/${attrId}`,{
          params:{
            attr_sel: this.activeName
          }
        })
        if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败！')      
      }
         this.editForm = res.data
        this.editDialogVisible = true  
    },
    async removeParams(attrId) {
       const confirmResult = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if(confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attrId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败！')
      }
       this.$message.success('删除参数成功！')
        this.getParamsData()
    },
    showInput(row){
      row.inputVisible = true
       this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
    },
    async handleInputConfirm(row){
     if(row.inputValue.trim().length === 0){
      row.inputValue = ''
      row.inputVisible = false
       return
     }
     row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrVals(row)
    },
    async saveAttrVals(row) {
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(',')
        })
        if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败！')
      }
      this.$message.success('修改参数项成功！')
    },
    handleClose(i,row){
      row.attr_vals.splice(i,1)
      this.saveAttrVals(row)
    }
 },
 created(){
   this.getCateList()
 },
 computed:{
     isBtnDisabled(){
       if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false  
     },
     cateId(){
    if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null 
     },
     titleText(){
       if(this.activeName === 'many'){
         return '动态参数'
       }
        return '静态属性'
     }
 }
}
</script>
 
<style scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag{
  margin: 10px;
}
.input-new-tag {
  width: 110px;
}
</style>