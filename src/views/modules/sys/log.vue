<template>
  <div class="mod-log">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="用户名／用户操作" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList(true)">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      style="width: 100%">
      <el-table-column
        prop="operateName"
        header-align="center"
        align="center"
        label="用户名">
      </el-table-column>
      <el-table-column
        prop="operateEvent"
        header-align="center"
        align="center"
        :show-overflow-tooltip="true"
        label="用户操作">
      </el-table-column>
      <el-table-column
        prop="requestMethod"
        header-align="center"
        align="center"
        :show-overflow-tooltip="true"
        label="请求方法">
      </el-table-column>
      <el-table-column
        prop="methodName"
        header-align="center"
        align="center"
        :show-overflow-tooltip="true"
        label="操作内容">
      </el-table-column>
      <el-table-column
        prop="time"
        header-align="center"
        align="center"
        label="执行时长(毫秒)">
      </el-table-column>
      <el-table-column
        prop="operateStatus"
        header-align="center"
        align="center"
        label="操作状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.operateStatus === '成功'" size="small" type="success">{{scope.row.operateStatus}}</el-tag>
          <el-tag v-if="scope.row.operateStatus === '失败'" style="cursor: pointer;" @click="openDetails(scope.row.id)" size="small" type="danger">{{scope.row.operateStatus}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="operateTime"
        header-align="center"
        align="center"
        width="180"
        label="操作时间">
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
    <!-- 异常详情 -->
    <log-details v-if="detailsVisible" ref="logDetails" />
  </div>
</template>

<script>
  import LogDetails from './log-details'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        selectionDataList: [],
        detailsVisible: false
      }
    },
    components: {
      LogDetails
    },
    created () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList (where) {
        if (where) {
          this.pageSize = 10
          this.pageIndex = 1
        }
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl(`/sys/log/page/list/${this.pageIndex}/${this.pageSize}`),
          method: 'get',
          params: this.$http.adornParams({
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.success) {
            this.dataList = data.queryResult.list
            this.totalPage = data.queryResult.total
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      openDetails (id) {
        this.detailsVisible = true
        this.$nextTick(() => {
          this.$refs.logDetails.init(id, 2)
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
      }
    }
  }
</script>
