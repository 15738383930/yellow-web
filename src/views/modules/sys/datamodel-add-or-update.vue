<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="数据模型名称" prop="modelName">
        <el-input v-model="dataForm.modelName" placeholder="数据模型名称" :readonly="dataForm.id !== undefined && dataForm.id !== null && dataForm.id > 0"></el-input>
      </el-form-item>
      <el-form-item label="数据模型类型" prop="modelType">
        <el-input v-model="dataForm.modelType" placeholder="数据模型类型" :readonly="dataForm.id !== undefined && dataForm.id !== null && dataForm.id > 0"></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input-number v-model="dataForm.sort" controls-position="right" :min="0"></el-input-number>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
      </el-form-item>
      <el-form-item label="绑定字典" prop="remark">
        <el-transfer
          filterable
          v-model="dataForm.dictionaryIds"
          :data="dictionaryList"
          :props="props"
          :filter-method="filterMethod"
          filter-placeholder="请输入字典名称"
          :titles="['字典池', '已绑定']">
        </el-transfer>
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
        dictionaryList: [],
        dataForm: {
          id: 0,
          modelName: '',
          modelType: '',
          remark: '',
          dictionaryIds: [],
          sort: ''
        },
        props: {
          label: 'dictName',
          key: 'id'
        },
        dataRule: {
          modelName: [
            { required: true, message: '数据模型名称不能为空', trigger: 'blur' }
          ],
          modelType: [
            { required: true, message: '数据模型类型不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.$http({
          url: this.$http.adornUrl('/sys/dictionary/select'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.dictionaryList = data && data.success ? data.queryResult.list : []
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }).then(() => {
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/datamodel/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.success) {
                this.dataForm.modelName = data.result.modelName
                this.dataForm.modelType = data.result.modelType
                this.dataForm.remark = data.result.remark
                this.dataForm.sort = data.result.sort
                this.dataForm.dictionaryIds = data.result.dictionaryIds
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/datamodel${this.dataForm.id ? '/' + this.dataForm.id : ''}`),
              method: `${this.dataForm.id ? 'put' : 'post'}`,
              data: this.$http.adornData({
                'modelName': this.dataForm.modelName,
                'modelType': this.dataForm.modelType,
                'remark': this.dataForm.remark,
                'sort': this.dataForm.sort,
                'dictionaryIds': this.dataForm.dictionaryIds
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
      },
      filterMethod (query, item) {
        return item.dictName.indexOf(query) > -1
      }
    }
  }
</script>
