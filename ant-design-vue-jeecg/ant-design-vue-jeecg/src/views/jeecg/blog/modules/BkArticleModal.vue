<template>
  <div v-if="visible">
    <a-spin :spinning="confirmLoading">
      <h2 style="margin-left: 10px;"> {{title}}</h2>
      <a-form :form="form">
        <a-form-item :wrapperCol="{xs: { span: 24 },sm: { span: 24 }}">
          <a-input v-decorator="[ 'title', validatorRules.title]" placeholder="请输入标题"></a-input>
        </a-form-item>
        <!--<a-form-item label="内容" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
        <!--<a-textarea v-decorator="['content']" rows="4" placeholder="请输入内容"/>-->
        <!--</a-form-item>-->
        <vue-ueditor-wrap ref="ueditor" v-decorator="['content',validatorRules.content]"
                          v-model="content"  :config="myConfig"></vue-ueditor-wrap>

        <a-form-item label="文章分类" style="margin-top: 30px;" :labelCol="labelCol" :wrapperCol="wrapperCol">

          <a-select v-decorator="[ 'categoryId', validatorRules.categoryId]" optionFilterProp="children"
                    style="width: 100%">
            <a-select-option value="">请选择</a-select-option>
            <a-select-option v-for="(catagory,catagoryIndex) in catagoryList" :key="catagoryIndex"
                             :value="catagory.id">
              {{ catagory.name }}
            </a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="可否评论" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-radio-group v-decorator="[ 'canComment',validatorRules.canComment]">
            <a-radio value="1">是</a-radio>
            <a-radio value="0">否</a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item label="是否公开" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-radio-group v-decorator="[ 'isPublic', validatorRules.isPublic]">
            <a-radio value="1">是</a-radio>
            <a-radio value="0">否</a-radio>
          </a-radio-group>
        </a-form-item>
      </a-form>
      <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
          <a-button type="primary" style="margin-left: 8px" @click="handleOk('1')" icon="save">发布博客</a-button>
          <a-button type="primary" style="margin-left: 8px" @click="handleOk('0')" icon="save">保存草稿</a-button>
          <a-button type="primary" @click="handleCancel" icon="reload" style="margin-left: 8px">取消</a-button>
          <a-button type="primary" @click="getSomeDate" icon="reload" style="margin-left: 8px">获取当前页面数据</a-button>
        </span>
    </a-spin>
  </div>
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
          sm: { span: 2 }
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
        debugger
        this.form.resetFields()
        this.findAllCategory()
        this.model = Object.assign({}, record)
        if (!this.model.isPublic) {
          this.model.isPublic = '1'
          this.model.canComment = '1'
          this.content = ''
        }else{
          this.content = this.model.content;
        }
        this.visible = true

        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'title', 'content', 'status', 'praiseNum', 'canComment', 'isPublic','categoryId'))
        })
      },
      close() {
        this.$emit('close')
        this.visible = false
        // this.$parent.showAddOrUpdate = false
        this.$emit('showList')
      },
      handleOk(status) {
        const that = this
        //将富文本内容拿出来
        debugger
        // this.content = this.$refs.ueditor.editor.getContent()
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
            values.status = status
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
      getSomeDate() {
        // debugger
        // let aaa = this.model
        // var bbb = this.$parent.showAddOrUpdate ;
        // var ccc = this.$parent ;
        // console.log(
        //   aaa
        // )
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