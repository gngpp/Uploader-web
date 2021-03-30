<template>
  <div>

    <el-upload
      class="upload-demo"
      ref="upload"
      action="https://jsonplaceholder.typicode.com/posts/"
      :on-preview="handlePreview"
      :on-remove="handleRemove"
      :show-file-list="false"
      :http-request='addFileToFormData'
      multiple
    >
      <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
      <el-button style="margin-left: 10px;" size="small" type="success" @click="upload">上传到服务器</el-button>
    </el-upload>
    <div>
      <p>上传进度</p>
      <el-progress :percentage="percentage" ></el-progress>
    </div>
    <el-table
      :data="tableData"
      style="width: 100%; margin: 10px 10px;">
      <el-table-column
        label="文件名">
        <template slot-scope="scope">
          <span>{{scope.row.name}}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="大小">
        <template slot-scope="scope">
          <span>{{scope.row.size}}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作">
        <template slot-scope="scope">
          <el-button type="danger" @click="deleteElement(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
  import Uploader from './Uploader'
  import axios from "axios";
  export default {
    name: "MultiFileUpload",
    data() {
      return {
        tableData: [],
        fileList:[],
        fileFormData: null, // 将要上传的formdata数据
        percentage: 0, // 存放上传百分比
      }
    },
    created() {
      this.fileFormData = new FormData();
    },
    watch: {
      fileList: {
        handler() {
          this.tableData = [];
          this.fileList.forEach((e) => {
            this.tableData.push({
              name: e.name,
              size: this.renderSize(e.size),
              id: e.uid
            });
          });
        }
      }
    },
    methods: {
      renderSize(value) {
        if (null == value || value == '') {
          return "0 Bytes";
        }
        var unitArr = new Array("Bytes", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB");
        var index = 0,
          srcsize = parseFloat(value);
        index = Math.floor(Math.log(srcsize) / Math.log(1024));
        var size = srcsize / Math.pow(1024, index);
        //  保留的小数位数
        size = size.toFixed(2);
        return size + unitArr[index];
      },
      addFileToFormData(file) {
        /*
         * 每选一个文件都会执行一次该函数
         * 也就把每个文件都添加到 this.fileFormData 里面
         * 在添加之前也可以加上自己的一些判断逻辑
         * 当然也可以直接利用 el-upload 组件自带的限制功能
        */
        this.fileList.push(file.file)
        //this.fileFormData.append('fileList', file.file);
      },
      uploadProgress(progressEvent){
        /*
         * progressEvent.loaded :已上传量
         * progressEvent.total :上传的总大小
        */
        const p = Math.floor((progressEvent.loaded / progressEvent.total) * 100);
        this.percentage = p;
      },
      upload() {
        if (this.fileList.length === 0) {
          this.$notify({
            title: '警告',
            message: '当前没有文件可上传',
            type: 'warning'
          });
          return
        }
        this.fileList.forEach(e=>{
          this.fileFormData.append('fileList', e);
        })
        const formData = this.fileFormData;
        const fn = this.uploadProgress; // 我们自己处理上传进度的函数
        axios.post('http://localhost:8080/multi_file', formData ,{
          headers:{
            'Content-Type': 'multipart/form-data'
          },
          onUploadProgress: fn
        }).then(res => {
          this.$notify({
            title: '成功',
            message: '批量上传成功',
            type: 'success'
          });
          this.fileList.forEach(e=>{
            console.log(e)
          })
          this.fileFormData.delete('fileList')
          this.fileList = []
          this.$refs.upload.clearFiles();
        })
      },
      handleRemove(file, fileList) {
        this.fileList = this.fileList.filter(value => {
          return value.uid !== file.uid
        })
      },
      handlePreview(file) {
        console.log(file);
      },
      deleteElement(uid) {
        this.fileList = this.fileList.filter(value => {
          return value.uid !== uid
        })
      }
    }
  }
</script>
