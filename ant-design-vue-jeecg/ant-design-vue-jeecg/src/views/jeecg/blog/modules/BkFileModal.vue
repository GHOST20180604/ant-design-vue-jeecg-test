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

        <a-form-item label="文件名" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入文件名"></a-input>
        </a-form-item>
        <a-form-item label="文件后缀" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'suffix', validatorRules.suffix]" placeholder="请输入文件后缀"></a-input>
        </a-form-item>
        <a-form-item label="文件路径" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'url', validatorRules.url]" placeholder="请输入文件路径"></a-input>
        </a-form-item>
        <a-form-item label="文件ID(modb中保存ID)" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'fileId', validatorRules.fileId]" placeholder="请输入文件ID(modb中保存ID)"></a-input>
        </a-form-item>
        <a-form-item label="上传人人" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'createUser', validatorRules.createUser]" placeholder="请输入上传人人"></a-input>
        </a-form-item>
        <a-form-item label="上传时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择上传时间" v-decorator="[ 'createTime', validatorRules.createTime]" :trigger-change="true" style="width: 100%"/>
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
    name: "BkFileModal",
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
        name:{},
        suffix:{},
        url:{},
        fileId:{},
        createUser:{},
        createTime:{},
        },
        url: {
          add: "/blog/bkFile/add",
          edit: "/blog/bkFile/edit",
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
          this.form.setFieldsValue(pick(this.model,'name','suffix','url','fileId','createUser','createTime'))
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
        this.form.setFieldsValue(pick(row,'name','suffix','url','fileId','createUser','createTime'))
      },

      
    }
  }
</script>