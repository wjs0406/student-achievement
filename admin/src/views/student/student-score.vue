<template>
  <div v-if="isWatch == 'true'">
    <el-table :data="data" style="width: 100%">
      <el-table-column prop="key" label="说明" width="200"> </el-table-column>
      <el-table-column prop="value" label="结果" width="200"> </el-table-column>
    </el-table>
    <el-button type="primary" icon="el-icon-download" @click="log"
      >打印成绩</el-button
    >
    <el-card class="box-card" v-if="isShow">
      <template #header>
        <div class="card-header">
          <span>下载成绩</span>
          <el-button class="button close" type="text" @click="isShow = false"
            >关闭</el-button
          >
        </div>
      </template>
      <div class="text item">
        <a :href="store.state.excel + url + '.xlsx'">点击下载成绩</a>
      </div>
    </el-card>
    <div class="alert" v-if="isShow"></div>
  </div>
  <h1 v-else>目前禁止察看成绩</h1>
</template>
<script setup>
import { ref } from "vue";
import { useStore } from "vuex";
import api from "@/modules/api";
import teacherPrint from "@/modules/teacher/teacher-print";

let store = useStore();
let url = ref("");
let tableData = ref([]);
let all = ref(0);
let gpa = ref(0);
let num = ref(0);

let data = ref([]);
api(`select * from achievement where stucode='${localStorage.student}'`).then(
  (res) => {
    tableData.value = res.res;
    let value = Object.values(res.res[0]).slice(2, 8);
    value.forEach((item, index) => {
      if (+item) {
        all.value += +item;
        num.value++;
      }
      if (+item > 90) {
        gpa.value += 4;
      } else if (+item > 80) {
        gpa.value += 3;
      } else if (+item > 70) {
        gpa.value += 2;
      } else if (+item > 60) {
        gpa.value += 1;
      }
    });
    let key = [
      "学号",
      "姓名",
      "VUE",
      "Node.js",
      "马克思",
      "创新",
      "MysSQL",
      "高等数学",
      "平局绩点",
      "平均分",
      "总分",
    ];
    let values = Object.values(tableData.value[0]).slice(0, 8);
    values.push((gpa.value / num.value).toFixed(2));
    values.push((all.value / num.value).toFixed(2));
    values.push(all.value);
    key.forEach((item, index) => {
      data.value.push({
        key: item,
        value: values[index],
      });
    });
  }
);

let isShow = ref(false);
function log() {
  let data = [];
  data.push([
    "姓名",
    "学号",
    "VUE",
    "Node.js",
    "马克思主义哲学",
    "创新与实践",
    "MySQL",
    "高等数学",
    "总分",
  ]);
  let newData = Object.values(tableData.value[0]);
  newData[8] = all.value;
  data.push(newData);
  let print = {
    name: localStorage.student + "同学的成绩单",
    data: data,
  };
  teacherPrint({ data: print }).then((res) => {
    isShow.value = true;
    url.value = print.name;
  });
}

let isWatch = ref("");
api(`select * from isshow`).then((res) => {
  isWatch.value = res.res[0].watch;
});
</script>
<style scoped lang='scss'>
.box-card {
  width: 500px !important;
  height: 400px !important;
  position: fixed;
  top: 200px;
  left: calc((100vw - 500px) / 2);
  z-index: 99999;
}
.close {
  float: right;
}
.alert {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0px;
  left: 0px;
  background: rgb(0, 0, 0, 0.3);
  z-index: 9999;
}
</style>