<template>
  <div style="display: inline;margin-right: 10px;">
    <el-button
      v-if="isIE"
      type="primary"
      size="mini"
      icon="el-icon-thumb"
      @click="dialogVisible = true"
    >拖拽生成表单</el-button>
    <el-dialog
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      top="100px"
      title="拖拽生成表单"
      width="1250px"
      append-to-body
    >
      <div class="updata-body" @dragenter="dragenter($event)">
        <div
          style="overflow-y: auto;cursor: pointer;height: 100px; line-height: 100px; text-align: center; font-size: 38px; color: #9a9a9a;"
          @drop="enentDrop($event)"
          @dragleave="dragleave($event)"
          @dragover="dragover($event)"
        >
          <div class="drag-box">{{drapText}}</div>
        </div>
        <el-tabs
          v-if="fileData && fileData.length > 0"
          v-model="currentFile"
          :tab-position="'left'"
          style="overflow-y: auto;height: 650px;width: 1150px;"
          @tab-click="leftTabClick(fileData[$event.index], $event.index)"
        >
          <template v-for="(item, index) in fileData">
            <el-tab-pane :name="item.file.name" :key="index+'fileData'">
              <div slot="label" @click.stop="leftTabClick(item, index)">{{ item.file.name }}</div>
              <template v-if="imgList && imgList.length > 0">
                <img
                  v-for="(img, index) in imgList"
                  :src="img.url"
                  :key="index"
                  style="margin-right: 10px;max-width: 300px;display:inline-block;"
                />
              </template>
              <el-button
                size="mini"
                type="primary"
                icon="el-icon-plus"
                style="float: right; margin-right: 15px;margin-bottom: 10px;"
                @click.stop="addMul(item.formDataArr)"
              >添加多条</el-button>
              <div
                v-for="(formD, ind) in item.formDataArr"
                :key="ind+'formDataArr'"
                style="margin-top: 10px;"
              >
                <div>
                  <el-input v-model="formD.name" :placeholder="'请输入姓名'+ind"></el-input>
                </div>
              </div>
            </el-tab-pane>
          </template>
        </el-tabs>
      </div>
      <span slot="footer" class="dialog-footer" style="text-align: center;">
        <el-button size="mini" type="primary" @click="dialogVisible = false">确 定</el-button>
        <el-button size="mini" @click="dialogVisible = false">取 消</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { isIE } from "@/utils";
import deepAssign from "@/utils/deepAssign";
export default {
  name: "FileReader",
  data() {
    return {
      dialogVisible: false,
      drapText: "拖拽一下, 开启美好一天",
      companySource: [],
      employeeSource: [],
      formData: {
        name: null
      },
      currentEntryFile: null,
      fileData: [],
      imgList: [],
      currentFile: null
    };
  },
  methods: {
    isIE,
    addMul(form) {
      form.push(deepAssign({}, this.formData));
    },
    isEmpty(val) {
      return val || "";
    },
    leftTabClick(data, index) {
      console.log(index);
      const entry = data.file;
      this.currentEntryFile = entry;
      const that = this;
      that.imgList = [];
      if (entry.isDirectory) {
        // 文件夹
        entry.getDirectory(
          entry.fullPath,
          {},
          function(entryDir) {
            var dirReader = entryDir.createReader();
            dirReader.readEntries(
              function(entries) {
                entries.forEach(file => {
                  if (file.isFile) {
                    file.filesystem.root.getFile(file.fullPath, {}, function(
                      fileEntry
                    ) {
                      fileEntry.file(function(getFile) {
                        var reader = new FileReader();
                        reader.readAsDataURL(getFile);
                        reader.onloadend = function() {
                          that.imgList.push({
                            url: reader.result,
                            fileName: getFile.name
                          });
                        };
                      });
                    });
                  }
                });
              },
              function() {}
            );
          },
          function() {
            console.log("打开失败");
          }
        );
      }
      this.currentFile = entry.name;
    },
    enentDrop: function(e) {
      e.stopPropagation();
      e.preventDefault();
      this.updataFun(e.dataTransfer.items);
    },
    dragleave(e) {
      e.stopPropagation();
      e.preventDefault();
    },
    dragenter(e) {
      e.stopPropagation();
      e.preventDefault();
      this.fileData = [];
      this.drapText = "松开鼠标完成上传";
    },
    dragover(e) {
      e.stopPropagation();
      e.preventDefault();
    },
    updataFun(data) {
      for (let i = 0; i <= data.length - 1; i++) {
        const item = data[i];
        if (item.kind === "file") {
          // FileSystemFileEntry 或 FileSystemDirectoryEntry 对象
          const entry = item.webkitGetAsEntry();
          // 递归地获取entry下包含的所有File
          this.getFileFromEntryRecursively(entry);
        }
      }
      this.drapText = "上传成功";
      setTimeout(() => {
        this.drapText = "将文件拖拽到此处进行上传";
      }, 500);
    },
    getFileFromEntryRecursively(entry) {
      const that = this;
      if (entry.isFile) {
        // 文件
      } else if (entry.isDirectory) {
        // 文件夹
        entry.getDirectory(
          entry.fullPath,
          {},
          function(entryDir) {
            var dirReader = entryDir.createReader();
            dirReader.readEntries(
              function(entries) {
                entries.forEach(item => {
                  if (item.isDirectory)
                    that.fileData.push({
                      file: item,
                      formDataArr: [deepAssign({}, that.formData)]
                    });
                });
                if (that.fileData && that.fileData.length > 0) {
                  that.currentFile = that.fileData[0].file.name;
                  that.leftTabClick(that.fileData[0], 0);
                  that.$emit("initData", that.fileData[0]);
                }
              },
              function() {}
            );
          },
          function() {
            console.log("打开失败");
          }
        );
      }
    }
  }
};
</script>

<style>
._magnifier {
  margin-right: 10px;
}
</style>