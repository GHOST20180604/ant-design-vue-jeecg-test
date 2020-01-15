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

        <a-form-item :wrapperCol="{xs: { span: 24 },sm: { span: 24 }}">
          <a-input v-decorator="[ 'title', validatorRules.title]" placeholder="请输入标题"></a-input>
        </a-form-item>
        <!--<a-form-item label="内容" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
        <!--<a-textarea v-decorator="['content']" rows="4" placeholder="请输入内容"/>-->
        <!--</a-form-item>-->
        <vue-ueditor-wrap ref="ueditor" v-decorator="['content',validatorRules.content]" :config="myConfig"></vue-ueditor-wrap>

        <a-form-item label="文章分类" style="margin-top: 30px;" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-select v-decorator="[ 'categoryId', validatorRules.categoryId]" optionFilterProp="children" style="width: 100%">
            <a-select-option v-for="(catagory,catagoryIndex) in catagoryList" :key="catagoryIndex.toString()" :value="catagory.id">
              {{ catagory.name }}
            </a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="可否评论" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-radio-group v-decorator="[ 'canComment',validatorRules.canComment]">
            <a-radio :value="1">是</a-radio>
            <a-radio :value="0">否</a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item label="是否公开" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-radio-group v-decorator="[ 'isPublic', validatorRules.isPublic]">
            <a-radio :value="1">是</a-radio>
            <a-radio :value="0">否</a-radio>
          </a-radio-group>
        </a-form-item>


      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import JDate from '@/components/jeecg/JDate'
  import { queryAllCategory } from '@/api/api'
  import VueUeditorWrap from '@/components/vue-ueditor-wrap.vue'

  export default {
    name: 'BkArticleModal',
    components: {
      JDate,
      VueUeditorWrap
    },
    data() {
      return {
        form: this.$form.createForm(this),
        title: '操作',
        catagoryList: [],
        content: '',
        myConfig: {
          autoHeightEnabled: false, // 编辑器不自动被内容撑高
          initialFrameHeight: 200, // 初始容器高度
          initialFrameWidth: '100%',   // 初始容器宽度
          serverUrl: 'http://35.201.165.105:8000/controller.php' // 上传文件接口（这个地址是我为了方便各位体验文件上传功能搭建的临时接口，请勿在生产环境使用！！！）
        },
        width: 800,
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 3 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 18 }
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
              required: true, message: '请选择评论!'
            }]
          },
          isPublic: {
            rules: [{
              required: true, message: '请选择公开!'
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
          this.form.setFieldsValue(pick(this.model, 'title', 'content', 'status', 'praiseNum', 'canComment', 'isPublic'))
        })
      },
      close() {
        this.$emit('close')
        this.visible = false
      },
      handleOk() {
        const that = this
        //将富文本内容拿出来
        this.content = this.$refs.ueditor.editor.getContent()
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
            if (this.content) {
              values.content = this.content
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