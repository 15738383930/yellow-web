<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :append-to-body="true"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="字典值" prop="dictValue">
        <el-input v-model="dataForm.dictValue" :readonly="dataForm.id !== undefined && dataForm.id !== null && dataForm.id > 0" placeholder="字典名称"></el-input>
      </el-form-item>
      <el-form-item label="字典标签" prop="dictLabel">
        <el-input v-model="dataForm.dictLabel" :readonly="dataForm.id !== undefined && dataForm.id !== null && dataForm.id > 0" placeholder="字典类型"></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input-number v-model="dataForm.sort" controls-position="right" :min="0"></el-input-number>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data () {
    return {
      visible: false,
      menuList: [],
      menuListTreeProps: {
        label: 'menuName',
        children: 'children'
      },
      dataForm: {
        id: 0,
        dictTypeId: 0,
        dictValue: '',
        dictLabel: '',
        sort: 0,
        remark: ''
      },
      dataRule: {
        dictValue: [
          { required: true, message: '字典值不能为空', trigger: 'blur' }
        ],
        dictLabel: [
          { required: true, message: '字典标签不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    init (pid, id) {
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
      })
      this.dataForm.id = id || 0
      this.dataForm.dictTypeId = pid
      if (this.dataForm.id) {
        this.$http({
          url: this.$http.adornUrl(`/sys/dictionary/data/${this.dataForm.id}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.success) {
            this.dataForm.dictValue = data.result.dictValue
            this.dataForm.dictLabel = data.result.dictLabel
            this.dataForm.sort = data.result.sort
            this.dataForm.remark = data.result.remark
          }
        })
      }
    },
    // 表单提交
    dataFormSubmit () {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/sys/dictionary/data${this.dataForm.id ? '/' + this.dataForm.id : ''}`),
            method: `${this.dataForm.id ? 'put' : 'post'}`,
            data: this.$http.adornData({
              'dictValue': this.dataForm.dictValue,
              'dictLabel': this.dataForm.dictLabel,
              'remark': this.dataForm.remark,
              'sort': this.dataForm.sort,
              'dictTypeId': this.dataForm.dictTypeId
            })
          }).then(({data}) => {
            if (data && data.success) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.visible = false
                  this.$emit('refreshDataList')
                }
              })
            } else {
              this.$message.error(data.message)
            }
          })
        }
      })
    }
  }
}
</script>
