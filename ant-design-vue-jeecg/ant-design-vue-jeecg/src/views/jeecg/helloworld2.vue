<template>
  <div>
    hello world!------ + {{this.msg}}
  </div>
</template>
<script>
  export default {
    data () {
      return {
        description: '表单页用于向用户收集或验证信息，基础表单常见于数据项较少的表单场景。',
        msg: '',
      }
    },
    methods: {

    },
    mounted(){

      // console.log(this.$route.params);  /*获取动态路由传值*/

      this.msg=this.$route.params.aid;
    },
  }
</script>