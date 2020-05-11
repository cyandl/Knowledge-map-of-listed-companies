<template>
  <div >
    <el-container>
      <el-input
        suffix-icon="el-icon-search"
        placeholder="请输入股票代码或简称"
        v-model="searchInput"
        clearable>
      </el-input>
      <el-button round @click="searchName">搜索股票</el-button>
    </el-container>
    <div>
      <el-card v-for="item in searchRes">
        <el-radio-group v-model="ChosenRadio" @change="searchStock(ChosenRadio)">
              <el-radio  :label="item.id">{{item.id}} {{item.code}}
                <span v-bind:class="{greenPrice:isGreen, redPrice:isRed}">{{item.price}}</span>
              </el-radio>
        </el-radio-group>
        <div style="font-size: 5px">
        {{item.chinesename}}
        </div>
      </el-card>
    </div>
  </div>

</template>

<script>
  export default {
    name: "MsgBoard",
    data(){//这个好像是返回给后端的数据
      return{
        isGreen:true,
        isRed:false,
        searchRes:[],
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
              let resLimit = Math.min(response.data.length,4)
              for(let i = 0;i <resLimit ;i++)
              {
                let code = response.data[i].stockcode.substr(2,8)
                _this.axios({
                  method:"post",
                  url:"/price",
                  params: {//请求提交给后端的参数,用params的key-value形式
                    "stockcode":code,
                  }
                }).then(function(res)
                {
                  stockNode.push({"id": response.data[i].shorthand, "group": 1,"type":"stock",
                    "code":code,"chinesename":response.data[i].chinesename,"price":"股价:"+res.data[2],
                    "openprice":"开盘价:"+res.data[0],"closeprice":"收盘价:"+res.data[1],"pricelimit":"涨跌幅:"+res.data[5],
                    "topprice":"最高价:"+res.data[3],"bottomprice":"最低价:"+res.data[4],"isRed":true,"isGreen":false
                  })
                  _this.changeColor(res.data[5].substr(0,1))//传入涨跌幅的符号
                })
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
        this.$parent.search("stock", code);//调用Canvas.vue的search函数，更新画布
      },

      changeColor(sign4price){//根据涨幅还是跌幅来确定股价颜色
        let _this = this
        if(sign4price=="+"){
          _this.isRed = true;
          _this.isGreen = false;
        }
        else{
          _this.isRed = false;
          _this.isGreen = true;
        }
      },

    }
  }

</script>

<style scoped>
  .greenPrice{
    color:forestgreen;
  }
  .redPrice{
    color:red;
  }
</style>
