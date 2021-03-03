<template>
  <div>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索 添加 -->
      <el-row :gutter="20">
        <el-col :span="6">
          <el-input v-model="queryInfo.query" placeholder="请输入内容" clearable @clear="getDataList">
            <el-button slot="append" icon="el-icon-search" @click="getDataList" />
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="datalist" border stripe>
        <!-- stripe: 斑马条纹
        border：边框-->
        <el-table-column type="index" label="#" />
        <el-table-column v-for="column in columns" :key="column.key" :prop="column.key" :label="column.title" />
        <!--        <el-table-column prop="username" label="姓名"></el-table-column>-->
        <!--        <el-table-column prop="email" label="邮箱"></el-table-column>-->
        <!--        <el-table-column prop="mobile" label="电话"></el-table-column>-->
        <!--        <el-table-column prop="role_name" label="角色"></el-table-column>-->
<!--        <el-table-column label="状态">-->
<!--          <template slot-scope="scope">-->
<!--            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)" />-->
<!--          </template>-->
<!--        </el-table-column>-->
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              circle
              @click="showEditDialog(scope.row.id)"
            />
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              circle
              @click="removeDataById(scope.row.id)"
            />
            <el-tooltip
              class="item"
              effect="dark"
              content="角色分配"
              :enterable="false"
              placement="top"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
                circle
                @click="showSetRole(scope.row)"
              />
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="totle"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </el-card>

    <!-- 添加用户的对话框 -->
    <el-dialog title="添加用户" :visible.sync="dialogVisible" width="50%" @close="dialogClosed">
      <!-- 内容主体 -->
      <el-form
        ref="formRef"
        :model="formData"
        :rules="formRules"
        label-width="100px"
      >
        <el-form-item v-for="form in forms" :key="form.key" :prop="form.key" :label="form.title">
          <!--        <el-form-item label="用户名" prop="username">-->
          <el-input v-model="formData.username" />
        </el-form-item>
        <!--        <el-form-item label="密码" prop="password">-->
        <!--          <el-input v-model="formData.password"></el-input>-->
        <!--        </el-form-item>-->
        <!--        <el-form-item label="邮箱" prop="email">-->
        <!--          <el-input v-model="formData.email"></el-input>-->
        <!--        </el-form-item>-->
        <!--        <el-form-item label="手机" prop="mobile">-->
        <!--          <el-input v-model="formData.mobile"></el-input>-->
        <!--        </el-form-item>-->
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { getList } from '@/api/table'
export default {
  data() {
    // 自定义邮箱规则
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        // 合法邮箱
        return callback()
      }
      callback(new Error('请输入合法邮箱'))
    }
    // 自定义手机号规则
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return callback()
      }
      // 返回一个错误提示
      callback(new Error('请输入合法的手机号码'))
    }
    return {
      // 获取用户列表查询参数对象
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 每页显示多少数据
        pagesize: 5
      },
      datalist: [],
      totle: 0,
      // 添加用户对话框
      dialogVisible: false,
      columns: [
        { key: 'cardScheme1', title: '用户', mapping: 'CARD_SCHEME' },
        { key: 'cardScheme2', title: '用户', mapping: 'CARD_SCHEME' },
        { key: 'cardScheme3', title: '用户', mapping: 'CARD_SCHEME' }
      ],
      forms: [
        {
          key: 'username',
          title: '用户',
          type: 'input',
          rule: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
            {
              min: 2,
              max: 10,
              message: '用户名的长度在2～10个字',
              trigger: 'blur'
            }
          ]
        },
        {
          key: 'password',
          title: '密码',
          type: 'input',
          rule: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
            {
              min: 2,
              max: 10,
              message: '用户名的长度在2～10个字',
              trigger: 'blur'
            }
          ]
        },
        {
          key: '123',
          title: '用户',
          type: 'input',
          rule: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
            {
              min: 2,
              max: 10,
              message: '用户名的长度在2～10个字',
              trigger: 'blur'
            }
          ]
        }
      ],
      // 用户添加
      formData: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 用户添加表单验证规则
      formRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '用户名的长度在2～10个字',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入用户密码', trigger: 'blur' },
          {
            min: 6,
            max: 18,
            message: '用户密码的长度在6～18个字',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getDataList()
  },
  methods: {
    async getDataList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败！')
      }
      this.datalist = res.data.users
      this.totle = res.data.totle
    },
    fetchData() {
      this.listLoading = true
      getList().then(response => {
        this.list = response.data.items
        this.listLoading = false
      })
    },
    // 监听 pagesize改变的事件
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getDataList()
    },
    // 监听 页码值 改变事件
    handleCurrentChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pagenum = newSize
      this.getDataList()
    },
    // 监听 添加用户对话框的关闭事件
    dialogClosed() {
      this.$refs.formRef.resetFields()
    },
    // 添加用户
    addData() {
      // 提交请求前，表单预验证
      this.$refs.formRef.validate(async valid => {
        // console.log(valid)
        // 表单预校验失败
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.formData)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功！')
        // 隐藏添加用户对话框
        this.dialogVisible = false
        this.getDataList()
      })
    },
    // 编辑用户信息
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败！')
      }
      this.formData = res.data
      this.dialogVisible = true
    },
    // 修改用户信息
    editData() {
      // 提交请求前，表单预验证
      this.$refs.formRef.validate(async valid => {
        // console.log(valid)
        // 表单预校验失败
        if (!valid) return
        const { data: res } = await this.$http.put(
          'users/' + this.formData.id,
          {
            email: this.formData.email,
            mobile: this.formData.mobile
          }
        )
        if (res.meta.status !== 200) {
          this.$message.error('更新用户信息失败！')
        }
        // 隐藏添加用户对话框
        this.dialogVisible = false
        this.$message.success('更新用户信息成功！')
        this.getDataList()
      })
    },
    // 删除
    async removeDataById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 点击确定 返回值为：confirm
      // 点击取消 返回值为： cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户失败！')
      this.$message.success('删除用户成功！')
      this.getDataList()
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
