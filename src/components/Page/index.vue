<template>
  <div class="app-container">
    <el-card class="filter-container" shadow="never">
      <div>
        <i class="el-icon-search" />
        <span>筛选搜索</span>
        <el-button
          style="float: right"
          type="primary"
          size="small"
          @click="getDataList()"
        >
          查询结果
        </el-button>
        <el-button
          style="float: right;margin-right: 15px"
          size="small"
          @click="handleResetSearch()"
        >
          重置
        </el-button>
      </div>
      <div style="margin-top: 15px">
        <el-form :inline="true" :model="queryInfo" size="small" label-width="140px">
          <el-form-item v-for="searchItem in searchList" :key="searchItem.key" :label="searchItem.label">
            <el-input v-if="searchItem.type==='input'" v-model="queryInfo.query[searchItem.key]" style="width: 185px" :placeholder="searchItem.placeholder" />
            <el-cascader
              v-if="searchItem.type==='cascader'"
              v-model="queryInfo.query[searchItem.key]"
              clearable
              :options="searchItem.options"
            />
            <el-select v-if="searchItem.type==='select'" v-model="queryInfo.query[searchItem.key]" :placeholder="searchItem.placeholder" clearable>
              <el-option
                v-for="item in searchItem.options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
        </el-form>
      </div>
    </el-card>
    <el-card class="operate-container" shadow="never">
      <i class="el-icon-tickets" />
      <span>数据列表</span>
      <el-button
        class="btn-add"
        size="mini"
        @click="dialogVisible = true, type = 'add'"
      >
        添加
      </el-button>
    </el-card>
    <div class="table-container">
      <el-table
        ref="tableList"
        v-loading="listLoading"
        :data="datalist"
        style="width: 100%"
        border
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="60" align="center" />
        <el-table-column v-for="column in columns" :key="column.key" :prop="column.key" :label="column.label" align="center">
          <template v-if="column.type==='texts'" slot-scope="scope">
            <p v-for="text in column.texts" :key="text.key">{{ text.pretext }}{{ scope.row[text.key] }}</p>
          </template>
          <template v-if="column.type==='img'" slot-scope="scope">
            <img style="height: 80px" :src="scope.row[column.key]">
          </template>
          <template v-if="column.type==='switch'" slot-scope="scope">
            <p v-for="switchItem in column.switches" :key="switchItem.key">{{ switchItem.pretext }}
              <el-switch
                v-model="scope.row[switchItem.key]"
                :active-value="1"
                :inactive-value="0"
                @change="handleStatusChange(switchItem, scope.row)"
              />
            </p>
          </template>
          <template v-if="column.type==='button-primary'" slot-scope="scope">
            <el-button type="primary" :icon="column.icon" circle @click="showDialog(scope.row.id, 'edit')" />
          </template>
        </el-table-column>
        <el-table-column label="操作" width="160" align="center">
          <template slot-scope="scope">
            <p>
              <el-button
                size="mini"
                @click="showDialog(scope.row.id, 'show')"
              >查看
              </el-button>
              <el-button
                size="mini"
                @click="showDialog(scope.row.id, 'edit')"
              >编辑
              </el-button>
            </p>
            <p>
              <el-button
                size="mini"
                type="danger"
                @click="removeDataById(scope.row.id)"
              >删除
              </el-button>
            </p>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="pagination-container">
      <el-pagination
        background
        layout="total, sizes,prev, pager, next,jumper"
        :page-size="queryInfo.pageSize"
        :page-sizes="[5,10,15]"
        :current-page.sync="queryInfo.pageNum"
        :total="total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>
    <!-- 对话框 -->
    <el-dialog title="表单" :visible.sync="dialogVisible" width="50%" @close="dialogClosed">
      <!-- 内容主体 -->
      <el-form
        ref="formRef"
        :model="formData"
        :rules="formRules"
        :disabled="type === 'show'"
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
    searchList: [],
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
        query: {},
        // 当前页数
        pageNum: 1,
        // 每页显示多少数据
        pageSize: 5
      },
      datalist: [],
      listLoading: false,
      total: 0,
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
        // this.total = res.data.total
        this.listLoading = false
      })
    },
    handleResetSearch() {
      this.queryInfo['query'] = {}
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
    showDialog(id, str) {
      const params = id
      request({
        url: this.apis.detail.url,
        method: this.apis.detail.method,
        params
      }).then(response => {
        // if (res.meta.status !== 200) {
        //   return this.$message.error('查询用户信息失败！')
        // }
        this.type = str
        this.formData = response.data
        this.dialogVisible = true
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
          this.$message.success('更新用户信息成功！')
          // 隐藏添加用户对话框
          this.dialogVisible = false
          this.formData = []
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
    handleStatusChange(switchItem, row) {
      const params = new URLSearchParams()
      params.append(switchItem.key, switchItem.key)
      params.append('changeItem', row)
      request({
        url: switchItem.url,
        method: switchItem.method,
        params
      }).then(response => {
        this.$message({
          message: '修改成功',
          type: 'success',
          duration: 1000
        })
      })
    },
    // 监听 pagesize改变的事件
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pageSize = newSize
      this.getDataList()
    },
    // 监听 页码值 改变事件
    handleCurrentChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pageNum = newSize
      this.getDataList()
    },
    // 监听 添加用户对话框的关闭事件
    dialogClosed() {
      this.$refs.formRef.resetFields()
    },
    handleSelectionChange(val) {
      this.multipleSelection = val
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
