<template>
  <div class="mod-role">
    <avue-crud ref="crud"
               :page="page"
               :data="dataList"
               :option="tableOption"
               @search-change="searchChange"
               @selection-change="selectionChange"
               @on-load="getDataList">
      <template slot="menuLeft">
        <el-button type="primary"
                   icon="el-icon-plus"
                   size="small"
                   v-if="isAuth('sys:role:save')"
                   @click.stop="addOrUpdateHandle()">新增</el-button>

        <el-button type="danger"
                   @click="deleteHandle()"
                   v-if="isAuth('sys:role:delete')"
                   size="small"
                   :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </template>

      <template slot-scope="scope"
                slot="menu">
        <el-button type="primary"
                   icon="el-icon-edit"
                   size="small"
                   v-if="isAuth('sys:role:update')"
                   @click.stop="addOrUpdateHandle(scope.row.roleId)">编辑</el-button>

        <el-button type="danger"
                   icon="el-icon-delete"
                   size="small"
                   v-if="isAuth('sys:role:delete')"
                   @click.stop="deleteHandle(scope.row.roleId)">删除</el-button>
      </template>
    </avue-crud>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible"
                   ref="addOrUpdate"
                   @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import {tableOption} from '@/crud/sys/role'
import AddOrUpdate from './role-add-or-update'

export default {
  data () {
    return {
      dataList: [],
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      tableOption: tableOption,
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      }
    }
  },
  components: {
    AddOrUpdate
  },
  methods: {
    // 获取数据列表
    getDataList (page, params, done) {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/sys/role/page'),
        method: 'get',
        params: this.$http.adornParams(
          Object.assign(
            {
              current: page == null ? this.page.currentPage : page.currentPage,
              size: page == null ? this.page.pageSize : page.pageSize
            },
            params
          )
        )
      }).then(({ data }) => {
        this.dataList = data.records
        this.page.total = data.total
        this.dataListLoading = false
        if (done) {
          done()
        }
      })
    },
    // 条件查询
    searchChange (params, done) {
      this.getDataList(this.page, params, done)
    },
    // 多选变化
    selectionChange (val) {
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
      var ids = id ? [id] : this.dataListSelections.map(item => {
        return item.roleId
      })
      this.$confirm(`确定进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/role'),
          method: 'delete',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.getDataList()
            }
          })
        })
      }).catch(() => { })
    }
  }
}
</script>
