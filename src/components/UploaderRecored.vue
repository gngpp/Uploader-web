<template>
  <el-card>
    <el-table
      :data="tableData"
      stripe
      highlight-current-row
      border
      style="width: 100%"
      height="500"
    >
      <el-table-column
        fixed
        prop="name"
        label="文件名"
        width="180">
      </el-table-column>
      <el-table-column
        prop="size"
        label="文件大小"
        width="180">
      </el-table-column>
      <el-table-column
        prop="md5"
        label="MD5">
      </el-table-column>
      <el-table-column
        sortable
        prop="uploadTime"
        label="上传时间">
      </el-table-column>
      <el-table-column label="操作" width="100">
        <template slot-scope="scope">
          <el-button
            plain
            size="mini"
            type="danger"
            @click="open(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="block">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :page-count="totalPage"
        :current-page="pageNumber"
        :page-size="pageSize"
        :total="totalRecord"
        layout="total, prev, pager, next"
      >
      </el-pagination>
    </div>
  </el-card>
</template>

<script>
import axios from "axios";
import moment from 'moment'
export default {
  name: "UploaderData",
  data() {
    return {
      pageSize: 5,
      pageNumber:1,
      totalPage: 0,
      totalRecord:0,
      tableData: []
    }
  },
  mounted() {
    this.changePage(this.pageNumber)
  },
  methods:{
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    changePage(pageNumber){
      let formData = new FormData();
      formData.append("pageNumber",pageNumber)
      formData.append("pageSize", this.pageSize)
      axios.post('http://localhost:8080/record/page', formData ,{
        headers:{
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }).then(res => {
        let  data = res.data
        console.log(data)
        this.pageNumber = data.pageCount
        this.totalRecord = data.totalRecord
        this.totalPage = data.totalPage
        this.tableData = data.data
      })
    },
    handleCurrentChange(val) {
        this.changePage(val)
    },
    handleEdit(index, row) {
      console.log(index, row);
    },
    handleDelete(index, row) {
      let formData = new FormData()
      formData.append('id', row.id)
      axios.post("http://localhost:8080/record/delete", formData , {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }).then(res => {
        this.$notify({
          title: '成功',
          message: '删除成功',
          type: 'success'
        });
        if (index == 0) {
          this.pageNumber -1
          this.changePage(this.pageNumber -1 )
        }
        this.changePage(this.pageNumber)
      }).catch(e => {
        this.$notify({
          title: '错误',
          message: '删除失败',
          type: 'error'
        });
      })
    },
    open(index, row) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(res => {
        this.handleDelete(index, row)
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    }
  }
}
</script>
