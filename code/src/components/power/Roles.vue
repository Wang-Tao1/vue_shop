<template>
 <div>
 
 <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

<el-card>

 <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
 </el-row>


    <el-table :data="rolelist" style="width: 100%" stripe border>
        <el-table-column type="expand">
            <template slot-scope="scope">
                <el-row  :class="['bdbottom', index1 == 0?'bdtop':'','vcenter']" v-for="(item1,index1) in scope.row.children" :key="item1.id">
                    <el-col :span="5">
                        <el-tag @close="removeRightById(scope.row, item1.id)" closable>{{item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="19">
                        <el-row :class="[index2 === 0?'':'bdtop','vcenter']" v-for="(item2,index2) in item1.children" :key="item2.id">
                            <el-col :span="6">
                                <el-tag @close="removeRightById(scope.row, item2.id)" closable type="success">{{item2.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="18">
                                <el-tag @close="removeRightById(scope.row, item3.id)" closable type="warning" v-for="item3 in item2.children" :key="item3.id">{{item3.authName}}</el-tag>
                            </el-col>
                        </el-row>
                    </el-col>
                </el-row>


            </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
            <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
</el-card>

<el-dialog
  title="分配权限"
  :visible.sync="setRightDialogVisible"
  width="40%"
  @close="setRightDialogClosed">
  <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id"
  :default-checked-keys="defKeys" ref="treeRef" default-expand-all></el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>





 </div>
</template>
 
<script>
export default {
 name: '',
 data () {
 return {
    rolelist: [],
    setRightDialogVisible:false,
    rightslist: [],
    treeProps: {
          children: 'children',
          label: 'authName'
    },
    defKeys: [],
    roleId: ''
 }
 },
created() {
    this.getRolesList()
  },
  methods:{
    async getRolesList(){
        const { data : res} = await this.$http.get('roles')
       if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
       this.rolelist = res.data
    },
    async removeRightById(role, rightId){
        const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(confirmResult !== 'confirm'){
            return this.$message.info('取消了删除！')
        }
        const  { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(res.meta.status !== 200){
             return this.$message.error('删除权限失败！')
        }
       role.children = res.data
    },
    async showSetRightDialog(role){
      this.roleId = role.id
      const { data : res } = await this.$http.get('rights/tree')
      if(res.meta.status !== 200) return this.$message.error('获取权限数据失败！')
      this.rightslist = res.data
      this.getLeafKeys(role)
      this.setRightDialogVisible = true
    },
    getLeafKeys(node) {
     if(!node.children){
       return this.defKeys.push(node.id)
     }
      node.children.forEach(item => this.getLeafKeys(item))
    },
    setRightDialogClosed(){
      this.defKeys = []
    },
    async allotRights(){
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data : res }  = await this.$http.post(`roles/${this.roleId}/rights`,{
        rids : idStr
      })
      if(res.meta.status !== 200){
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>
 
<style scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>