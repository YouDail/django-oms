<template>
  <div
    class="components-container"
    style="height:100vh">
    <el-card>
      <div class="head-lavel">
        <div class="table-button">
          <el-button
            type="primary"
            icon="el-icon-plus"
            disabled
            @click="addForm=true">新建
          </el-button>
        </div>
        <div class="table-search">
          <el-input
            v-model="listQuery.search"
            placeholder="搜索 ..."
            @keyup.enter.native="searchClick">
            <i
              slot="suffix"
              class="el-icon-search el-input__icon"
              @click="searchClick"/>
          </el-input>
        </div>
      </div>
      <div>
        <el-table
          :data="tableData"
          border
          style="width: 100%"
          @selection-change="handleSelectionChange">
          <el-table-column type="selection"/>
          <el-table-column
            prop="username"
            label="用户名"
            sortable/>
          <el-table-column
            prop="email"
            label="Email"/>
          <el-table-column
            prop="avatar"
            label="Avatar">
            <template slot-scope="scope">
              <div class="avatar-wrapper">
                <img
                  :src="scope.row.avatar+'?imageView2/1/w/80/h/80'"
                  class="user-avatar">
              </div>
            </template>
          </el-table-column>
          <el-table-column
            prop="groups"
            label="所在组">
            <template slot-scope="scope">
              <div
                slot="reference"
                class="name-wrapper"
                style="text-align: center">
                <el-tag
                  v-for="item in scope.row.groups"
                  :key="item"
                  type="success">{{ item }}
                </el-tag>
              </div>
            </template>
          </el-table-column>
          <el-table-column
            prop="roles"
            label="角色"/>
        </el-table>
      </div>
      <div class="table-footer">
        <div class="table-button">
          <el-button
            :disabled="butstatus"
            type="danger"
            @click="deleteForm">删除用户
          </el-button>
        </div>
        <div class="table-pagination">
          <el-pagination
            :current-page.sync="currentPage"
            :page-sizes="pagesize"
            :page-size="listQuery.limit"
            :layout="pageformat"
            :total="tabletotal"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"/>
        </div>
      </div>
    </el-card>
    <el-dialog :visible.sync="addForm">
      <add-user @DialogStatus="getDialogStatus"/>
    </el-dialog>
    <el-dialog :visible.sync="editForm">
      <edit-user
        :rowdata="rowdata"
        @DialogStatus="getDialogStatus"/>
    </el-dialog>
  </div>
</template>

<script>
  import {getUser, deleteUser} from '@/api/user'
  import {LIMIT, pagesize, pageformat} from '@/config'
  import addUser from './components/adduser.vue'
  import editUser from './components/edituser.vue'
  import {mapGetters} from 'vuex'

  export default {
    components: {addUser, editUser},
    data() {
      return {
        tableData: [],
        tabletotal: 0,
        currentPage: 1,
        pagesize: pagesize,
        pageformat: pageformat,
        listQuery: {
          id__gt: 1,
          limit: LIMIT,
          offset: '',
          search: ''
        },
        addForm: false,
        editForm: false,
        rowdata: {},
        selectId: [],
        butstatus: true
      }
    },
    computed: {
      ...mapGetters([
        'username'
      ])
    },
    created() {
      this.fetchData()
    },
    methods: {
      fetchData() {
        getUser(this.listQuery).then(response => {
          this.tableData = response.data.results
          this.tabletotal = response.data.count
        })
      },

      getDialogStatus(data) {
        this.editForm = data
        this.addForm = data
        setTimeout(this.fetchData, 3000)
      },
      handleEdit(row) {
        this.editForm = true
        this.rowdata = row
        setTimeout(this.fetchData, 1000)
      },
      handleSelectionChange(val) {
        this.selectId = []
        for (var i = 0, len = val.length; i < len; i++) {
          this.selectId.push(val[i].id)
        }
        if (this.selectId.length > 0) {
          this.butstatus = false
        } else {
          this.butstatus = true
        }
      },

      searchClick() {
        this.fetchData()
      },
      handleSizeChange(val) {
        this.listQuery.limit = val
        this.fetchData()
      },
      handleCurrentChange(val) {
        this.listQuery.offset = (val - 1) * LIMIT
        this.fetchData()
      },
      deleteForm() {
        for (var i = 0, len = this.selectId.length; i < len; i++) {
          deleteUser(this.selectId[i]).then(response => {
            delete this.selectId[i]
          })
        }
        setTimeout(this.fetchData, 3000)
      }
    }
  }
</script>

<style lang='scss'>
  .avatar-wrapper {
    text-align: center;
    .user-avatar {
      width: 35px;
      height: 35px;
      border-radius: 10px;
    }
  }
</style>
