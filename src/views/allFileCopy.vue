<template>
  <div class="home">
    <h3 style="text-align: center;">文件拷贝</h3>

    <div style="margin: 10px 0 30px 0;">只复制文件夹下的文件：
      <el-radio v-model="originBin" :label="true">不包括子文件夹</el-radio>
      <el-radio v-model="originBin" :label="false">包括子文件夹</el-radio>
    </div>
    
    <form id="formData">
      <el-row>
        <el-col :span="10">
          <p style="margin-top: 0;">
            <strong>资源文件夹：</strong>
            <input type="file" webkitdirectory id="source-folder" @change="folderChange" name="请选择资源文件夹" />
          </p>
          <p>
            <strong>资源文件：</strong>
            <input type="file" multiple id="source-file" @change="fileChange" name="请选择资源文件夹" />
          </p>

          <div style="text-align: left;">
            <h4 style="padding-left: 7px;">已选的文件:{{ source.length }} 个</h4>
            <div class="dyolege">
              <p
                v-for="(item, index) in source"
                :key="item.path"
                >
                <span :class="{'fileType': true, 'folder': item.type === 'folder'}">
                  {{ item.type === 'file' ? '文件' : '文件夹'}}
                </span>
                <span>{{ item.path }}</span>
                <i class="el-icon-delete" @click="deleteSourceItem(index)"></i>
              </p>
            </div>
          </div>
        </el-col>
        <el-col :span="14">
          <strong>目标文件夹（可多选）：</strong>
          <input type="file" webkitdirectory id="target" name="请选择目的文件夹" @change="targetChange"/>
          <div style="text-align: left; padding-left: 33px;">
            <h4 style="width: 110px;">已选的文件夹:</h4>
            <div class="dyolege">
              <p
                v-for="(item, index) in targetPathList"
                :key="item.path"
                >
                <span>{{ item.path }}</span>
                <el-progress style="width: 200px;display: inline-block;margin: 0 10px;" :text-inside="true" :stroke-width="26" :percentage="item.progress"></el-progress>
                <i class="el-icon-delete" @click="deleteItem(index)"></i>
              </p>
            </div>
          </div>

        </el-col>
      </el-row>
    </form>
    
    <el-button type="primary" @click="start">开始拷贝</el-button>
    <el-button type="primary" @click="reset">重置选中的文件</el-button>

    <div>
      <!-- <div style="width: 700px;margin: 30px auto;">
        <span>总进度：</span>
          <el-progress
            style="width: 500px;display: inline-block;"
            :text-inside="true"
            :stroke-width="26"
            :percentage="allProgress">
          </el-progress>
      </div> -->

      <p
        v-for="(item, index) in copyErrorPathList"
        :key="index">
        文件： <span style="color: red;">{{ item.copuOverPath }} （拷贝失败）</span>
      </p>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import copyFile from './copy.js';
import countSize from './count.js';
import child_process from 'child_process';
import nodeCmd from 'node-cmd';
import os from 'os';

export default {
  name: 'Home',
  components: {
    HelloWorld,
  },
  data() {
    return {
      originBin: false,
      source: [],
      targetPathList: [],
      copyErrorPathList: [],
    }
  },
  mounted() {
    console.log('文件合并模式');
  },
  methods: {
    reset() {
      document.getElementById('formData').reset();
      this.source = [];
      this.targetPathList = [];
      this.copyErrorPathList = [];
    },
    deleteItem(index) {
      this.targetPathList.splice(index, 1);
    },
    deleteSourceItem(index) {
      this.source.splice(index, 1);
    },
    targetChange() {
      let targetPath = document.getElementById('target').files[0];
      this.targetPathList.push({
        path: targetPath.path,
        progress: 0,
        nums: 0,
      });
      let list = new Set(this.targetPathList);
      this.targetPathList = [...list];
    },
    folderChange() {
      let sourceFolder = document.getElementById('source-folder').files[0];
      if(!sourceFolder) return;
      this.source.push({
        type: 'folder',
        path: sourceFolder.path,
      });
      let list = new Set(this.source);
      this.source = [...list];
    },
    fileChange() {
      let sourceFile = document.getElementById('source-file').files;
      if(!sourceFile) return;
      Array.from(sourceFile).forEach(item => {
        this.source.push({
          type: 'file',
          path: item.path,
          name: item.name,
        });
      });
      let list = new Set(this.source);
      this.source = [...list];
    },
    async start() {
      let self = this;
      if(this.source.length <= 0) {
        return this.$message.error('你还未选择资源文件夹');
      }
      
      if(this.targetPathList.length <= 0) {
        return this.$message.error('你还未选择目标文件夹');
      }

      let allSourceList = countSize(this.source).fileObj;
      let commandType = '';
      // window系统
      if (os.type() == 'Windows_NT') {
        commandType = 'Xcopy /Y /C';
        if(!this.originBin) commandType = commandType + ' /S'
      } else if (os.type() == 'Darwin') {
        // linux系统
        commandType = 'cp -r -f';
      }

      this.targetPathList.forEach(item => {
        allSourceList.forEach(ele => {
          let command = `${commandType} "${ele.path}" "${item.path}"`;
          child_process.exec(command, {}, (e) => {
            console.log('error: ', e);
            if(!e) {
              ++item.nums;
              item.progress = (item.nums / allSourceList.length).toFixed(1) * 100;
            }
          })
        });
      });
    },
  },
}
</script>

<style lang='scss' scoped>
.home {
  width: 900px;
  margin: 0 auto;
}

#formData {
  border-top: 1px solid #ddd;
  padding-top: 30px;
}

.fileType {
  display: inline-block;
  border: 1px solid #fff;
  padding: 2px 5px;
  margin-right: 5px;
  background-color: #71d3e2;
  color: #000;
  border-radius: 5px;
}

.folder {
  background-color: #e2e216;
}

.dyolege {
  max-height: 400px;
  overflow: auto;
  margin-top: 25px;

  /deep/ .el-icon-delete {
    cursor: pointer;
    font-size: 18px;
    color: red;
  }

  p {
    margin-top: 0;
    color: #0d4ad8;

    &:hover {
      color: green;
    }
  }
}

</style>
