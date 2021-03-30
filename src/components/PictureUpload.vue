<template>
  <div>
    <uploader
      browse_button="browse_button"
      :url="server_config.url+'/file'"
      :FilesAdded="filesAdded"
      @inputUploader="inputUploader"
    />
    <el-button id="browse_button" type="primary" plain>选择图片</el-button>
    <br/>
    <br/>
    <el-tag type="info">图片预览区域</el-tag>
    <el-row >
      <el-col style="margin: 20px 20px" :span="8" v-for="(image, index) in images" :key="index" :offset="index > 0 ? 1 : 0">
        <el-card :body-style="{ padding: '0px' }">
          <img  :src="image.src" class="image">
          <div style="padding: 5px; display: flex">
              <div style="flex: 3; display: flex; justify-content:center;align-items:center;">
                <span v-if="image.file.status === 1" style="color: aqua;">准备上传</span>
                <span v-if="image.file.status === 4" style="color: brown">上传失败</span>
                <span v-if="image.file.status === 5" style="color: chartreuse">已上传</span>
                <el-progress v-if="image.file.status === 2" :text-inside="true" :stroke-width="20" :percentage="image.file.percent"></el-progress>
              </div>
              <el-button style="flex: 1" type="text" class="button" @click="deleteFile(index,image.file)" plain>删除</el-button>
          </div>
        </el-card>
      </el-col>
    </el-row>
    <br/>
    <el-button type="danger" @click="up.start()" plain>开始上传</el-button>
  </div>
</template>

<script>
  import Uploader from './Uploader'
  import FileUrl from '../models/file-url'

  export default {
    name: "PictureUpload",
    data() {
      return {
        currentDate: new Date(),
        server_config: this.global.server_config,
        files:[],
        up: {},
        images: [],
        changed: false
      }
    },
    components: {
      'uploader': Uploader
    },
    watch: {
      changed() {
        let images = [];
        this.files.forEach((e) => {
          images.push({src: e.imgsrc, file: e});
        });
        this.images = images;
      }
    },
    methods: {
      renderSize(value){
        if(null==value||value==''){
          return "0 Bytes";
        }
        var unitArr = new Array("Bytes","KB","MB","GB","TB","PB","EB","ZB","YB");
        var index=0,
          srcsize = parseFloat(value);
        index=Math.floor(Math.log(srcsize)/Math.log(1024));
        var size =srcsize/Math.pow(1024,index);
        //  保留的小数位数
        size=size.toFixed(2);
        return size+unitArr[index];
      },
      inputUploader(up) {
        this.up = up;
        this.files = up.files;
      },
      filesAdded(up, files) {
        files.forEach((f) => {
          FileUrl(f.getNative(), (err, imgsrc) => {
            f.imgsrc = imgsrc;
            this.changed = !this.changed;
          });
        });
      },
      deleteFile(index,file) {
        this.images.splice(index,1);
        this.up.removeFile(file);
      }
    }
  }
</script>

<style>
.time {
  font-size: 13px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  padding: 0;
  float: right;
}

.image {
  width: 100%;
  display: block;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both
}
</style>
