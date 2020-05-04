<template>
  <div height="100%">
  <el-container height="100%">
    <el-input
      suffix-icon="el-icon-search"
      placeholder="请输入股票代码或简称"
      v-model="input"
      clearable>
    </el-input>
    <el-button round @click="searchName">搜索股票</el-button>
  </el-container>
<!--    <el-radio-group v-model="input">-->
<!--      <el-radio label="线上品牌商赞助"></el-radio>-->
<!--      <el-radio label="线下场地免费"></el-radio>-->
<!--    </el-radio-group>-->
  </div>
</template>

<script>
    export default {
      name: "MsgBoard",
      data(){
        return{
          stockList:[{"id": "广东鸿图", "group": 1,"type":"stock","code":"002101"},
            {"id": "东方财富", "group": 1,"type":"stock","code":"300059"}],
          input:'',
        }
      },
    methods:{
        searchName(data){//搜索按钮绑定函数
          // this.$parent.searchStock(data);
          this.$parent.search("stock",data);//调用Canvas.vue的search函数，更新画布
        }
    }
    }

</script>

<style scoped>
  .el-container {
    height: 100%;
  }
  .el-header{
    background-color: #B3C0D1;
    color: #333;
    text-align: center;
    /*line-height: 5px;*/
    height: 100%;
  }
  .el-footer {
    background-color: #B3C0D1;
    color: #333;
    text-align: center;
    line-height: 50px;
  }

  .el-main {
    background-color: #E9EEF3;
    color: #333;
    text-align: center;
    line-height: 260px;
  }

</style>
