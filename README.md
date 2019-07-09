# vue-test

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run dev
```

### Compiles and minifies for production
```
npm run build
```

### Run your tests
```
npm run test
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


## VUE+ElementUI 实现合并单元格
#### 先写一个表格，模拟一些数据: 

`mergeTable.vue` :
```
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
``` 
准备数据:

```
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
        }
        //........ 过长省略
      ],
```

在`data`中声明两个关键数组： 

```
 //合并单元格
  colNameArray: [
    "class",
    "name",
  ], //需要合并的列数组 [字段名1,字段名2,字段名2,...]
  mergeArray: [] //计算合并行的数组
```

编辑计算函数： 

> 通过给table传入span-method方法可以实现合并行或列，方法的参数是一个对象，里面包含当前行row、当前列column、当前行号rowIndex、当前列号columnIndex四个属性。该函数可以返回一个包含两个元素的数组，第一个元素代表rowspan，第二个元素代表colspan。 也可以返回一个键名为rowspan和colspan的对象。 

要编辑一个函数 `span-method` 用来计算每个单元格的 `rowspan` 和 `colspan` ;

在编写`span-method`函数前，要先计算出哪些单元格需要合并，以及合并的规则。 
在上面的表格中，暂且规定需要合并的是`班级`和`姓名`的行合并；也就是也一个班级或者同一姓名的单元格合并显示。 效果如下： 

![合并后效果](https://user-gold-cdn.xitu.io/2019/7/8/16bd0e4dd70d69b5?w=819&h=648&f=png&s=42271)

所以先要有一个函数用来处理合并规则及合并的行和列。 

编写一个通用工具，`mergeCell.js ：  /src/util/mergeCell.js`

```
let colNumArray = [];   // 每一列需要合并的行数数组
let colPosIndexArray = [];   // 记录需要合并的列的位置

/**
 * 数据初始化
 * @param {需要合并的列数，即 字段名数组的长度 colNameArrayParam.length} colNum 
 */
function initMerge(colNum) {
    colNumArray = [], colPosIndexArray = [];
    for (let i = 0; i < colNum; i++) {
        colNumArray.push([1]); // 初始化第一行 
        colPosIndexArray.push([0]); 
    }
}

/**
 * 合并处理
 * @param {需要合并的字段名数组} colNameArrayParam 
 * @param {用来合并的源数据} dataParam 
 */
const colNumberMergeCount = function (colNameArrayParam, dataParam) {
    for (let i = 0; i < dataParam.length; i++) { //遍历每一条数据
        if (i == 0) {
            initMerge(colNameArrayParam.length);  //初始化第一行
        } else {
            let isAnd = true; // 记录必须前面的列合并才能合并当前列 如第3列合并  必须第1、2列也要合并
            for (let j = 0; j < colNameArrayParam.length; j++) { //遍历需要合并的列，计算合并的行数
                if (isAnd && dataParam[i][colNameArrayParam[j]] === dataParam[i - 1][colNameArrayParam[j]]) {  //判断是否和上一列内容相同
                    colNumArray[j][colPosIndexArray[j]] += 1; // 如相同 则+1 表示合并的行数，
                    colNumArray[j].push(0);  // 再push新的一行为0，表示这一行被合并
                    isAnd = true;  // 记录并且关系
                } else {
                    isAnd = false; // 如果不相同，刚之后的全都不合并
                    colNumArray[j].push(1);  //push新行 默认为1 
                    colPosIndexArray[j] = i; //更新计算合并行数的位置，如 从第1行开始计算，如果一直相同位置不变，后面一直push新行为0，反之，没有合并刚更新位置+1，从下一行开始计算合并
                }
            }
        }
    }
    console.log(colNumArray)
    return colNumArray;  //返回合并处理后的数据
}

/**
 * 合并函数  绑定表格上的span-method函数 (暂未使用，该函数使用的页面内部实现)
 * @param {span-method 回调参数} param0 
 */
function spanMethod({ row, column, rowIndex, columnIndex }) {
    const _row = colNumArray[columnIndex][rowIndex];
    const _col = _row > 0 ? 1 : 0; //如果被合并了_row=0则它这个列需要取消
    return {
        rowspan: _row,
        colspan: _col
    };
}
export default colNumberMergeCount;
```

引入到组件中: 
```
import mergeCell from "@/util/mergeCell"; 
```

定义一个函数处理接受原始data数据以及需要合并的列组成的数组,返回处理后数组(包含合并的单元格信息{rowspan,colsapn})：  

```
// this.colNameArray 需要合并的列 data列表中的数据
merge(data) {
  this.mergeArray = mergeCell(this.colNameArray, data);
},
```
`this.mergeArray` 就是之前声明用来接收处理合并结果，数组中保存了单元格的合并规则。 

之后编写`span-method`方法 ： 

```
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
```


绑定到`<el-table> span-method` 属性中：

```
<el-table :data="tableData" stripe border :span-method="arraySpanMethod">
```

最后一步调用`meger`函数: 

```

created(){
  this.merge(this.tableData);
},

```


OK！ 完成。

如果要实现两列之前的合并，则需要修改一下mergeCell.js 。 可以多扩展一个函数，一并封闭成通用的库。之后所有的项目都可以使用。 

代码 ： [git地址](https://github.com/Rewa-Fang/vue-test/blob/master/src/components/mergeTable.vue)

