<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="角色代码" prop="roleCode">
        <el-input v-model="dataForm.roleCode" :readonly="dataForm.id !== undefined && dataForm.id !== null && dataForm.id > 0" placeholder="SYSTEM">
          <template slot="prepend">ROLE_</template>
        </el-input>
      </el-form-item>
      <el-form-item label="角色名称" prop="roleName">
        <el-input v-model="dataForm.roleName" placeholder="角色名称"></el-input>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
      </el-form-item>
      <el-form-item size="mini" label="授权">
        <el-tree
          :data="menuList"
          :props="menuListTreeProps"
          node-key="id"
          ref="menuListTree"
          :default-expand-all="true"
          show-checkbox>
        </el-tree>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { treeDataTranslate } from '@/utils'
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
          roleCode: '',
          roleName: '',
          remark: ''
        },
        dataRule: {
          roleCode: [
            { required: true, message: '角色代码不能为空', trigger: 'blur' }
          ],
          roleName: [
            { required: true, message: '角色名称不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.$http({
          url: this.$http.adornUrl('/sys/menu/list'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.success) {
            this.menuList = treeDataTranslate(data.queryResult.list)
          } else {
            this.$message.error(data.message)
          }
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
            this.$refs.menuListTree.setCheckedKeys([])
          })
        }).then(() => {
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/role/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.success) {
                this.dataForm.roleCode = data.result.roleCode
                this.dataForm.roleName = data.result.roleName
                this.dataForm.remark = data.result.remark
                const idx = data.result.menus.indexOf(this.tempKey)
                if (idx !== -1) {
                  data.result.menus.splice(idx, data.result.menus.length - idx)
                }
                data.result.menus.forEach((value) => {
                  this.$refs.menuListTree.setChecked(value, true, false)
                })
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
              url: this.$http.adornUrl(`/sys/role/${!this.dataForm.id ? 'save' : `update/${this.dataForm.id}`}`),
              method: 'post',
              data: this.$http.adornData({
                'roleId': this.dataForm.id || undefined,
                'roleCode': this.dataForm.roleCode,
                'roleName': this.dataForm.roleName,
                'remark': this.dataForm.remark,
                'menus': [].concat(this.$refs.menuListTree.getCheckedKeys(), this.$refs.menuListTree.getHalfCheckedKeys())
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
