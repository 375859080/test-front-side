<template>
  <div class="app-container">
    <h2>用户管理</h2>
    <el-table
      v-loading="listLoading"
      :data="list.slice((currentPage-1)*pageSize,currentPage*pageSize)"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column label="用户名" prop="userName" />
      <el-table-column label="工号" prop="workNumber" />
      <el-table-column label="电话号码" prop="phoneNumber" />
      <el-table-column label="姓名" prop="name" />
      <el-table-column label="性别" prop="gender" />
      <el-table-column label="用户余额" prop="remainMoney" />
      <el-table-column label="用户星级" prop="starLevel" />
      <!--      <el-table-column label="身份证信息" />-->
      <!--      <el-table-column label="驾驶证信息" />-->
      <el-table-column label="操作">
        <template scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index,scope.row)">编辑</el-button>
          <el-button size="mini" @click="handleDelete(scope.$index,scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div style="text-align: center">
      <el-pagination
        background
        layout="total, prev, pager, next"
        :total="total"
        :current-page="currentPage"
        :page-size="pageSize"
        @current-change="handleCurrentChange"
      />
    </div>
    <!--    弹出的编辑框-->
    <el-dialog :visible.sync="deleteFormVisible" title="确定删除这一行数据么？">
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="deleteSubmit">确定</el-button>
      </div>
    </el-dialog>
    <el-dialog :visible.sync="editFormVisible" title="编辑用户" :close-on-click-modal="false">
      <el-form ref="editForm" :model="editForm" label-width="80px"><!--// :rules="editFormRules">-->
        <el-form-item label="用户名">
          <el-input v-model="editForm.userName" auto-complete="off" />
        </el-form-item>
        <el-form-item label="用户名">
          {{ editForm.userName }}
        </el-form-item>
        <el-form-item label="工号">
          <el-input v-model="editForm.workNumber" />
        </el-form-item>
        <el-form-item label="电话号码">
          <el-input v-model="editForm.phoneNumber" />
        </el-form-item>
        <el-form-item label="姓名">
          <el-input v-model="editForm.name" />
        </el-form-item>
        <el-form-item label="性别">
          <el-input v-model="editForm.gender" />
        </el-form-item>
        <el-form-item label="用户余额">
          <el-input v-model="editForm.remainMoney" />
        </el-form-item>
        <el-form-item label="用户星级">
          <el-input v-model="editForm.starLevel" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="editSubmit">提交</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { updateUser, getUsersInCompany } from '@/api/user'
import { getIndex } from '@/utils/index'

export default {
  name: 'Index',
  data() {
    return {
      currentPage: 1,
      pageSize: 4,
      listLoading: false,
      list: [],
      editFormVisible: false,
      editFormIndex: 0,
      editForm: {
      },
      deleteFormVisible: false
    }
  },
  computed: {
    tables() {
      const type = this.chooseType
      const search = this.searchReason
      return this.datalist.filter((item) => {
        return String(item['reimburseType']) === type || type === '' || type === null
      }).filter((dataNews) => {
        return Object.keys(dataNews).some((key) => {
          const item = (function(item, filters) {
            for (var index in filters) {
              if (filters[index] === item) {
                return item
              }
            }
          })(key, this.searchItems)
          return String(dataNews[item]).toLowerCase().indexOf(search) > -1 || search === null
        })
      })
    },
    total() {
      return this.list.length
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      this.listLoading = true
      getUsersInCompany().then(response => {
        this.list = response.data
        this.listLoading = false
      })
    },
    handleCurrentChange(val) {
      this.currentPage = val
    },
    handleEdit(index, row) {
      this.editFormVisible = true
      this.editFormIndex = index
      this.editForm = Object.assign({}, row)
    },
    handleDelete(index, row) {
      this.deleteFormVisible = true
    },
    editSubmit() {
      // 提交修改的用户信息
      updateUser(this.editForm).then(response => {
        if (response.code === 200) {
          this.$message({
            showClose: true,
            message: '修改成功',
            type: 'success'
          })
          const index = getIndex(this.currentPage, this.pageSize, this.editFormIndex)
          this.list[index] = this.editForm// Object.assign({}, this.editForm)
          console.log(index + '**' + this.editForm)
          this.editFormVisible = false
        }
      })
    }
  }
}
</script>

<style scoped>

</style>
