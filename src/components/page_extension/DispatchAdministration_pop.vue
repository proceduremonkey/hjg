<template>
  <el-dialog :title="title" :visible.sync="dialogVisible" class="dialog-size" size="small">
  	<el-form label-width="100px" ref="form" :model="form" :rules="rules">
  		
  		<!-- <el-form-item label="发件人" prop="from">
  			<remote-select type="member" v-model="form.from"></remote-select>
  		</el-form-item> -->

			<el-form-item label="收件人" prop="to" v-show="type !='confirm'">
				<remote-select type="member" v-model="form.to"></remote-select>
			</el-form-item>

			<el-form-item label="快递公司" prop="company" v-show="type !='confirm'">
				<el-autocomplete
          v-model="form.company"
          :fetch-suggestions="querySearch"
          placeholder="请输入快递公司名称"
        ></el-autocomplete>
			</el-form-item>

			<el-form-item label="快递单号" prop="number" v-show="type !='confirm'">
				<el-input v-model="form.number" placeholder="请填写快递单号"></el-input>
			</el-form-item>

      <el-form-item label="收文日期" prop="receipt_date" v-show="type != 'add'" class="is-required">
        <el-date-picker v-model="form.receipt_date" type="date" placeholder="请选择收件日期"></el-date-picker>
      </el-form-item>

			<el-form-item label="备注" prop="description">
        <span v-if="type != 'add'">{{ form.description ? form.description : '暂无备注信息' }}</span>
        <el-input v-else type="textarea" v-model="form.description" placeholder="请填写备注"></el-input>
      </el-form-item>

      <el-form-item label="发文日期" prop="mail_date" format="yyyy-MM-dd" v-show="type !='confirm'">
        <el-date-picker v-model="form.mail_date" type="date" placeholder="请选择发文日期"></el-date-picker>
      </el-form-item>
      <el-form-item label="文件清单" prop="projects">
        <span v-if="type != 'add' && form.projects.length == 0">暂无文件清单</span>
				<express-list v-else v-model="form.projects" :typeMessage="type"></express-list>
			</el-form-item>

			<el-form-item style="margin-bottom: 0px;">
				<el-button type="primary" @click="add" v-if="type === 'add'" :disabled="btn_disabled">添加</el-button>
				<el-button type="primary" @click="confirmFunc" v-if="type === 'confirm'" :disabled="btn_disabled">确认收到上述文件</el-button>
			</el-form-item>

  	</el-form>
  </el-dialog>
</template>

<script>
import AxiosMixins from '@/mixins/axios-mixins'
import PopMixins from '@/mixins/pop-mixins'
import RemoteSelect from '@/components/form/RemoteSelect'
import ExpressList from '@/components/form/ExpressList'
export default {
  name: 'DisatchAdministrationPop',
  mixins: [ AxiosMixins, PopMixins ],
  props:['confirm'],
  data () {
		return {
      dateDisable:false,
      form: {
        to: '',
        company: '',
        number: '',
        mail_date: '',
        receipt_date: '',
        description: '',
        projects: [],
      },
      rules: {
        to: {type: 'number', required: true, message: '收件人必填', trigger: 'change'},
        company: {required: true, message: '快递公司必填', trigger: 'change'},
        number: [
          { required: true,  message: '快递单号必填', trigger: 'blur' },
          {pattern: /^[0-9a-zA-Z]*$/, message : '快递单号只能是字母与数字', trigger: 'blur' }
        ],
        mail_date: { type:'date',required: true, message: '发文日期必填', trigger: 'change'},
      },
      queryData: [
        { "value": '顺丰'},
        { "value": '邮政EMS'},
        { "value": '圆通快递'},
        { "value": '申通快递'},
        { "value": '韵达快运'},
        { "value": '中通快递'},
        { "value": '汇通快递'},
        { "value": '天天快递'},
        { "value": '宅急送'},
      ],
		}
  },
  computed: {
    dateDisabled () {
      return this.confirm === 'confirm' ? this.dateDisable = true :this.dateDisable = false;
    }
  },
  methods: {
    querySearch(queryString, cb) {
      var queryData = this.queryData;
      var results = queryString ? queryData.filter(this.createFilter(queryString)) : queryData;
      // 调用 callback 返回建议列表的数据
      cb(results);
    },
    createFilter(queryString) {
      return _ => {
        return (_.value.indexOf(queryString.toLowerCase()) === 0);
      };
    },
    keepSameTime (val) {
      // console.log(typeof(new Date(val)));
      this.form.receipt_date = val;
      console.log(val)
    },
    confirmFunc () {
      if( !this.form.receipt_date ) { return this.$message({type: 'warning', message: '请选择收文日期'});  }

      const url = `/api/expresses/${this.id}`;
      const data = {receipt_date: this.$tool.getDate(this.form.receipt_date)};
      const success = _=>{
        this.$message({message: '确认收文成功', type: 'success'});
        this.dialogVisible = false;
        this.$emit('refresh', 'edit');
      }
      const complete = _=>{
        this.btn_disabled = false;
      }

      this.btn_disabled = true;
      this.$axiosPut({url, data, success, complete});
      
    },
    setForm (data) {
      for(let key in this.form) {
        const d = data[key];
        if(d == undefined) continue;
        if(key == 'mail_date') {
          this.form[key] = new Date(d);
        }else {
          this.form[key] = d;
        }
      }
    },
    submitForm () {
      return this.$tool.shallowCopy(this.form, {date: true});
    }
  },
  components: {  ExpressList, RemoteSelect  },
  URL: '/api/expresses',
  REMINDER_TEXT: '发文信息',
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>