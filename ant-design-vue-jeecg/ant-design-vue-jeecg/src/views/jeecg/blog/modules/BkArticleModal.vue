<template>
  <a-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <!--<a-form-item label="用户ID" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
        <!--<a-input v-decorator="[ 'userId', validatorRules.userId]" placeholder="请输入用户ID"></a-input>-->
        <!--</a-form-item>-->
        <a-form-item label="标题" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'title', validatorRules.title]" placeholder="请输入标题"></a-input>
        </a-form-item>
        <a-form-item label="内容" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['content']" rows="4" placeholder="请输入内容"/>
        </a-form-item>
        <a-form-item label="分类" :labelCol="labelCol" :wrapperCol="wrapperCol">

          <a-select v-decorator="[ 'categoryId', validatorRules.categoryId]"
                    optionFilterProp="children"
                    style="width: 100%">
            <a-select-option v-for="(catagory,catagoryIndex) in catagoryList" :key="catagoryIndex.toString()"
                             :value="catagory.id">
              {{ catagory.name }}
            </a-select-option>
          </a-select>
        </a-form-item>
        <!--<a-form-item label="状态;0:草稿;1:已发布;2:已删除;" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
        <!--<a-input v-decorator="[ 'status', validatorRules.status]" placeholder="请输入状态;0:草稿;1:已发布;2:已删除;"></a-input>-->
        <!--</a-form-item>-->
        <a-form-item label="获赞次数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'praiseNum', validatorRules.praiseNum]" placeholder="请输入获赞次数"
                          style="width: 100%"/>
        </a-form-item>
        <a-form-item label="是否可评论;0否;1是" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'canComment', validatorRules.canComment]" placeholder="请输入是否可评论;0否;1是"></a-input>
        </a-form-item>
        <a-form-item label="是否公开;0否;1是" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'isPublic', validatorRules.isPublic]" placeholder="请输入是否公开;0否;1是"></a-input>
        </a-form-item>
        <!--<a-form-item label="创建人" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
        <!--<a-input v-decorator="[ 'createBy', validatorRules.createBy]" placeholder="请输入创建人"></a-input>-->
        <!--</a-form-item>-->
        <!--<a-form-item label="创建时间" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
        <!--<j-date placeholder="请选择创建时间" v-decorator="[ 'createTime', validatorRules.createTime]" :trigger-change="true" style="width: 100%"/>-->
        <!--</a-form-item>-->
        <!--<a-form-item label="变更人" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
        <!--<a-input v-decorator="[ 'updateBy', validatorRules.updateBy]" placeholder="请输入变更人"></a-input>-->
        <!--</a-form-item>-->
        <!--<a-form-item label="变更时间" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
        <!--<j-date placeholder="请选择变更时间" v-decorator="[ 'updateTime', validatorRules.updateTime]" :trigger-change="true" style="width: 100%"/>-->
        <!--</a-form-item>-->
        <!--<a-form-item label="分类ID" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
        <!--<a-input v-decorator="[ 'categoryId', validatorRules.categoryId]" placeholder="请输入分类ID"></a-input>-->
        <!--</a-form-item>-->

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import JDate from '@/components/jeecg/JDate'
  import { queryAllCategory } from '@/api/api'

  export default {
    name: 'BkArticleModal',
    components: {
      JDate
    },
    data() {
      return {
        form: this.$form.createForm(this),
        title: '操作',
        catagoryList: [],
        width: 800,
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 }
        },

        confirmLoading: false,
        validatorRules: {
          userId: {},
          title: {
            rules: [{
              required: true, message: '请填写标题!'
            }]
          },
          content: {},
          status: {},
          praiseNum: {},
          canComment: {
            rules: [{
              required: true, message: '是否可评论?'
            }]
          },
          isPublic: {
            rules: [{
              required: true, message: '请选择分类!'
            }]
          },
          createBy: {},
          createTime: {},
          updateBy: {},
          updateTime: {},
          categoryId: {
            rules: [{
              required: true, message: '请选择分类!'
            }]
          }
        },
        url: {
          add: '/blog/bkArticle/add',
          edit: '/blog/bkArticle/edit'
        }

      }
    },
    created() {
    },
    methods: {
      add() {
        this.edit({})
      },
      edit(record) {
        this.form.resetFields()
        this.findAllCategory()
        this.model = Object.assign({}, record)
        this.visible = true
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'userId', 'title', 'content', 'status', 'praiseNum', 'canComment', 'isPublic', 'createBy', 'createTime', 'updateBy', 'updateTime', 'categoryId'))
        })
      },
      close() {
        this.$emit('close')
        this.visible = false
      },
      handleOk() {
        const that = this
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true
            let httpurl = ''
            let method = ''
            if (!this.model.id) {
              httpurl += this.url.add
              method = 'post'
            } else {
              httpurl += this.url.edit
              method = 'put'
            }
            debugger
            let formData = Object.assign(this.model, values)
            console.log('表单提交数据', formData)
            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$message.warning(res.message)
              }
            }).finally(() => {
              that.confirmLoading = false
              that.close()
            })
          }

        })
      },
      handleCancel() {
        this.close()
      },
      popupCallback(row) {
        this.form.setFieldsValue(pick(row, 'userId', 'title', 'content', 'status', 'praiseNum', 'canComment', 'isPublic', 'createBy', 'createTime', 'updateBy', 'updateTime', 'categoryId'))
      },
      findAllCategory() {
        queryAllCategory().then((res) => {
          if (res.success) {
            this.catagoryList = res.result
          } else {
            console.log(res.message)
          }
        })
      }

    }
  }
</script>