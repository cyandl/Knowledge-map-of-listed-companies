<template>
  <div >
    <el-container >
      <el-input
        suffix-icon="el-icon-search"
        placeholder="请输入股票代码或简称"
        v-model="searchInput"
        clearable>
      </el-input>
      <el-button round @click="searchName">搜索股票</el-button>
    </el-container>

    <div>
      <el-form>
      <el-radio-group v-model="ChosenRadio" @change="searchStock(ChosenRadio)">
        <el-radio v-for="item in searchRes"  :label="item.id">{{item.id}}</el-radio><br>
      </el-radio-group>
      </el-form>
    </div>
  </div>

</template>

<script>
  export default {
    name: "MsgBoard",
    data(){//这个好像是返回给后端的数据
      return{
        searchRes:[],
        // testRes:[{"id": "广东鸿图", "group": 1,"type":"stock","code":"002101"},
        //     {"id": "东方财富", "group": 1,"type":"stock","code":"300059"}],
        searchInput:"",
        ChosenRadio:"",
      }
    },
    methods: {
      // searchName(data) {//搜索按钮绑定函数,本地调试
      //    let _this = this
      //    _this.searchRes = _this.testRes
      // },

      searchName() {//搜索按钮绑定函数，后端交互
        let _this = this
        this.axios({
          method:"post",
          url:"/queryByName",//这个是请求后端的哪个url的名称，跟后端的要一样
          params: {//请求提交给后端的参数,用params的key-value形式
            chinesename: _this.searchInput,//提交的数据内容
            //region:this.form.region
          }
        })
          .then(function(response)
          {
            if(response.status === 200)
            {
              let stockNode = []//存放生成的Node信息
              for(let i = 0;i < response.data.length;i++)
              {
                stockNode.push({"id": response.data[i].shorthand, "group": 1,"type":"stock",
                  "code":response.data[i].stockcode,"nodeAttr":response.data[i].chinesename})
              }
              _this.searchRes = stockNode
            }
          })
          .catch(function(error)
          {
            console.log(error)
          })
      },

      searchStock(code)//选中radio button 更改画布
      {
        // if(code == "002101")//本地版
        // {
        //     this.$parent.search("stock", code);//调用Canvas.vue的search函数，更新画布
        // }
        // else
        // {
        //     this.$parent.search("plate", code);//调用Canvas.vue的search函数，更新画布
        // }
        this.$parent.search("stock", code);//调用Canvas.vue的search函数，更新画布
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
