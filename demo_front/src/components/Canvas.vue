<template>
  <div class="BackContainer" >
      <div class="CanvasRegion">
      </div>
    <div class="BoardRegion">
      <el-container>
        <el-input
          suffix-icon="el-icon-search"
          placeholder="请输入股票代码或简称"
          v-model="searchInput"
          clearable>
        </el-input>
        <el-button round @click="searchName">搜索股票</el-button>
      </el-container>
      <el-tabs type="border-card" v-model="activeTab" >
        <el-tab-pane label="搜索结果" name="搜索结果">
            <el-card v-for="item in searchRes[CurrentSearchPage-1]"
                     :key="item.id"
                     style="font-size: 5px">
              <el-radio-group v-model="ChosenRadio" @change="search('stock',ChosenRadio)">
                <el-radio  :label="item.id">{{item.id}} {{item.code}}
                </el-radio>
              </el-radio-group>
              <div v-bind:class="{greenPrice:item.isGreen, redPrice:item.isRed}">{{item.price}} {{item.pricelimit}}</div>
              <div >
                {{item.chinesename}}
              </div>
            </el-card>
          <div>
            <el-pagination
              small
              @current-change="changeCurrentSearchPage"
              :hide-on-single-page="SwitchValue"
              :page-count="SearchPageNum"
              :current-page="CurrentSearchPage"
              layout="prev, pager, next">
            </el-pagination>
          </div>

        </el-tab-pane>
        <el-tab-pane label="信息详情" name="信息详情">
          <el-card class="box-card" >
            <div>{{stockAttr.shorthand}} {{stockAttr.code}}</div>
            <div v-bind:class="{greenPrice:isGreen, redPrice:isRed}">{{stockAttr.price}} {{stockAttr.pricelimit}}</div>
            <div>{{stockAttr.openprice}} {{stockAttr.closeprice}}</div>
            <div>{{stockAttr.topprice}} {{stockAttr.bottomprice}}</div>
            <div v-for="item in nodeAttr">
              {{item.type}}:{{item.msg}}
            </div>
            <a :href="tabAttr.website" target="_blank" class="buttonText">{{tabAttr.webTag}}</a>
            <el-tabs >
              <el-tab-pane label="主要业务">{{tabAttr.business}}</el-tab-pane>
              <el-tab-pane label="公司介绍">{{tabAttr.introduction}}</el-tab-pane>
            </el-tabs>
          </el-card>
        </el-tab-pane>
        <el-tab-pane label="股价走势">
          <KChart ref="kChart"> </KChart>
        </el-tab-pane>
        <el-tab-pane label="相关股票" name="相关股票">
          <el-table style="width: 100%; font-size: 6px"
                    @row-click="clickTab"
                    :data="RelativeNodeList.slice((CurrentRelativePage-1)*RelativePageSize,CurrentRelativePage*RelativePageSize)">  //对数据请求的处理，最为重要的一句话
          <el-table-column type="index" width="50">
          </el-table-column>
          <el-table-column label="股票简称" prop="shorthand" width="100">
          </el-table-column>
          <el-table-column label="股票代码" prop="stockcode" width="100">
          </el-table-column>
          </el-table>
          <el-pagination
          small
          @current-change="changeCurrentRelativePage"
          :current-page="CurrentRelativePage"
          :page-size="RelativePageSize"
          layout="prev, pager, next"
          :total="RelativeNodeList.length">
          </el-pagination>
        </el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<script>
  import * as d3 from 'd3'
  import KChart from "./KChart";

  export default {
  name: 'Canvas',
  components:{KChart},
  data () {
    // node结构：{"id": "广东鸿图", "group": 1,"type":"stock","code":"002101","nodeAttr":"广东鸿图科技股份有限公司"},
    return {
      width:750,
      height:460,
      isGreen:true,
      isRed:false,
      SwitchValue:true,
      SearchPageNum:1,
      CurrentSearchPage:1,
      CurrentRelativePage:1,
      RelativePageNum:1,
      RelativePageSize:10,
      RelativeNodeList:[],
      price:{},
      colorList:["#57C7E3","#ECB5C9","#FFC454","#4C8EDA","#D9C8AE","#C990C0","#8DCC93","#57C7E3"],
      //关系图谱的中心节点颜色、股票、地域、主板、行业、概念、子概念
      ChosenRadio:"",
      searchRes:[],
      searchInput:"",
      currentType:"",
      currentMsg:"",
      g:null,
      mainSvg:null,
      stockAttr: {},
      tabAttr:{},
      blank:[],
      nodes:[],//使用全局变量以更新画布
      links:[],
      nodeNameText:[],
      linkRelation:[],
      nodeAttr:[],
      activeTab:"",
      simulation:null,
      Graph4Update:{
        "nodes":[{"id": "广东鸿图", "group": 1,"type":"stock","code":"002101","nodeAttr":"广东鸿图科技股份有限公司"},
          {"id": "文化长城", "group": 1,"type":"stock","code":"300089","nodeAttr":"广东文化长城集团股份有限公司"},
          {"id": "白云山", "group": 1,"type":"stock","code":"600332","nodeAttr":"广州白云山制药股份有限公司"},
          {"id": "东方财富", "group": 1,"type":"stock","code":"300059","nodeAttr":"东方财富信息股份有限公司"},
          {"id": "中国石油", "group": 1,"type":"stock","code":"601857","nodeAttr":"中国石油天然气集团公司"},],
        "links":[],
        // ]
      },
      stockGraph:{
        "nodes":[
          {"id": "广东鸿图", "group": 1,"type":"stock","code":"002101","nodeAttr":"广东鸿图科技股份有限公司"},
          {"id": "广东", "group": 2,"type":"location"},
          {"id": "主板", "group": 2,"type":"plate"},
          {"id": "汽车配件", "group": 2,"type":"industry"},
          {"id": "所属概念", "group": 2,"type":"conceptionTag"}
        ],
        "links":[
          {"source": "广东鸿图", "target": "汽车配件", "value": 1,"relation":"行业"},
          {"source": "广东鸿图", "target": "广东", "value": 10,"relation":"地域"},
          {"source": "广东鸿图", "target": "主板", "value": 8,"relation":"板块"},
          {"source": "广东鸿图", "target": "所属概念", "value": 8,"relation":"概念"}],
        "conceptionList":["特斯拉概念","国资重组"],
        "conceptionExtension":false,
      },
    }
  },

  mounted(){
    this.initPage();
  },
  methods:{
    initPage(){
      let _this = this
      this.axios.post("/init",{
        limit:25
      })//初始化页面，向后端发送请求
        .then(function(res)
        {
          if(res.status === 200)
          {
            let plateNode = []//存放生成的Node信息
            for(let i = 0;i < res.data.length;i++)
            {
              plateNode.push({"id":res.data[i].platename,"group": 3,"type":"plate"})
            }
            _this.Graph4Update.nodes=plateNode;//将返回节点加入Graph
            _this.initGraph(_this.Graph4Update)//初始化画布
          }
        })
        .catch(function(error)
        {
          console.log(error)
        })},

    initGraph(data){
      let _this = this
      const links = data.links
      const nodes = data.nodes

      _this.simulation = d3.forceSimulation(nodes)
        .force("link", d3.forceLink(links).id(d => d.id).distance(140))
        .force("charge", d3.forceManyBody().strength(-100))
        .force("collide", d3.forceCollide().strength(100))
        .force("center", d3.forceCenter(_this.width / 2, _this.height / 2));

      _this.mainSvg = d3.select(".CanvasRegion")
        .append("svg")
        .attr("viewBox", [0, 0, _this.width, _this.height]);
      _this.g = _this.mainSvg.append("g")
      _this.mainSvg.call(d3.zoom().on("zoom",function(){
        _this.g.attr("transform",d3.event.transform)
      }))

      _this.links = _this.g.append("g")//节点的关系连线
        .attr("marker-end","url(#positiveMarker)")
        .attr("stroke", "#999")
        .attr("stroke-opacity", 0.6)
        .selectAll("path")
        .data(links)
        .join("path")
        .attr("stroke-width", d => Math.sqrt(d.value))
        .attr("id",d => d.source + "_" + d.relation+"_"+d.target);

      _this.nodes = _this.g.append("g")//节点声明
        .attr("stroke", "#ffffff")
        .attr("stroke-width", 1.5)
        .selectAll("circle")
        .data(nodes)
        .join("circle")
        .attr("r", 30)
        .attr("fill", function (d) {
          return _this.colorList[d.group%10];
        })
        .on("mouseover", function(d) {
          d3.select(this).style("stroke", "orange");
          if (d.type == "stock")
          {
            _this.axios({
              method:"post",
              url:"/price",
              params: {//请求提交给后端的参数,用params的key-value形式
                "stockcode":d.code,
              }
            }).then(function(res) {
              _this.stockAttr = {"shorthand":d.id,"code":d.code,"price":"股价:"+res.data[2],
                "openprice":"开盘价:"+res.data[0],"closeprice":"收盘价:"+res.data[1],"pricelimit":"涨跌幅:"+res.data[5]+"%",
                "topprice":"最高价:"+res.data[3],"bottomprice":"最低价:"+res.data[4]};
              _this.changeColor(res.data[5].substr(0,1))
              _this.nodeAttr = d.nodeAttr;
              _this.tabAttr = {"website":d.website,
                "webTag":"公司网址："+d.website,
              "business":d.business,
                "introduction":d.introduction}
              _this.activeTab = "信息详情"
            });
          }
        })
        .on("mouseout", function(d) {
          d3.select(this).style("stroke", "white");
          // _this.stockAttr = ""
          // _this.nodeAttr = "";
        })
        .call(_this.drag(_this.simulation))
        .on("click",d=>//鼠标监听
        {
          _this.search(d.type,d.id)
        })
      ;

      _this.nodes.append("title")
        .text(d => d.id)

      _this.linkRelation = _this.g.append("g")//连接的文字显示
      .selectAll("text")
      .data(links)
      .join("text")
      .style("text-anchor", "middle")
      .style("fill","#71d7ff")
      .style("font-size","15")
      .append("textPath")
      .attr("class","linkRelation")
      .attr(
        "xlink:href", d => "#" + d.source +"_" + d.relation + "_" +d.target
      )
      .attr("startOffset" , "50%")
      .text(function (d)
      {
        return d.relation
      })

      _this.nodeNameText = _this.g.append("g")//节点的文字显示
        .selectAll("text")
        .data(nodes)
        .join("text")
        .attr("fill","white")
        .attr("font-weight","bold")
        .classed("nodeName",true)
        .attr("font-size",15)
        .attr("class","nodeName")
        .text(function (d)
        {
          return d.id
        })
        .attr("dx",function(){
          return this.getBoundingClientRect().width/2*(-1)+8;
        })
        .attr("dy",45)
      ;

      const positiveMarker = _this.mainSvg.append("marker")//连接线的箭头效果，正反方向
        .attr("id","positiveMarker")
        .attr("orient","auto")
        .attr("stroke-width",2)
        .attr("markerUnits","strokeWidth")
        .attr("markerUnits", "userSpaceOnUse")
        .attr("viewBox", "0 -5 10 10 ")
        .attr("refX",35)
        .attr("refY",0)
        .attr("markerWidth",12)
        .attr("markerHeight",12)
        .append("path")
        .attr("d", "M 0 -5 L 10 0 L 0 5")
        .attr("fill", "#999")
        .attr("stroke-opacity", 0.6)

      const negativeMarker = _this.mainSvg.append("marker")
        .attr("id","negativeMarker")
        .attr("orient","auto")
        .attr("stroke-width",2)
        .attr("markerUnits","strokeWidth")
        .attr("markerUnits", "userSpaceOnUse")
        .attr("viewBox", "0 -5 10 10")
        .attr("refX",105)
        .attr("refY",0)
        .attr("markerWidth",12)
        .attr("markerHeight",12)
        .append("path")
        .attr("d", "M 10 -5 L 0 0 L 10 5")
        .attr("fill", "#999")
        .attr("stroke-opacity", 0.6)

      _this.simulation.on("tick", () => {
        _this.links
        .attr("d", function(d)
        {
        if(d.source.x<d.target.x)
        {
          return "M " + d.source.x+" " + d.source.y + " L " + d.target.x + " " + d.target.y
        }
        else{
          return "M " + d.target.x + " " + d.target.y +" L " + d.source.x+" " + d.source.y;
        }
        })
        .attr("marker-end",function (d) {
            if (d.source.x < d.target.x)
            {
              return "url(#positiveMarker)"
            }
            else
            {
              return "url(#negativeMarker)"
            }
          })

        _this.nodes
          .attr("cx", d => d.x)
          .attr("cy", d => d.y);

        _this.nodeNameText
          .attr("x",d => d.x)
          .attr("y",d => d.y);

      });
    },

    drag(simulation){
      function dragstarted(d) {
        if (!d3.event.active) simulation.alphaTarget(0.3).restart();
        d.fx = d.x;
        d.fy = d.y;
      }

      function dragged(d) {
        d.fx = d3.event.x;
        d.fy = d3.event.y;
      }

      function dragended(d) {
        if (!d3.event.active) simulation.alphaTarget(0);
        d.fx = null;
        d.fy = null;
      }

      return d3.drag()
        .on("start", dragstarted)
        .on("drag", dragged)
        .on("end", dragended);
    },

    search(nodeType,nodeMsg)//点击/查询事件，向后端请求数据（d.type,d.id）也就是mounted初始化请求里的格式
    {
      let _this = this
      let extensionLimit = 8
      if (nodeType == "stock")
        //点击节点为股票，输入股票简称（nodeMsg），返回股票信息node
      {
        this.axios({
          method:"post",
          url:"/" + nodeType,//这个是请求后端的哪个url的名称，跟后端的要一样
          params: {//请求提交给后端的参数,用params的key-value形式
            shorthand:nodeMsg,//提交的股票简称
          }
        })
          .then(function(response)
          {
            if(response.status === 200)
            {
              let stockNode = []
              let stockLink = []
              let res=response.data;
              console.log(res);
              _this.stockGraph.conceptionList = []
              for(let i =0;i<res.concepts.length;i++)
              {
                _this.stockGraph.conceptionList.push(res.concepts[i].conceptname)//向二级概念列表写入概念
              }
              let resAttr = []//属性列表
              resAttr.push({"type":"公司全称","msg":res.chinesename})
              resAttr.push({"type":"英文名称","msg":res.englishname})
              resAttr.push({"type":"公司董事长","msg":res.chairman})
              resAttr.push({"type":"雇员数量","msg":res.employee+"人"})
              resAttr.push({"type":"交易所","msg":res.market})
              resAttr.push({"type":"注册资本","msg":res.regcapital+"千元"})
              resAttr.push({"type":"总股本","msg":res.profit+"亿元"})
              resAttr.push({"type":"公司地址","msg":res.office})
              _this.stockGraph.conceptionExtension = false//概念展开标记置为false
              stockNode.push({"id": res.shorthand, "group": 1,"type":"stock","code":res.stockcode.substr(2,8),
                "website":'http://'+res.website,"business":res.business,
                "introduction":res.introduction,"nodeAttr":resAttr})//信息写入临时节点
              stockNode.push({"id": res.locations[0].provinces, "group": 2,"type":"location"})
              stockNode.push({"id": res.plates[0].platename, "group": 3,"type":"plate"})
              stockNode.push({"id": "所属概念", "group": 5,"type":"conceptionTag"})
              let StockIndustry = res.industries
              for(let i =0;i<StockIndustry.length;i++)
              {
                stockNode.push({"id": StockIndustry[i].name, "group": 4,"type":"industry"})
                stockLink.push({"source": res.shorthand, "target": StockIndustry[i].name, "value": 3,"relation":"行业"+(i+1)})//写入关系
              }
              _this.stockGraph.nodes = stockNode

              stockLink.push({"source": res.shorthand, "target": res.locations[0].provinces, "value": 3,"relation":"地域"})//写入关系
              stockLink.push({"source": res.shorthand, "target": res.plates[0].platename, "value": 3,"relation":"板块"})
              stockLink.push({"source": res.shorthand, "target": "所属概念", "value": 3,"relation":"概念"})
              _this.stockGraph.links = stockLink
              _this.updateGraph(_this.stockGraph)
              _this.axios({
                method:"post",
                url:"/kline",
                params: {//请求提交给后端的参数,用params的key-value形式
                  stockcode:res.stockcode.substr(2,8),//股票代码
                }
                })
                .then(function(KlineRes)
                {
                  let dataset = KlineRes.data
                  if(KlineRes.status === 200)
                  {
                    let LocalStock = res.shorthand+" "+res.stockcode.substr(2,8)
                    _this.$refs.kChart.updateChart(dataset,LocalStock)
                    // _this.activeTab = "股价走势"
                  }
                })
                .catch(function(error)
                {
                  console.log(error)
                })
            }
          })
          .catch(function(error)
          {
            console.log(error)
          })

      }
      else if (nodeType == "conceptionTag")
        //展开概念信息
      {
        let limit = Math.min(_this.stockGraph.conceptionList.length,extensionLimit)
        if (limit==0)
        {
          _this.noConceptionMsg()
        }
        else if (_this.stockGraph.conceptionExtension)//概念已展开，弹出所有概念节点和关系，并将概念展开记为false
        {
          for(let i=0;i<limit;i++)
          {
            _this.stockGraph.links.pop()
            _this.stockGraph.nodes.pop()
          }
          _this.stockGraph.conceptionExtension = false
        }
        else//概念未展开，添加节点和关系
        {
          for(let i=0;i<limit;i++)
          {
            _this.stockGraph.nodes.push({"id":_this.stockGraph.conceptionList[i], "group": 6,"type":"conception"})
            _this.stockGraph.links.push({"source": "所属概念", "target": _this.stockGraph.conceptionList[i],
              "value": 3,"relation":" ",})
          }
          _this.stockGraph.conceptionExtension = true
        }
        _this.updateGraph(_this.stockGraph)
      }
      else//根据关系查询股票,nodeMsg为关系名称,nodeType为关系类型（plate/location/conception）
      {
        _this.axios({
          method:"post",
          url:"/" + nodeType+"1",//这个是请求后端的哪个url的名称，跟后端的要一样
          params: {//请求提交给后端的参数,用params的key-value形式
            nodeMsg:nodeMsg,//提交关系节点名称
          }
        })
        .then(function(res)
        {
          _this.currentType = nodeType
          _this.currentMsg = nodeMsg
          if(res.status === 200) {
            _this.searchRelativeStocks(1)
            _this.RelativeNodeList=res.data
            _this.activeTab = "相关股票"
            _this.CurrentRelativePage = 1
          }
        })
        .catch(function(error)
        {
          console.log(error)
        })
      }
      },

    updateGraph(data){
      const _this = this
      let links = data.links
      let nodes = data.nodes

      _this.mainSvg.select("g").remove()
      _this.g = _this.mainSvg.append("g")

      _this.links = _this.g.append("g")//节点的关系连线
        .attr("marker-end","url(#positiveMarker)")
        .attr("stroke", "#999")
        .attr("stroke-opacity", 0.6)
        .selectAll("path")
        .data(links)
        .join("path")
        .attr("stroke-width", d => Math.sqrt(d.value))
        .attr("id",d => d.source + "_" + d.relation+"_"+d.target);

      _this.nodes = _this.g.append("g")//节点声明
        .attr("stroke", "#ffffff")
        .attr("stroke-width", 1.5)
        .selectAll("circle")
        .data(nodes)
        .join("circle")
        .attr("r", 30)
        .attr("fill", function (d) {
          return _this.colorList[d.group%10];
        })
        .on("mouseover", function(d) {
          d3.select(this).style("stroke", "orange");
          if (d.type == "stock")
          {
            _this.axios({
              method:"post",
              url:"/price",
              params: {//请求提交给后端的参数,用params的key-value形式
                "stockcode":d.code,
              }
              // endresult[0] = openprice;//今日开盘价
              // endresult[1] = closeprice;//今日收盘价
              // endresult[2] = price;//当前价格
              // endresult[3] = topprice;//今日最高价
              // endresult[4] = bottomprice;//今日最低价
              // endresult[5] = pricelimit;//涨跌幅
            }).then(function(res) {
              _this.stockAttr = {"shorthand":d.id,"code":d.code,"price":"股价:"+res.data[2],
                "openprice":"开盘价:"+res.data[0],"closeprice":"收盘价:"+res.data[1],"pricelimit":"涨跌幅:"+res.data[5]+"%",
                "topprice":"最高价:"+res.data[3],"bottomprice":"最低价:"+res.data[4]};
              _this.changeColor(res.data[5].substr(0,1))
              _this.nodeAttr = d.nodeAttr;
              _this.tabAttr = {"website":d.website,
                "webTag":"公司网址："+d.website,
                "business":d.business,
                "introduction":d.introduction}
              _this.activeTab = "信息详情";
            });
          }
        })
        .on("mouseout", function(d) {
          d3.select(this).style("stroke", "white");
          // _this.stockAttr = "";
          // _this.nodeAttr = "";
        })
        .call(_this.drag(_this.simulation))
        .on("click",d=>//鼠标监听
        {
          _this.search(d.type,d.id)
        })
      ;
      _this.nodes.append("title")
        .text(d => d.id)

      _this.linkRelation = _this.g.append("g")//连接的文字显示
        .selectAll("text")
        .data(links)
        .join("text")
        .style("text-anchor", "middle")
        .style("fill","#71d7ff")
        .style("font-size","15")
        .append("textPath")
        .attr("class","linkRelation")
        .attr(
          "xlink:href", d => "#" + d.source +"_" + d.relation + "_" +d.target
        )
        .attr("startOffset" , "50%")
        .text(function (d)
        {
          return d.relation
        })

      _this.nodeNameText = _this.g.append("g")//节点的文字显示
        .selectAll("text")
        .data(nodes)
        .join("text")
        .attr("fill","white")
        .classed("nodeName",true)
        // .attr("fill","white")
        .attr("font-size",15)
        .attr("font-weight","bold")
        .attr("class","nodeName")
        .text(function (d)
        {
          return d.id;
        })
        .attr("dx",function(){
          // -10;
          return this.getBoundingClientRect().width/2*(-1)+8;
        })
        .attr("dy",45);


      _this.simulation.nodes(nodes)
      _this.simulation.force("link").links(links)
      _this.simulation
        // .force("link", d3.forceLink(links).id(d => d.id).distance(130))
        .force("charge", d3.forceManyBody().strength(-400))
        .force("collide", d3.forceCollide().strength(100))
      //   .force("center", d3.forceCenter(_this.width / 2, _this.height / 2));
      _this.simulation.alpha(1).restart()

    },

    searchName() {//搜索按钮绑定函数，后端交互
      let _this = this
      this.axios({
        method:"post",
        url:"/queryByName",//这个是请求后端的哪个url的名称，跟后端的要一样
        params: {//请求提交给后端的参数,用params的key-value形式
          chinesename: _this.searchInput,//提交的数据内容
        }
      })
        .then(function(response)
        {
          if(response.status === 200)
          {
            _this.SwitchValue = false
            let PageSize = 5
            _this.SearchPageNum = Math.ceil(response.data.length/PageSize)//页码数
            _this.searchRes=[]//分页存放搜索结果的存储数组
            for(let i = 0;i < _this.SearchPageNum;i++)
            {
              let stockNode = []//存放生成的Node信息
              for(let j = 0;j < PageSize;j++)
              {
                let n = i*PageSize+j//将第n个节点的数据分别加入每页中
                if(n>=response.data.length)
                {
                  break
                }
                let code = response.data[n].stockcode.substr(2,8)
                _this.axios({
                  method:"post",
                  url:"/price",
                  params: {
                    "stockcode":code,
                  }
                }).then(function(res)
                {
                  _this.changeColor(res.data[5].substr(0,1))//传入涨跌幅的符号
                  stockNode.push({"id": response.data[n].shorthand, "group": 1,"type":"stock",
                    "code":code,"chinesename":response.data[n].chinesename,"price":"股价:"+res.data[2],
                    "pricelimit":"涨跌幅:"+res.data[5]+"%",
                    // "openprice":"开盘价:"+res.data[0],"closeprice":"收盘价:"+res.data[1],
                    // "topprice":"最高价:"+res.data[3],"bottomprice":"最低价:"+res.data[4],
                    "isRed":_this.isRed,"isGreen":_this.isGreen
                  })
                })
              }
              _this.searchRes.push(stockNode)//将本页结果添加到数组中去
            }
            _this.activeTab = "搜索结果"
            _this.CurrentSearchPage = 1
          }
        })
        .catch(function(error)
        {
          console.log(error)
        })
    },

    searchRelativeStocks(page)
    {
      let _this = this
      let nodeType = _this.currentType
      let nodeMsg = _this.currentMsg
      this.axios({
        method:"post",
        url:"/" + nodeType,//这个是请求后端的哪个url的名称，跟后端的要一样
        params: {//请求提交给后端的参数,用params的key-value形式
          nodeMsg:nodeMsg,//提交关系节点名称
          page:page
        }
      })
      .then(function(response)
      {
        if(response.status === 200)
        {
          let relationNode = []//存放获取的node
          //stockNode.push({"id": res.shorthand, "group": 1,"type":"stock","code":res.code,"nodeAttr":res.chesename})//信息写入临时节
          for(let i = 0;i <response.data.length;i++)
          {
            let resAttr = []//属性列表
            resAttr.push({"type":"公司全称","msg":response.data[i].chinesename})
            resAttr.push({"type":"英文名称","msg":response.data[i].englishname})
            resAttr.push({"type":"公司董事长","msg":response.data[i].chairman})
            resAttr.push({"type":"雇员数量","msg":response.data[i].employee+"人"})
            resAttr.push({"type":"交易所","msg":response.data[i].market})
            resAttr.push({"type":"注册资本","msg":response.data[i].regcapital+"千元"})
            resAttr.push({"type":"总股本","msg":response.data[i].profit+"亿元"})
            resAttr.push({"type":"公司地址","msg":response.data[i].office})
            relationNode.push({"id": response.data[i].shorthand, "group": 1,"type":"stock",
              "website":'http://'+response.data[i].website,"business":response.data[i].business,
              "introduction":response.data[i].introduction,
              "code":response.data[i].stockcode.substr(2,8),"nodeAttr":resAttr})
          }
          _this.Graph4Update.nodes=relationNode;//将返回节点加入Graph

          _this.Graph4Update.nodes.push({"id": nodeMsg, "group": 0,"type":nodeType})//增放中心节点，即点击的节点
          let relationLink = []//存放生成的link信息
          for(let i = 0;i <response.data.length;i++)
          {
            relationLink.push({"source": response.data[i].shorthand, "target": nodeMsg, "value": 3,"relation":" "})
          }
          _this.Graph4Update.links = relationLink//将关系link加入Graph
          _this.updateGraph(_this.Graph4Update)
        }
      })
      .catch(function(error)
      {
        console.log(error)
      })
    },

    changeColor(sign4price){
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

    noConceptionMsg() {
      this.$message('未找到关联的概念信息');
    },

    changeCurrentSearchPage(val){
      let _this = this
      _this.CurrentSearchPage = val
    },

    changeCurrentRelativePage(val){
      let _this = this
      console.log(val)
      _this.CurrentRelativePage = val
      _this.searchRelativeStocks(val)
    },

    clickTab(d)
    {
      let _this = this
      // this.search("stock",d.shorthand)
      this.axios({
        method:"post",
        url:"/" + "stock",//这个是请求后端的哪个url的名称，跟后端的要一样
        params: {//请求提交给后端的参数,用params的key-value形式
          shorthand:d.shorthand,//提交的股票简称
        }
        })
        .then(function(response)
        {
          if(response.status === 200)
          {
            let stockNode = []
            let stockLink = []
            let res=response.data;
            _this.stockGraph.conceptionList = []
            for(let i =0;i<res.concepts.length;i++)
            {
              _this.stockGraph.conceptionList.push(res.concepts[i].conceptname)//向二级概念列表写入概念
            }
            let resAttr = []//属性列表
            resAttr.push({"type":"公司全称","msg":res.chinesename})
            resAttr.push({"type":"英文名称","msg":res.englishname})
            resAttr.push({"type":"公司董事长","msg":res.chairman})
            resAttr.push({"type":"雇员数量","msg":res.employee+"人"})
            resAttr.push({"type":"交易所","msg":res.market})
            resAttr.push({"type":"注册资本","msg":res.regcapital+"千元"})
            resAttr.push({"type":"总股本","msg":res.profit+"亿元"})
            resAttr.push({"type":"公司地址","msg":res.office})
            _this.tabAttr = {"website":res.website,
              "webTag":"公司网址："+res.website,
              "business":res.business,
              "introduction":res.introduction}
            _this.nodeAttr = resAttr
            _this.stockGraph.conceptionExtension = false//概念展开标记置为false
            stockNode.push({"id": res.shorthand, "group": 1,"type":"stock","code":res.stockcode.substr(2,8),
              "website":'http://'+res.website,"business":res.business,
              "introduction":res.introduction,"nodeAttr":resAttr})//信息写入临时节点
            stockNode.push({"id": res.locations[0].provinces, "group": 2,"type":"location"})
            stockNode.push({"id": res.plates[0].platename, "group": 3,"type":"plate"})
            stockNode.push({"id": "所属概念", "group": 5,"type":"conceptionTag"})
            let StockIndustry = res.industries
            for(let i =0;i<StockIndustry.length;i++)
            {
              stockNode.push({"id": StockIndustry[i].name, "group": 4,"type":"industry"})
              stockLink.push({"source": res.shorthand, "target": StockIndustry[i].name, "value": 3,"relation":"行业"+(i+1)})//写入关系
            }
            _this.stockGraph.nodes = stockNode

            stockLink.push({"source": res.shorthand, "target": res.locations[0].provinces, "value": 3,"relation":"地域"})//写入关系
            stockLink.push({"source": res.shorthand, "target": res.plates[0].platename, "value": 3,"relation":"板块"})
            stockLink.push({"source": res.shorthand, "target": "所属概念", "value": 3,"relation":"概念"})
            _this.stockGraph.links = stockLink
            console.log(_this.stockGraph)
            _this.updateGraph(_this.stockGraph)
            _this.axios({
              method:"post",
              url:"/kline",
              params: {//请求提交给后端的参数,用params的key-value形式
                stockcode:res.stockcode.substr(2,8),//股票代码
              }
            })
            .then(function(KlineRes)
            {
              let dataset = KlineRes.data
              if(KlineRes.status === 200)
              {
                let LocalStock = res.shorthand+" "+res.stockcode.substr(2,8)
                _this.$refs.kChart.updateChart(dataset,LocalStock)
                _this.axios({
                  method:"post",
                  url:"/price",
                  params: {
                    "stockcode":res.stockcode.substr(2,8),
                  }
                }).then(function(Price) {
                  _this.stockAttr = {"shorthand":res.shorthand,"code":res.code,"price":"股价:"+Price.data[2],
                    "openprice":"开盘价:"+Price.data[0],"closeprice":"收盘价:"+Price.data[1],"pricelimit":"涨跌幅:"+Price.data[5]+"%",
                    "topprice":"最高价:"+Price.data[3],"bottomprice":"最低价:"+Price.data[4]};
                  _this.changeColor(Price.data[5].substr(0,1))
                })
              }
            })
            .catch(function(error)
            {
              console.log(error)
            })

          }
        })
        .catch(function(error)
        {
          console.log(error)
        })
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  /*display: inline-block;*/
  /*margin: 0 10px;*/
}
a {
  color: #42b983;
}
  .BackContainer{
    display: flex;
    justify-content: space-between;
    /*overflow: hidden;*/
  }
  .CanvasRegion
  {
    width:72%;
    height: 100%;
    flex-shrink:0;
    border: 1px solid #2c3e50;
    border-radius: 8px;
    margin-left:auto;
    margin-right:auto;
    /*background: #154360 repeating-linear-gradient(30deg,*/
    /*hsla(0,0%,100%,.1),hsla(0,0%,100%,.1) 15px,*/
    /*transparent 0, transparent 30px);*/
    font-family: "Exo", sans-serif;
    color: #fff;
    background: linear-gradient(-45deg, #7d4664, #66658b, #196c8b, #186d59);
    animation: gradientBG 15s ease infinite;
    background-size: 400% 400%;
  }
  .BoardRegion
  {
    width:25%;
  }
  .box-card{
    text-align:left;
  }
  .greenPrice{
    color:forestgreen;
  }
  .redPrice{
    color:red;
  }
  .GradientColor{

  }
  @keyframes gradientBG {
    0% {
      background-position: 0% 50%;
    }
    50% {
      background-position: 100% 50%;
    }
    100% {
      background-position: 0% 50%;
    }
  }
</style>
