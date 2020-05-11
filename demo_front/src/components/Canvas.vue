<template>
  <div class="BackContainer" >
    <div class="CanvasRegion" >
    </div>
    <div class="BoardRegion">
    <MsgBoard></MsgBoard>
    <el-card class="box-card" >
    <div>{{stockAttr.shorthand}} {{stockAttr.code}}
      <span v-bind:class="{greenPrice:isGreen, redPrice:isRed}">{{stockAttr.price}}</span>
    </div>
      <div v-for="item in nodeAttr">
        {{item.type}}:{{item.msg}}
      </div>
    </el-card>
    </div>
  </div>
</template>

<script>
  import * as d3 from 'd3'
  import MsgBoard from "./MsgBoard";
  export default {
  name: 'Canvas',
  components:{MsgBoard},
  data () {
    // node结构：{"id": "广东鸿图", "group": 1,"type":"stock","code":"002101","nodeAttr":"广东鸿图科技股份有限公司"},
    return {
      width:750,
      height:460,
      isGreen:true,
      isRed:false,
      price:{},
      colorList:["#57C7E3","#ECB5C9","#FFC454","#4C8EDA","#D9C8AE","#C990C0","#8DCC93","#57C7E3"],
      //关系图谱的中心节点颜色、股票、地域、主板、行业、概念、子概念
      g:null,
      stockAttr: {},
      blank:[],
      nodes:[],//使用全局变量以更新画布
      links:[],
      nodeNameText:[],
      linkRelation:[],
      nodeAttr:[],
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
          // {"id": "广东鸿图科技股份有限公司", "group": 2,"type":"company"},
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
      gdGraph:{
        "nodes":[
          {"id": "广东鸿图", "group": 1,"type":"stock","code":"002101","nodeAttr":"广东鸿图科技股份有限公司"},
          {"id": "文化长城", "group": 1,"type":"stock","code":"300089","nodeAttr":"广东文化长城集团股份有限公司"},
          {"id": "白云山", "group": 1,"type":"stock","code":"600332","nodeAttr":"广州白云山制药股份有限公司"},
          {"id": "广东", "group": 2,"type":"location"},
        ],
        "links":[
          {"source": "广东", "target": "白云山", "value": 1,"relation":" "},
          {"source": "广东", "target": "广东鸿图", "value": 10,"relation":" "},
          {"source": "广东", "target": "文化长城", "value": 8,"relation":" "},
        ]
      },
      // zbGraph:{
      //   "nodes":[
      //     {"id": "广东鸿图", "group": 1,"type":"stock","code":"002101","nodeAttr":"广东鸿图科技股份有限公司"},
      //     {"id": "文化长城", "group": 1,"type":"stock","code":"300089","nodeAttr":"广东文化长城集团股份有限公司"},
      //     {"id": "白云山", "group": 1,"type":"stock","code":"600332","nodeAttr":"广州白云山制药股份有限公司"},
      //     {"id": "东方财富", "group": 1,"type":"stock","code":"300059","nodeAttr":"东方财富信息股份有限公司"},
      //     {"id": "中国石油", "group": 1,"type":"stock","code":"601857","nodeAttr":"中国石油天然气集团公司"},
      //     {"id": "主板", "group": 2,"type":"plate"},
      //   ],
      //   "links":[
      //     {"source": "主板", "target": "白云山", "value": 1,"relation":" "},
      //     {"source": "主板", "target": "广东鸿图", "value": 10,"relation":" "},
      //     {"source": "主板", "target": "东方财富", "value": 8,"relation":" "},
      //     {"source": "主板", "target": "中国石油", "value": 8,"relation":" "},
      //     {"source": "主板", "target": "文化长城", "value": 8,"relation":" "},
      //   ]
      // },
      // tslGraph:{
      //   "nodes":[
      //     {"id": "广东鸿图", "group": 1,"type":"stock","code":"002101","nodeAttr":"广东鸿图科技股份有限公司"},
      //     {"id": "华特气体","group": 1,"type":"stock","code":"688268","nodeAttr":"广东华特气体股份有限公司"},
      //     {"id": "虹软科技", "group": 1,"type":"stock","code":"688088","nodeAttr":"虹软科技股份有限公司"},
      //     {"id": "特斯拉概念", "group": 2,"type":"conception"},
      //   ],
      //   "links":[
      //     {"source": "特斯拉概念", "target": "华特气体", "value": 1,"relation":" "},
      //     {"source": "特斯拉概念", "target": "广东鸿图", "value": 10,"relation":" "},
      //     {"source": "特斯拉概念", "target": "虹软科技", "value": 8,"relation":" "},
      //   ]
      // }
    }
  },

  mounted(){
    this.initPage();
    // _this.initGraph(_this.Graph4Update)//初始化画布
  },
  methods:{
    initPage(){
      let _this = this
      this.axios.post("/init",{
        limit:25
      })//初始化页面，向后端发送请求
        .then(function(response)
        {
          if(response.status === 200)
          {
            let stockNode = []//存放生成的Node信息
            for(let i = 0;i < response.data.length;i++)
            {
              let resAttr = []//属性列表
              resAttr.push({"type":"公司全称","msg":response.data[i].chinesename})
              resAttr.push({"type":"英文名称","msg":response.data[i].englishname})
              resAttr.push({"type":"交易所","msg":response.data[i].market})
              resAttr.push({"type":"公司市值","msg":response.data[i].regcapital+"元"})
              resAttr.push({"type":"利润","msg":response.data[i].profit+"元"})
              resAttr.push({"type":"雇员数量","msg":response.data[i].employee+"人"})
              stockNode.push({"id": response.data[i].shorthand, "group": 1,"type":"stock",
                "code":response.data[i].stockcode.substr(2,8),"nodeAttr":resAttr})
            }
            _this.Graph4Update.nodes=stockNode;//将返回节点加入Graph
            _this.initGraph(_this.Graph4Update)//初始化画布
          }
        })
        .catch(function(error)
        {
          console.log(error)
        })},

    initGraph(data){
      var _this = this
      const links = data.links
      const nodes = data.nodes
      // const nodes = data.nodes.map(d => Object.create(d));
      // const links = data.links.map(d => Object.create(d));

      _this.simulation = d3.forceSimulation(nodes)
        .force("link", d3.forceLink(links).id(d => d.id).distance(130))
        .force("charge", d3.forceManyBody().strength(-30))
        .force("collide", d3.forceCollide().strength(50))
        .force("center", d3.forceCenter(_this.width / 2, _this.height / 2));

      const svg = d3.select(".CanvasRegion")
        .append("svg")
        .attr("viewBox", [0, 0, _this.width, _this.height]);
      _this.g = svg.append("g")
      svg.call(d3.zoom().on("zoom",function(){
        // svg.attr("transform",d3.event.transform)
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
                "openprice":"开盘价:"+res.data[0],"closeprice":"收盘价:"+res.data[1],"pricelimit":"涨跌幅:"+res.data[5],
                "topprice":"最高价:"+res.data[3],"bottomprice":"最低价:"+res.data[4]};
              _this.changeColor(res.data[5].substr(0,1))
              _this.nodeAttr = d.nodeAttr;
            });
          }
        })
        .on("mouseout", function(d) {
          d3.select(this).style("stroke", "white");
          _this.stockAttr = ""
          _this.nodeAttr = "";
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

      const positiveMarker = svg.append("marker")//连接线的箭头效果，正反方向
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

      const negativeMarker = svg.append("marker")
        .attr("id","negativeMarker")
        .attr("orient","auto")
        .attr("stroke-width",2)
        .attr("markerUnits","strokeWidth")
        .attr("markerUnits", "userSpaceOnUse")
        .attr("viewBox", "0 -5 10 10")
        .attr("refX",95)
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

        // _this.linkRelation
        //   .attr("font-size",15)
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

    // search(nodeType,nodeMsg){
    //   //点击\查询事件，数据写死本地
    //   let tGraph = this.Graph4Update;
    //   if (nodeType == "location")
    //   {
    //     tGraph = this.gdGraph
    //   }
    //   if (nodeType == "stock")
    //   {
    //     tGraph = this.stockGraph
    //   }
    //   if (nodeType == "plate")
    //   {
    //     tGraph = this.zbGraph
    //   }
    //   if (nodeType == "conception")
    //   {
    //     tGraph = this.tslGraph
    //   }
    //   if (nodeType == "conceptionTag")
    //   {
    //     if (this.stockGraph.conceptionExtension)
    //     {
    //       for(let i=0;i<this.stockGraph.conceptionList.length;i++)
    //       {
    //         this.stockGraph.links.pop()
    //         this.stockGraph.nodes.pop()
    //       }
    //       this.stockGraph.conceptionExtension = false
    //     }
    //     else
    //     {
    //       for(let i=0;i<this.stockGraph.conceptionList.length;i++)
    //       {
    //         this.stockGraph.nodes.push({"id":this.stockGraph.conceptionList[i], "group": 2,"type":"conception"})
    //         this.stockGraph.links.push({"source": "所属概念", "target": this.stockGraph.conceptionList[i],
    //           "value": 2,"relation":"概念"+(i+1)},)
    //       }
    //       this.stockGraph.conceptionExtension = true
    //     }
    //     tGraph = this.stockGraph
    //   }
    //   this.updateGraph(tGraph);
    // },

    search(nodeType,nodeMsg)//点击/查询事件，向后端请求数据（d.type,d.id）也就是mounted初始化请求里的格式
    {
      let extensionLimit = 10
      let _this = this
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
            console.log(response.data)
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
              resAttr.push({"type":"交易所","msg":res.market})
              resAttr.push({"type":"公司市值","msg":res.regcapital+"元"})
              resAttr.push({"type":"利润","msg":res.profit+"元"})
              resAttr.push({"type":"雇员数量","msg":res.employee+"人"})

              _this.stockGraph.conceptionExtension = false//概念展开标记置为false
              stockNode.push({"id": res.shorthand, "group": 1,"type":"stock","code":res.stockcode.substr(2,8),"nodeAttr":resAttr})//信息写入临时节点
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
          console.log("extend")
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
        console.log(_this.stockGraph)
        _this.updateGraph(_this.stockGraph)
      }
      else//根据关系查询股票,nodeMsg为关系名称,nodeType为关系类型（plate/location/conception）
      {
        this.axios({
          method:"post",
          url:"/" + nodeType,//这个是请求后端的哪个url的名称，跟后端的要一样
          params: {//请求提交给后端的参数,用params的key-value形式
            nodeMsg:nodeMsg,//提交关系节点名称
          }
        })
          .then(function(response)
          {
            if(response.status === 200)
            {
              console.log(response.data)

              let relationNode = []//存放获取的node
              //stockNode.push({"id": res.shorthand, "group": 1,"type":"stock","code":res.code,"nodeAttr":res.chesename})//信息写入临时节
              for(let i = 0;i <response.data.length;i++)
              {
                let resAttr = []//属性列表
                resAttr.push({"type":"公司全称","msg":response.data[i].chinesename})
                resAttr.push({"type":"英文名称","msg":response.data[i].englishname})
                resAttr.push({"type":"交易所","msg":response.data[i].market})
                resAttr.push({"type":"公司市值","msg":response.data[i].regcapital+"元"})
                resAttr.push({"type":"利润","msg":response.data[i].profit+"元"})
                resAttr.push({"type":"雇员数量","msg":response.data[i].employee+"人"})
                relationNode.push({"id": response.data[i].shorthand, "group": 1,"type":"stock",
                  "code":response.data[i].stockcode.substr(2,8),"nodeAttr":resAttr})
              }
              _this.Graph4Update.nodes=relationNode;//将返回节点加入Graph

              _this.Graph4Update.nodes.push({"id": nodeMsg, "group": 0,"type":nodeType})//增放中心节点，即点击的节点
              let relationLink = []//存放生成的link信息
              for(let i = 0;i <response.data.length;i++)
              {
                relationLink.push({"source": response.data[i].shorthand, "target": nodeMsg, "value": 3,"relation":" "})}
              _this.Graph4Update.links = relationLink//将关系link加入Graph
              _this.updateGraph(_this.Graph4Update)
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
      // const nodes = data.nodes.map(d => Object.create(d));
      // const links = data.links.map(d => Object.create(d));
      // console.log(data)
      let links = data.links
      let nodes = data.nodes

      d3.select("svg").select("g").remove()
      _this.g = d3.select("svg").append("g")

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
                "openprice":"开盘价:"+res.data[0],"closeprice":"收盘价:"+res.data[1],"pricelimit":"涨跌幅:"+res.data[5],
                "topprice":"最高价:"+res.data[3],"bottomprice":"最低价:"+res.data[4]};
              _this.changeColor(res.data[5].substr(0,1))
              _this.nodeAttr = d.nodeAttr;
            });
          }
        })
        .on("mouseout", function(d) {
          d3.select(this).style("stroke", "white");
          _this.stockAttr = "";
          _this.nodeAttr = "";
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
        .force("link", d3.forceLink(links).id(d => d.id).distance(130))
        .force("charge", d3.forceManyBody().strength(-500))
        .force("collide", d3.forceCollide().strength(200))
        .force("center", d3.forceCenter(_this.width / 2, _this.height / 2));
      _this.simulation.alpha(1).restart()

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
    /*margin-top: 40px;*/
    margin-left:auto;
    margin-right:auto;
    background: #154360 repeating-linear-gradient(30deg,
    hsla(0,0%,100%,.1),hsla(0,0%,100%,.1) 15px,
    transparent 0, transparent 30px);
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
</style>
