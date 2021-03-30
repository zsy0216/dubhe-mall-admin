<template>
  <div class="app-container">
    <GeneralPage
      :columns="columns"
      :form-rules="formRules"
      :forms="forms"
      :apis="apis"
      :search-list="searchList"
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
      searchList: [
        {
          key: 'keyword',
          label: '输入搜索: ',
          type: 'input',
          placeholder: '商品名称'
        },
        {
          key: 'productSn',
          label: '商品货号: ',
          type: 'input',
          placeholder: '商品名称'
        },
        {
          key: 'ProductCate',
          label: '商品分类: ',
          type: 'cascader',
          model: this.selectProductCateValue,
          options: this.productCateOptions,
          placeholder: '商品名称'
        },
        {
          key: 'brandId',
          label: '商品品牌: ',
          type: 'select',
          options: this.brandOptions,
          placeholder: '请选择品牌'
        },
        {
          key: 'publishStatus',
          label: '上架状态: ',
          type: 'select',
          options: this.publishStatusOptions,
          placeholder: '全部'
        },
        {
          key: 'verifyStatus',
          label: '审核状态: ',
          type: 'select',
          options: this.verifyStatusOptions,
          placeholder: '全部'
        }
      ],
      columns: [
        {
          key: 'id',
          label: '编号',
          type: 'texts',
          texts: [
            {
              key: 'id',
              pretext: '编号'
            }
          ]
        },
        {
          key: 'pic',
          label: '商品图片',
          type: 'img'
        },
        {
          key: 'name',
          label: '商品名称',
          type: 'texts',
          texts: [
            {
              key: 'name',
              pretext: ''
            },
            {
              key: 'brandName',
              pretext: '品牌：'
            }
          ]
        },
        {
          key: 'price',
          label: '价格/货号',
          type: 'texts',
          texts: [
            {
              key: 'price',
              pretext: '价格：￥'
            },
            {
              key: 'productSn',
              pretext: '货号：'
            }
          ]
        },
        {
          key: 'lab',
          label: '标签',
          type: 'switch',
          switches: [
            {
              pretext: '上架：',
              key: 'publishStatus',
              method: 'post',
              url: '/vue-admin-template/table/list'
            },
            {
              pretext: '新品：',
              key: 'newStatus',
              method: 'post',
              url: '/vue-admin-template/table/list'
            },
            {
              pretext: '推荐：',
              key: 'recommandStatus',
              method: 'post',
              url: '/vue-admin-template/table/list'
            }
          ]
        },
        {
          key: 'sort',
          label: '排序',
          type: 'texts',
          texts: [
            {
              key: 'sort',
              pretext: ''
            }
          ]
        },
        {
          key: 'SKUButton',
          label: 'SKU库存',
          type: 'button-primary',
          icon: 'el-icon-edit'
        },
        {
          key: 'sale',
          label: '销量',
          type: 'texts',
          texts: [
            {
              key: 'sale',
              pretext: ''
            }
          ]
        }
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

