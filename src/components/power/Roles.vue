<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{path: '/'}">首页</el-breadcrumb-item>
      <el-breadcrumb-item :to="{path: 'home'}">权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <!-- 添加角色对话框 -->
      <el-dialog
        title="添加角色"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed"
      >
        <!-- 内容主题区 -->
        <el-form
          :model="addRoleForm"
          :rules="addRoleFormRules"
          ref="addRoleFormRef"
          label-width="100px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="addRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRole">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过v-for渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="285px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeRoleById(scope.row.id)"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 修改角色对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 内容主题区 -->
      <el-form
        :model="editRoleForm"
        :rules="editRoleFormRules"
        ref="editRoleFormRef"
        label-width="100px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="30%"
      @close="setRightDialogClosed"
    >
      <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolesList: [],
      addDialogVisible: false,
      editDialogVisible: false,
      setRightDialogVisible: false,
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      addRoleFormRules: {
        roleName: [{ required: true, message: '请输入角色名称', trigger: 'blur' }]
      },
      editRoleFormRules: {
        roleName: [{ required: true, message: '请输入角色名称', trigger: 'blur' }]
      },
      editRoleForm: {},
      // 所有权限的数据
      rightsList: [],
      // 树形控件属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的权限id数组
      defKeys: [],
      roleId: ''
    }
  },
  created() {
    // 获取所有角色列表数据
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
    },
    addRole() {
      this.$refs.addRoleFormRef.validate(async valid => {
        if (!valid) return
        // 可以放起添加用户的请求
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加角色失败！')
        }
        this.$message.success('添加角色成功！')
        // 隐藏添加用户对话框
        this.addDialogVisible = false
        // 刷新列表
        this.getRolesList()
      })
    },
    addDialogClosed() {
      // 重置表单项信息
      this.$refs.addRoleFormRef.resetFields()
    },
    editRole() {
      this.$refs.editRoleFormRef.validate(async valid => {
        if (!valid) return
        // 可以发起修改角色的请求
        const { data: res } = await this.$http.put(
          'roles/' + this.editRoleForm.roleId,
          {
            roleName: this.editRoleForm.roleName,
            roleDesc: this.editRoleForm.roleDesc
          }
        )
        if (res.meta.status !== 201) {
          this.$message.error('修改角色失败！')
        }
        this.$message.success('修改角色成功！')
        // 隐藏修改用户对话框
        this.editDialogVisible = false
        // 刷新列表
        this.getRolesList()
      })
    },
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色信息失败！')
      }
      this.editRoleForm = res.data
      this.editDialogVisible = true
    },
    editDialogClosed() {
      // 重置表单项信息
      this.$refs.editRoleFormRef.resetFields()
    },
    async removeRoleById(id) {
      // 弹框询问用户是否删除数据
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消了删除！')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败！')
      }
      this.$message.success('成功删除！')
      this.getRolesList()
    },
    // 根据id删除权限
    async removeRightById(role, rightId) {
      // 弹框询问用户是否删除数据
      const confirmResult = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消了删除！')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败！')
      }
      this.$message.success('成功删除！')
      role.children = res.data
    },
    // 展示分配角色对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取权限列表
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }
      this.rightsList = res.data
      // 递归获取三级节点
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下的三级权限id并保存到defKeys数组中
    getLeafKeys(node, arr) {
      // 如果当前节点不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights `, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('权限分配失败！')
      }
      this.$message.success('权限分配成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eeeeee;
}
.bdbottom {
  border-bottom: 1px solid #eeeeee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
