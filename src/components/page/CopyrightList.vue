<template>
  <div class="main">
    <strainer v-model="filter" @refresh="refresh"></strainer>
    <table-component :tableOption="tableOption" :data="tableData" @refreshTableData="refreshTableData" ref="table"></table-component>
    
      <common-detail
        :title="currentRow.title"
        :visible.sync="shrinkVisible" 
        type="copyright" 
        :id="currentRow.id"
        @editSuccess="refresh">
      </common-detail>

  </div>
</template>

<script>
import AxiosMixins from '@/mixins/axios-mixins'
import TableComponent from '@/components/common/TableComponent'
import Strainer from '@/components/page_extension/CopyrightList_strainer'
import AppShrink from '@/components/common/AppShrink'
import CommonDetail from '@/components/page_extension/Common_detail'

const URL = '/api/copyrights';

export default {
  name: 'copyrightList',
  mixins: [ AxiosMixins ],
  data () {
    return {
      currentRow: '',
      shrinkVisible: false,
      tableOption: {
        'name': 'copyrightList',
        'url': URL,
        'height': 'default',
        'highlightCurrentRow': true, 
        'rowClick': this.handleRowClick,
        'is_filter': true,
        'import_type': 'copyright',
        'upload_type': 'copyright',
        'header_btn': [
          { type: 'add', click: this.add },
          { type: 'delete' },
          { type: 'export' },
          { type: 'import' },
          { type: 'batch_upload' },
          { type: 'control', label: '字段' },
        ],
        'columns': [
          { type: 'selection' },
          { type: 'text', label: '案号', prop: 'serial', width: '200', sortable: true },
          { type: 'text', label: '事务所案号', prop: 'agency_serial', width: '200', sortable: true },
          { type: 'text', label: '版权类型', prop: 'type', is_import: true, render_simple: 'name', width: '160', sortable: true },
          { type: 'text', label: '标题', prop: 'title', is_import: true, width: '160'},
          { type: 'text', label: '摘要', prop: 'abstract', width: '280' },
          { type: 'text', label: '完成时间', prop: 'create_time', width: '173', sortable: true },
          { type: 'text', label: '申请日', prop: 'apd', sortable: true, is_import: true, width: '173', sortable: true },
          { type: 'text', label: '申请号', prop: 'apn', width: '263', sortable: true },
          { type: 'text', label: '公告日', prop: 'issue_date', is_import: true, width: '183', sortable: true },
          { type: 'text', label: '公告号', prop: 'issue_number', is_import: true, width: '263', sortable: true },
          { type: 'text', label: '代理人', prop: 'agent', render_simple: 'name', is_import: true, width: '140' },
          { type: 'text', label: '代理机构名称', prop: 'agency', render_simple: 'name', width: '143' },
          { type: 'text', label: '代理机构案号', prop: 'agency_serial', width: '263', sortable: true },
          { type: 'text', label: '备注', prop: 'remark', is_import: true, width: '285' },
          { type: 'array', label: '申请人', prop: 'applicants', is_import: true, render: _=>_.map(_=>_.name), width: '220' },
          { type: 'text', label: '提案人', prop: 'proposer', is_import: true, render_simple: 'name', width: '145' },
          { type: 'array', label: '标签', prop: 'tags', is_import: true, width: '200' },
          { type: 'array', label: '产品名称', prop: 'products', is_import: true, render: _=>_.map(_=>_.name), width: '150' },
          // {
          //   type: 'action',
          //   width: '150',
          //   btns: [
          //     // { type: 'detail', click: this.detail },
          //     { type: 'delete', click: this.deleteSingle },
          //   ], 
          // },
        ] 
      },
      tableData: [],
      filter: {},
    };
  },
  methods: {
    add () {
      this.$router.push('/copyright/add');
    },
    refreshTableData (option) {
      const url = URL;
      const data = Object.assign({}, option, this.filter);
      const success = d=>{
        if(data['format'] == 'excel') {
          window.location.href = d.copyrights.downloadUrl;
        }else {
          this.tableData = d.copyrights;  
        }      
      };
      this.axiosGet({url, data, success});
    },
    refresh () {
      this.$refs.table.refresh();
    },
    deleteSingle ({ title, id }) {
      this.$confirm(`删除后不可恢复，确认删除‘${title}’吗？`)
        .then(()=>{
          const url=`${URL}/${id}`;
          const success = _=>{this.$refs.table.update()};
          this.axiosDelete({url, success});    
        })
        .catch(()=>{});
    },
    detail ({id}) {
      const path = `/copyright/list/detail/${id}`; 
      this.$router.push( path );
    },
    handleRowClick (row) {
      this.currentRow = row;
      if(!this.shrinkVisible) this.shrinkVisible = true;
    },
    close () {
      this.$refs.table.setCurrentRow();
    }
  },
  mounted () {
    this.$refs.table.refresh();
  },
  components: { 
    TableComponent, 
    Strainer, 
    AppShrink, 
    CommonDetail 
  }
}
</script>
<style>

</style>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">



</style>
