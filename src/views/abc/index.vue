<template>
  <div class="app-container">

    <div class="filter-container">
      <el-input v-model="params.title" placeholder="Title" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-select v-model="params.importance" placeholder="Imp" clearable style="width: 90px" class="filter-item">
        <el-option v-for="item in importanceOptions" :key="item" :label="item" :value="item" />
      </el-select>
      <el-select v-model="params.type" placeholder="Type" clearable class="filter-item" style="width: 130px">
        <el-option v-for="item in calendarTypeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key" />
      </el-select>
      <el-select v-model="params.sort" style="width: 140px" class="filter-item" @change="handleFilter">
        <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key" />
      </el-select>
      <el-button class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">
        Search
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        Add
      </el-button>
      <el-button :loading="downloadLoading" class="filter-item" type="primary" icon="el-icon-download" @click="handleDownload">
        Export
      </el-button>
      <el-checkbox v-model="showReviewer" class="filter-item" style="margin-left:15px;" @change="tableKey=tableKey+1">
        reviewer
      </el-checkbox>
    </div>

    <el-table v-loading="listLoading" :data="list" border fit highlight-current-row style="width: 100%">
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>
      <el-table-column align="center" label="ID" width="80">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>

      <el-table-column width="180px" align="center" label="Date">
        <template slot-scope="{row}">
          <span>{{ row.timestamp | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>

      <el-table-column width="100px" label="Importance">
        <template slot-scope="{row}">
          <svg-icon v-for="n in + row.importance" :key="n" icon-class="star" class="meta-item__icon" />
        </template>
      </el-table-column>

      <el-table-column class-name="status-col" label="Status" width="110">
        <template slot-scope="{row}">
          <el-tag :type="row.status | statusFilter">
            {{ row.status }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column min-width="300px" label="Title">
        <template slot-scope="{row}">
          <template v-if="row.edit">
            <el-input v-model="row.title" class="edit-input" size="small" />
            <el-button
              class="cancel-btn"
              size="small"
              icon="el-icon-refresh"
              type="warning"
              @click="cancelEdit(row)"
            >
              cancel
            </el-button>
          </template>
          <span v-else>{{ row.title }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="Actions" width="300px">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.edit"
            type="success"
            size="mini"
            @click="confirmEdit(scope.row)"
          >
            Ok
          </el-button>
          <el-button
            v-else
            type="primary"
            size="mini"
            @click="scope.row.edit=!scope.row.edit"
          >
            Edit
          </el-button>
          <el-button
            type="danger"
            size="mini"
            @click="handleDelete(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <template>
      <div class="pagination-container">
        <el-pagination
          background
          :hide-on-single-page="true"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="params.page"
          :page-size="params.limit"
          :page-sizes="[10, 20, 50]"
          layout="sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </div>

    </template>

  </div>
</template>

<script>

  import { fetchList } from '@/api/article'

  export default {
    name: 'abc', // 页面标识
    data() { // 变量定义
      return {
        list: [],
        total: 100,
        listLoading: true,
        params: { // 查询参数
          page: 1,
          limit: 10,
          title: '',
          importance: '',
          type: '',
          sort: '',
        },
        importanceOptions: [1,2,3,4,5],
        calendarTypeOptions: [{key:'key1',display_name:'value1'},{key:'key2',display_name:'value2'}],
        sortOptions: [{key:'key1',label:'label1'},{key:'key2',label:'label2'}],
        showReviewer: '',
        tableKey: 0,
        downloadLoading: false
      }
    },
    created() {
      this.getList() // 查询列表
    },
    filters: {
      statusFilter(status) { // 状态枚举
        const statusMap = {
          published: 'success',
          draft: 'info',
          deleted: 'danger'
        }
        return statusMap[status]
      }
    },
    methods: {
      handleFilter() { // 查询按钮点击事件
        console.log('查询按钮点击事件');
        this.getList();
      },
      handleCreate() { // Add按钮点击事件
        console.log('Add按钮点击事件');
      },handleDownload() { // 下载按钮点击事件
        console.log('下载按钮点击事件');
      },
      async getList() { // 请求后台, 当前使用模拟数据, 请自行修改为接口请求
        this.listLoading = true // 打开加载动画
        const { data } = await fetchList(this.params)
        const items = data.items
        this.list = items.map(v => {
          this.$set(v, 'edit', false) // https://vuejs.org/v2/guide/reactivity.html
          v.originalTitle = v.title //  will be used when user click the cancel botton
          return v
        })

        this.listLoading = false // 关闭加载动画
      },
      cancelEdit(row) { // 取消编辑按钮事件
        row.title = row.originalTitle
        row.edit = false
        this.$message({
          message: 'The title has been restored to the original value',
          type: 'warning'
        })
      },
      confirmEdit(row) { // 确认修改按钮事件
        row.edit = false
        row.originalTitle = row.title
        this.$message({
          message: 'The title has been edited',
          type: 'success'
        })
      },
      handleSizeChange(val) { // 分页插件每页数量变更事件
        this.params.limit = val;
        this.getList();
        console.log(`每页 ${val} 条`);
      },
      handleCurrentChange(val) { // 分页插件当前页变更事件
        this.params.page = val;
        this.getList();
        console.log(`当前页: ${val}`);
      },
      handleDelete(index, row) { // 删除按钮事件
        console.log(index, row);
      }
    }
  }
</script>

<style scoped>
  .edit-input {
    padding-right: 100px;
  }
  .cancel-btn {
    position: absolute;
    right: 15px;
    top: 10px;
  }
</style>
