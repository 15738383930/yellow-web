<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.username" placeholder="用户名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button v-if="isAuth('sys:user:list')" @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('sys:user:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('sys:user:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>

      <el-table-column
        prop="username"
        header-align="center"
        align="center"
        label="用户名"
        width="150">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="昵称">
      </el-table-column>
      <el-table-column
        prop="phone"
        header-align="center"
        align="center"
        label="手机号">
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="状态">
        <template slot-scope="scope">
          <el-switch :disabled="!isAuth('sys:user:changestatus')" v-model="scope.row.status" :active-value="1" :inactive-value="2" @change="changestatusHandle(scope.row.id)" />
        </template>
      </el-table-column>
      <el-table-column
        prop="remark"
        header-align="center"
        align="center"
        label="备注">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间"
        width="100">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="50">
        <template slot-scope="scope">
          <el-popover placement="right" trigger="click" @hide="popoverHandle(scope.row.id)">
            <el-button v-if="isAuth('sys:user:update')" type="primary" icon="el-icon-edit" size="small" @click="addOrUpdateHandle(scope.row.id)" title="修改" />
            <el-button v-if="isAuth('sys:user:reset')" type="warning" icon="el-icon-refresh" size="small" @click="popoverHandle(scope.row.id, 'reset')" title="重置密码" />
            <el-button v-if="isAuth('sys:user:delete')" type="danger" icon="el-icon-delete" size="small" @click="popoverHandle(scope.row.id, 'delete')" title="删除" />
            <el-popover trigger="click" :ref="'reset' + scope.row.id">
              <p>确认要对【{{scope.row.name}}】进行密码重置吗？</p>
              <div style="text-align: right; margin: 0">
                <el-button size="mini" type="text" @click="popoverHandle(scope.row.id)">取消</el-button>
                <el-button type="primary" size="mini" @click="resetHandle(scope.row.id)">确定</el-button>
              </div>
            </el-popover>
            <el-popover trigger="click" :ref="'delete' + scope.row.id">
              <p>确认要删除【{{scope.row.name}}】吗？</p>
              <div style="text-align: right; margin: 0">
                <el-button size="mini" type="text" @click="popoverHandle(scope.row.id)">取消</el-button>
                <el-button type="primary" size="mini" @click="deleteHandle(scope.row.id)">确定</el-button>
              </div>
            </el-popover>
            <el-button type="text" icon="el-icon-more" slot="reference" />
          </el-popover>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from './user-add-or-update'

export default {
  data () {
    return {
      dataForm: {
        username: ''
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    }
  },
  components: {
    AddOrUpdate
  },
  activated () {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl(`/sys/user/page/list/${this.pageIndex}/${this.pageSize}`),
        method: 'get',
        params: this.$http.adornParams({
          'username': this.dataForm.username
        })
      }).then(({data}) => {
        if (data && data.success) {
          this.dataList = data.queryResult.list
          this.totalPage = data.queryResult.total
        } else {
          this.dataList = []
          this.totalPage = 0
          this.$message.error(data.message)
        }
        this.dataListLoading = false
      })
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
    // 新增 / 修改
    addOrUpdateHandle (id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    // 删除
    deleteHandle (id) {
      var userIds = id ? [id] : this.dataListSelections.map(item => {
        return item.id
      })
      this.$confirm(`确定对[id=${userIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/user/delete'),
          method: 'post',
          data: this.$http.adornData(userIds, false)
        }).then(({data}) => {
          if (data && data.success) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.message)
          }
        })
      }).catch(() => {})
    },
    // 重置密码
    resetHandle (id) {
      this.$http({
        url: this.$http.adornUrl(`/sys/user/reset/${id}`),
        method: 'post',
        data: this.$http.adornData()
      }).then(({data}) => {
        if (data && data.success) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.popoverHandle(id)
            }
          })
        } else {
          this.$message.error(data.message)
        }
      })
    },
    changestatusHandle (id) {
      this.$http({
        url: this.$http.adornUrl(`/sys/user/change/status/${id}`),
        method: 'post',
        data: this.$http.adornData()
      }).then(({data}) => {
        if (data && data.success) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.getDataList()
            }
          })
        } else {
          this.$message.error(data.message)
        }
      })
    },
    popoverHandle (id, type) {
      this.$refs[`reset${id}`].doClose()
      this.$refs[`delete${id}`].doClose()
      if (type) {
        this.$refs[`${type}${id}`].doShow()
      }
    }
  }
}
</script>
