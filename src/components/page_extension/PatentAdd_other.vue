<template>
  <app-collapse col-title="其他信息及附件">
      <el-form label-width="120px">
        <el-form-item label="状态" v-if="type == 'edit'">
          {{ progress_name }}
        </el-form-item>
        <el-form-item label="说明书字数">
          <el-input v-model="form.words" placeholder="请填写说明书字数"></el-input>
        </el-form-item>
        <el-form-item label="备注">
          <el-input v-model="form.remark" type="textarea" placeholder="请填写备注信息"></el-input>
        </el-form-item>
        <el-form-item label="附件">
          <upload action="/api/files?action=parseDisclosure" @uploadSuccess="handleUploadSuccess" v-model="form.attachments" :file-list="attachments"></upload>
        </el-form-item>
      </el-form>
    </app-collapse>
</template>

<script>
import AppCollapse from '@/components/common/AppCollapse'
import Upload from '@/components/form/Upload'

export default {
  name: 'patentAddOther',
  props: ['type'],
  data () {
		return {
      progress_name: '',
			form: {
        words: '',
        remark: '',
        attachments: [],
			},
      attachments: [],
		}
  },
  methods: {
  	setForm (data) {
      for(let k in this.form) {
        const d = data[k];
        if(d == undefined) continue;
        if(k == 'attachments') {
          this.form[k] = d.map(_=>_.id);
          this.attachments = d;
        }else {
          this.form[k] = d;
        }
      }
      this.progress_name = data['progress'] ? data['progress']['name'] : ''; 
  	},
    submitForm () {
      return this.form;
    },
    checkForm (callback) {
      callback(true);
    },
    handleUploadSuccess (a, b, c) {
      this.$emit('uploadSuccess', a, b, c);
    }
  },
  components: { 
    AppCollapse, 
    Upload 
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>