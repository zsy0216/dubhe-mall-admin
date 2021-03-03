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
          <el-button type="primary" @click="dialogVisible = true, type = 'add'">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="datalist" border stripe>
        <!-- stripe: 斑马条纹
        border：边框-->
        <el-table-column type="index" label="#" />
        <el-table-column v-for="column in columns" :key="column.key" :prop="column.key" :label="column.title" />
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
    <el-dialog title="表单" :visible.sync="dialogVisible" width="50%" @close="dialogClosed">
      <!-- 内容主体 -->
      <el-form
        ref="formRef"
        :model="formData"
        :rules="formRules"
        label-width="100px"
      >
        <el-form-item v-for="form in forms" :key="form.key" :prop="form.key" :label="form.title">
          <el-input v-model="formData[form.key]" />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>

import request from '@/utils/request'

export default {
  name: 'GeneralPage',
  props: {
    columns: {
      type: Object,
      required: true
    },
    forms: {
      type: Object,
      required: true
    },
    formRules: {
      type: Object,
      required: true
    },
    apis: {}
  },
  data() {
    return {
      type: '',
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
      // 用户添加
      formData: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      }
    }
  },
  created() {
    this.getDataList()
  },
  methods: {
    // getList(params) {
    //   console.log('getList-API', params)
    //   return request({
    //     url: '/vue-admin-template/table/list',
    //     method: 'get',
    //     params
    //   })
    // },

    getDataList() {
      this.listLoading = true
      // this.getList(this.queryInfo)
      const params = this.queryInfo
      request({
        url: this.apis.list.url,
        method: this.apis.list.method,
        params
      }).then(response => {
        this.datalist = response.data.items
        // this.totle = res.data.totle
        this.listLoading = false
      })
    },
    submitForm() {
      console.log('Type', this.type)
      if (this.type === 'add') {
        this.addData()
      } else {
        this.editData()
      }
    },
    addData() {
      // 提交请求前，表单预验证
      this.$refs.formRef.validate(async valid => {
        // console.log(valid)
        // 表单预校验失败
        if (!valid) return
        const params = this.formData
        request({
          url: this.apis.create.url,
          method: this.apis.create.method,
          params
        }).then(response => {
          // if (res.meta.status !== 201) {
          //   this.$message.error('添加用户失败！')
          // }
          // this.$message.success('添加用户成功！')
          // 隐藏添加用户对话框
          this.dialogVisible = false
          this.getDataList()
        })
      })
    },
    // 编辑用户信息
    showEditDialog(id) {
      const params = this.formData
      request({
        url: this.apis.detail.url,
        method: this.apis.detail.method,
        params
      }).then(response => {
        // if (res.meta.status !== 200) {
        //   return this.$message.error('查询用户信息失败！')
        // }
        this.formData = response.data
        this.dialogVisible = true
        this.type = 'edit'
      })
    },
    // 修改用户信息
    editData() {
      // 提交请求前，表单预验证
      this.$refs.formRef.validate(async valid => {
        // console.log(valid)
        // 表单预校验失败
        if (!valid) return
        const params = this.formData
        request({
          url: this.apis.update.url,
          method: this.apis.update.method,
          params
        }).then(response => {
          // if (res.meta.status !== 200) {
          //   this.$message.error('更新用户信息失败！')
          // }
          // // 隐藏添加用户对话框
          // this.dialogVisible = false
          this.$message.success('更新用户信息成功！')
          this.getDataList()
        })
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
      const params = id
      request({
        url: this.apis.delete.url,
        method: this.apis.delete.method,
        params
      }).then(response => {
        // if (res.meta.status !== 200) return this.$message.error('删除用户失败！')
        this.$message.success('删除用户成功！')
        this.getDataList()
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
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
