<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索框卡片 -->
    <el-card class="box-card">
      <div>
        <el-row :gutter="20">
          <el-col :span="9">
            <el-input
              placeholder="请输入内容"
              v-model="queryInfo.query"
              clearable
              @clear="clearSearch"
            >
              <el-button
                @click="search"
                slot="append"
                icon="el-icon-search"
              ></el-button>
            </el-input>
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="addDialog">添加用户</el-button>
          </el-col>
        </el-row>
      </div>
    </el-card>
    <!-- 弹出的对话框 -->
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      @close="addDialogClosed"
      width="50%"
    >
      <!-- 对话框主体区域 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="70px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input
            v-model="addForm.username"
            :disabled="title == '修改用户'"
          ></el-input>
        </el-form-item>
        <el-form-item v-if="title == '添加用户'" label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 对话框尾部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 用户列表 -->
    <el-table stripe border :data="userList" style="width: 100%">
      <el-table-column type="index"></el-table-column>
      <el-table-column
        prop="username"
        label="用户名"
        width="180"
      ></el-table-column>
      <el-table-column
        prop="role_name"
        label="角色"
        width="180"
      ></el-table-column>
      <el-table-column prop="email" label="邮址"></el-table-column>
      <el-table-column
        prop="mobile"
        label="手机号码"
        width="180"
      ></el-table-column>
      <el-table-column prop="mg_state" label="状态">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.mg_state"
            @change="userStateChanged(scope.row)"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180px">
        <template slot-scope="scope">
          <el-button
            @click="editDialog(scope.row.id)"
            type="primary"
            size="mini"
            icon="el-icon-edit"
          ></el-button>
          <el-tooltip
            class="item"
            effect="dark"
            content="分配角色"
            placement="top"
            :enterable="false"
          >
            <el-button
              @click="handleClick(scope.row)"
              type="warning"
              size="mini"
              icon="el-icon-setting"
            ></el-button>
          </el-tooltip>
          <el-button
            @click="removeUserById(scope.row.id)"
            type="danger"
            size="mini"
            icon="el-icon-delete"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="this.queryInfo.pagenum"
      :page-sizes="[1, 2, 10, 20]"
      :page-size="this.queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="this.total"
    >
    </el-pagination>
  </div>
</template>

<script>
export default {
  data() {
    // 验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        return cb()
      }
      // 返回一个错误提示
      cb(new Error('请输入合法的邮箱'))
    }
    // 验证手机号码的规则
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return cb()
      }
      // 返回一个错误提示
      cb(new Error('请输入合法的手机号码'))
    }
    return {
      // 获取查询用户信息的参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      // 保存请求回来的用户列表数据
      userList: [],
      total: 0,
      dialogVisible: false,
      title: '',
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单的验证规则对象
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名在3~10个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          {
            validator: checkEmail,
            message: '邮箱格式不正确，请重新输入',
            trigger: 'blur'
          }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          {
            validator: checkMobile,
            message: '手机号码不正确，请重新输入',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      this.userList = res.data.users
      this.total = res.data.total
    },

    handleClick(data) {
      console.log(data)
    },

    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getUserList()
    },

    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getUserList()
    },

    async userStateChanged(user) {
      const { data: res } = await this.$http.put(
        `users/${user.id}/state/${user.mg_state}`
      )
      if (res.meta.status !== 200) {
        user.mg_state = !user.mg_state
        return this.$message.console.error('修改状态失败！')
      }
      this.$message.success('更新状态成功！')
    },

    search() {
      this.getUserList()
    },

    clearSearch() {
      this.queryInfo.query = ''
      this.getUserList()
    },

    addDialog() {
      this.title = '添加用户'
      this.dialogVisible = true
    },

    async editDialog(id) {
      this.title = '修改用户'
      // 发送请求根据id获取用户信息
      const { data: res } = await this.$http.get('users/' + id)
      // 判断如果添加失败，就做提示
      if (res.meta.status < 200 || res.meta.status > 300) {
        console.log(res, '----', id)
        return this.$message.error('获取用户信息失败')
      }
      // 将获取到的数据保存到数据editForm中
      this.addForm = res.data
      // 显示弹出窗
      this.dialogVisible = true
    },

    addDialogClosed() {
      // 对话框关闭之后，重置表达
      this.$refs.addFormRef.resetFields()
    },

    addUser() {
      // 点击确定按钮，添加新用户
      // 调用validate进行表单验证
      this.$refs.addFormRef.validate(async (valid) => {
        // let res
        if (!valid) return this.$message.error('请填写完整用户信息')
        if (this.title === '添加用户') {
          // 发送请求完成添加用户的操作
          const { data: res } = await this.$http.post('users', this.addForm)
          // 判断如果添加失败，就做提示
          if (res.meta.status < 200 || res.meta.status > 300) return this.$message.error('添加用户失败')
          // 添加成功的提示
          this.$message.success('添加用户成功')
        }
        if (this.title === '修改用户') {
          // 发送请求完成修改用户的操作
          const { username, email, mobile } = this.addForm
          const { data: res } = await this.$http.put(
            'users/' + this.addForm.id,
            { username, email, mobile }
          )
          // 判断如果添加失败，就做提示
          if (res.meta.status < 200 || res.meta.status > 300) return this.$message.error('修改用户失败')
          // 添加成功的提示
          this.$message.success('修改用户成功')
        }
        // 关闭对话框
        this.dialogVisible = false
        // 重新请求最新的数据
        this.getUserList()
      })
    },

    async removeUserById(id) {
      // 弹出确定取消框，是否删除用户
      const confirmResult = await this.$confirm(
        '请问是否要永久删除该用户',
        '删除提示',
        {
          confirmButtonText: '确认删除',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      // 如果用户点击确认，则confirmResult 为'confirm'
      // 如果用户点击取消, 则confirmResult获取的就是catch的错误消息'cancel'
      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除')
      }
      // 发送请求根据id完成删除操作
      const { data: res } = await this.$http.delete('users/' + id)
      // 判断如果删除失败，就做提示
      if (res.meta.status !== 200) return this.$message.error('删除用户失败')
      // 修改成功的提示
      this.$message.success('删除用户成功')
      // 重新请求最新的数据
      this.getUserList()
    }
  }
}
</script>

<style>
</style>
