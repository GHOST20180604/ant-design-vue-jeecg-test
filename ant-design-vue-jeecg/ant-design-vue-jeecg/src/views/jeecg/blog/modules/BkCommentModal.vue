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

        <a-form-item label="评论内容" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'content', validatorRules.content]" placeholder="请输入评论内容"></a-input>
        </a-form-item>
        <a-form-item label="文章ID" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'articleId', validatorRules.articleId]" placeholder="请输入文章ID"></a-input>
        </a-form-item>
        <a-form-item label="回复评论ID,如果是评论文章则为-1" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'pid', validatorRules.pid]" placeholder="请输入回复评论ID,如果是评论文章则为-1"></a-input>
        </a-form-item>
        <a-form-item label="回复人名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'pname', validatorRules.pname]" placeholder="请输入回复人名称"></a-input>
        </a-form-item>
        <a-form-item label="获赞数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'praiseNum', validatorRules.praiseNum]" placeholder="请输入获赞数" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="评论人留email" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'email', validatorRules.email]" placeholder="请输入评论人留email"></a-input>
        </a-form-item>
        <a-form-item label="评论人留链接路径" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'url', validatorRules.url]" placeholder="请输入评论人留链接路径"></a-input>
        </a-form-item>
        <a-form-item label="创建人" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'createTime', validatorRules.createTime]" placeholder="请输入创建人"></a-input>
        </a-form-item>
        <a-form-item label="创建时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择创建时间" v-decorator="[ 'createUser', validatorRules.createUser]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import JDate from '@/components/jeecg/JDate'  

  export default {
    name: "BkCommentModal",
    components: { 
      JDate,
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        validatorRules:{
        content:{},
        articleId:{},
        pid:{},
        pname:{},
        praiseNum:{},
        email:{},
        url:{},
        createTime:{},
        createUser:{},
        },
        url: {
          add: "/blog/bkComment/add",
          edit: "/blog/bkComment/edit",
        }
     
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'content','articleId','pid','pname','praiseNum','email','url','createTime','createUser'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
         
        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'content','articleId','pid','pname','praiseNum','email','url','createTime','createUser'))
      },

      
    }
  }
</script>