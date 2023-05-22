<template>
  <el-dialog
    title="异常日志"
    :close-on-click-modal="false"
    :visible.sync="visible"
    style="background-color: #8a979e">
    <div v-html="text" style="height: 500px; overflow-y: auto; background-color: #f0f9eb"></div>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">关闭</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        text: ''
      }
    },
    created () {
    },
    methods: {
      init (id) {
        this.$http({
          url: this.$http.adornUrl(`/sys/log/details/${id}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.success) {
            this.text = data.result
          } else {
            this.$message.error(data.message)
          }
        }).then(() => {
          this.visible = true
        })
      }
    }
  }
</script>
