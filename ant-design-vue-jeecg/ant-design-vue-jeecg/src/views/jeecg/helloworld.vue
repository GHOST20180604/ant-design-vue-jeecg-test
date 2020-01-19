<template>
  <div>
    <vue-ueditor-wrap v-model="msg" :config="myConfig"  ></vue-ueditor-wrap>
  </div>
</template>

<script>
  import VueUeditorWrap from '@/components/vue-ueditor-wrap.vue';

  export default {
    components: {
      VueUeditorWrap
    },
    data () {
      return {
        msg: '',
        myConfig: {
          autoHeightEnabled: false, // 编辑器不自动被内容撑高
          initialFrameHeight: 200, // 初始容器高度
          initialFrameWidth: '100%',   // 初始容器宽度
          serverUrl: 'http://35.201.165.105:8000/controller.php' // 上传文件接口（这个地址是我为了方便各位体验文件上传功能搭建的临时接口，请勿在生产环境使用！！！）
        }
      };
    },
    methods: {

    }
  };
</script>

