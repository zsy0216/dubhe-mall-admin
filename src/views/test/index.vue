<template>
  <div class="app-container">
    <GeneralPage
      :columns="columns"
      :form-rules="formRules"
      :forms="forms"
      :apis="apis"
      @getList="fetchData"
    />

  </div>
</template>

<script>
import GeneralPage from '@/components/Page/index'
export default {
  components: {
    GeneralPage
  },
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
      apis: {
        list: {
          method: 'get',
          // url: `${constants.remoteService ? 'api/' : ''}parameter/binTable/get/page`,
          url: '/vue-admin-template/table/list'
        },
        update: {
          method: 'post',
          url: '/vue-admin-template/table/list'
        },
        detail: {
          method: 'post',
          url: '/vue-admin-template/table/list'
        },
        create: {
          method: 'post',
          url: '/vue-admin-template/table/list'
        },
        delete: {
          method: 'post',
          url: '/vue-admin-template/table/list'
        }
      },
      columns: [
        { key: 'title', title: 'title' },
        { key: 'author', title: '作者' },
        { key: 'pageviews', title: 'pageviews' }
      ],
      forms: [
        {
          key: 'title',
          title: '标题',
          type: 'input'
        },
        {
          key: 'author',
          title: '作者',
          type: 'input'
        },
        {
          key: 'pageviews',
          title: 'pageviews',
          type: 'input'
        }
      ],
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
        title: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      }
    }
  }

}
</script>

