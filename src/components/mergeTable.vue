<template>
  <div style="width:800px">
    <el-table :data="tableData" stripe border :span-method="arraySpanMethod">
      <el-table-column prop="class" label="班级"></el-table-column>
      <el-table-column prop="name" label="姓名"></el-table-column>
      <el-table-column prop="course" label="科目"></el-table-column>
      <el-table-column prop="score" label="成绩"></el-table-column>
    </el-table>
  </div>
</template>

<script>
import mergeCell from "@/util/mergeCell"; 
export default {
  data() {
    return {
      tableData: [
        {
          class: "class1",
          name: "王小虎",
          course: "英语",
          score: "88"
        },
        {
          class: "class1",
          name: "王小虎",
          course: "语文",
          score: "90"
        },
        {
          class: "class1",
          name: "王小虎",
          course: "数字",
          score: "75"
        },
        {
          class: "class1",
          name: "李明",
          course: "英语",
          score: "76"
        },
        {
          class: "class1",
          name: "李明",
          course: "语文",
          score: "80"
        },
        {
          class: "class1",
          name: "李明",
          course: "数字",
          score: "65"
        },
        {
          class: "class2",
          name: "张斌",
          course: "英语",
          score: "85"
        },
        {
          class: "class2",
          name: "张斌",
          course: "语文",
          score: "70"
        },
        {
          class: "class2",
          name: "张斌",
          course: "数字",
          score: "100"
        },
        {
          class: "class2",
          name: "赵四",
          course: "英语",
          score: "56"
        },
        {
          class: "class2",
          name: "赵四",
          course: "语文",
          score: "45"
        },
        {
          class: "class2",
          name: "赵四",
          course: "数字",
          score: "21"
        }
      ],
      //合并单元格
      colNameArray: [
        "class",
        "name",
      ], //需要合并的列数组 [字段名1,字段名2,字段名2,...]
      mergeArray: [] //计算合并行的数组
    };
  },
  created(){
      this.merge(this.tableData);
  },
  methods: {
    merge(data) {
      this.mergeArray = mergeCell(this.colNameArray, data);
      console.log("=============");
      console.log(this.mergeArray);
    },
    //单元格合并方法
    arraySpanMethod({ row, column, rowIndex, columnIndex }) {
      if (columnIndex < this.mergeArray.length) {
        const _row = this.mergeArray[columnIndex][rowIndex];
        const _col = _row > 0 ? 1 : 0; //如果被合并了_row=0则它这个列需要取消
        return {
          rowspan: _row,
          colspan: _col
        };
      }
    }
  }
};
</script>

<style lang="scss" scoped>
</style>