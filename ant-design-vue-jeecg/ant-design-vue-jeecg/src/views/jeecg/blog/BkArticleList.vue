<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="标题">
              <j-input placeholder="请输入标题查询" v-model="queryParam.title"></j-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 5px">重置</a-button>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('文章表')"  style="margin-left: 5px;">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel"  style="margin-left: 5px;">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{fixed:true,selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"

        @change="handleTableChange">

        <!--<template slot="htmlSlot" slot-scope="text">-->
          <!--<div v-html="text"></div>-->
        <!--</template>-->
        <!--<template slot="imgSlot" slot-scope="text">-->
          <!--<span v-if="!text" style="font-size: 12px;font-style: italic;">无此图片</span>-->
          <!--<img v-else :src="getImgView(text)" height="25px" alt="图片不存在"-->
               <!--style="max-width:80px;font-size: 12px;font-style: italic;"/>-->
        <!--</template>-->
        <!--<template slot="fileSlot" slot-scope="text">-->
          <!--<span v-if="!text" style="font-size: 12px;font-style: italic;">无此文件</span>-->
          <!--<a-button-->
            <!--v-else-->
            <!--:ghost="true"-->
            <!--type="primary"-->
            <!--icon="download"-->
            <!--size="small"-->
            <!--@click="uploadFile(text)">-->
            <!--下载-->
          <!--</a-button>-->
        <!--</template>-->

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <bkArticle-modal ref="modalForm" @ok="modalFormOk"></bkArticle-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import BkArticleModal from './modules/BkArticleModal'
  import JInput from '@/components/jeecg/JInput'

  export default {
    name: 'BkArticleList',
    mixins: [JeecgListMixin],
    components: {
      BkArticleModal,
      JInput
    },
    data() {
      return {
        description: '文章表管理页面',
        queryParam: {},
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: 'center',
            customRender: function(t, r, index) {
              return parseInt(index) + 1
            }
          },
          {
            title: '标题',
            align: 'center',
            width: 120,
            dataIndex: 'title',
            customRender: function(text, r, index) {
              return !text ? '' : (text.length > 5 ? text.substr(0, 5) + '...' : text)
            }
          },
          {
            title: '分类名称',
            align: 'center',
            width: 120,
            dataIndex: 'categoryName',
            customRender: function(text, r, index) {
              return !text ? '' : (text.length > 5 ? text.substr(0, 5) + '...' : text)
            }
          },

          {
            title: '状态',
            align: 'center',
            width: 60,
            dataIndex: 'status',
            customRender: function(text, r, index) {
              if (text === '0') {
                return '草稿'
              } else if (text === '1') {
                return '发布'
              } else {
                return text
              }
            }
          },
          {
            title: '获赞',
            align: 'center',
            width: 50,
            dataIndex: 'praiseNum'
          },
          {
            title: '可评论',
            align: 'center',
            width: 70,
            dataIndex: 'canComment',
            customRender: function(text, r, index) {
              if (text === '0') {
                return '否'
              } else if (text === '1') {
                return '是'
              } else {
                return text
              }
            }
          },
          {
            title: '公开',
            align: 'center',
            width: 50,
            dataIndex: 'isPublic',
            customRender: function(text, r, index) {
              if (text === '0') {
                return '否'
              } else if (text === '1') {
                return '是'
              } else {
                return text
              }
            }
          },
          {
            title: '创建人',
            align: 'center',
            width: 120,
            dataIndex: 'createBy'
          },
          {
            title: '创建时间',
            align: 'center',
            width: 100,
            dataIndex: 'createTime',
            customRender: function(text) {
              return !text ? '' : (text.length > 10 ? text.substr(0, 10) : text)
            }
          },
          {
            title: '变更人',
            align: 'center',
            width: 120,
            dataIndex: 'updateBy'
          },
          {
            title: '变更时间',
            align: 'center',
            width: 100,
            dataIndex: 'updateTime',
            customRender: function(text) {
              return !text ? '' : (text.length > 10 ? text.substr(0, 10) : text)
            }
          },
          {
            title: '操作',
            dataIndex: 'action',
            width: 120,
            align: 'center',
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: '/blog/bkArticle/list',
          delete: '/blog/bkArticle/delete',
          deleteBatch: '/blog/bkArticle/deleteBatch',
          exportXlsUrl: '/blog/bkArticle/exportXls',
          importExcelUrl: 'blog/bkArticle/importExcel'
        },
        dictOptions: {}
      }
    },
    computed: {
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      }
    },
    methods: {
      initDictConfig() {
      },
      handleAdd: function() {
        this.$refs.modalForm.add()
        this.$refs.modalForm.title = '新增'
        this.$refs.modalForm.disableSubmit = false
      }
    }
  }
</script>
<!--<style scoped>-->
<!--@import '~@assets/less/common.less'-->
<!--</style>-->