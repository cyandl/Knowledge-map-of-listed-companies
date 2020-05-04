<template>
  <div style="width: 100%;" >
    <div class="container" >
    </div>
    <MsgBoard style="height: 40%;" ></MsgBoard>
    <div>
      <el-card class="box-card">
        <div>
          {{NodeAttr}}
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
      width:800,
      height:500,
      colorList:["blue","pink","blue","pink","blue","pink","blue","pink","blue","pink"],
      g:null,
      blank:[],
      nodes:[],//使用全局变量以更新画布
      links:[],
      nodeNameText:[],
      linkRelation:[],
      nodeAttr:"",
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
      zbGraph:{
        "nodes":[
          {"id": "广东鸿图", "group": 1,"type":"stock","code":"002101","nodeAttr":"广东鸿图科技股份有限公司"},
          {"id": "文化长城", "group": 1,"type":"stock","code":"300089","nodeAttr":"广东文化长城集团股份有限公司"},
          {"id": "白云山", "group": 1,"type":"stock","code":"600332","nodeAttr":"广州白云山制药股份有限公司"},
          {"id": "东方财富", "group": 1,"type":"stock","code":"300059","nodeAttr":"东方财富信息股份有限公司"},
          {"id": "中国石油", "group": 1,"type":"stock","code":"601857","nodeAttr":"中国石油天然气集团公司"},
          {"id": "主板", "group": 2,"type":"plate"},
        ],
        "links":[
          {"source": "主板", "target": "白云山", "value": 1,"relation":" "},
          {"source": "主板", "target": "广东鸿图", "value": 10,"relation":" "},
          {"source": "主板", "target": "东方财富", "value": 8,"relation":" "},
          {"source": "主板", "target": "中国石油", "value": 8,"relation":" "},
          {"source": "主板", "target": "文化长城", "value": 8,"relation":" "},
        ]
      },
      tslGraph:{
        "nodes":[
          {"id": "广东鸿图", "group": 1,"type":"stock","code":"002101","nodeAttr":"广东鸿图科技股份有限公司"},
          {"id": "华特气体","group": 1,"type":"stock","code":"688268","nodeAttr":"广东华特气体股份有限公司"},
          {"id": "虹软科技", "group": 1,"type":"stock","code":"688088","nodeAttr":"虹软科技股份有限公司"},
          {"id": "特斯拉概念", "group": 2,"type":"conception"},
        ],
        "links":[
          {"source": "特斯拉概念", "target": "华特气体", "value": 1,"relation":" "},
          {"source": "特斯拉概念", "target": "广东鸿图", "value": 10,"relation":" "},
          {"source": "特斯拉概念", "target": "虹软科技", "value": 8,"relation":" "},
        ]
      }
    }
  },

  mounted(){
    this.axios.post("/init",{
      limit:25
    })//向后端发送请求
    .then(function(response)
    {
      if(response.status === 200)
      {
        let relationNode = []//存放生成的link信息
        //stockNode.push({"id": res.shorthand, "group": 1,"type":"stock","code":res.code,"nodeAttr":res.chesename})//信息写入临时节
        for(let i = 0;i <response.data.length;i++)
        {
          relationNode.push({"id": response.data[i].node.shorthand, "group": 1,"type":"stock",
            "code":response.data[i].code,"nodeAttr":response.data[i].chinesename})
        }
        this.Graph4Update.nodes=relationNode;//将返回节点加入Graph
      }
    })
    .catch(function(error)
    {
      console.log(error)
    })
    this.initGraph(this.Graph4Update)//初始化画布

  },
  methods:{
    updateGraph(data){
      const _this = this
      // const nodes = data.nodes.map(d => Object.create(d));
      // const links = data.links.map(d => Object.create(d));
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
        .attr("r", 40)
        .attr("fill", function (d) {
          return _this.colorList[d.group%10];
        })
        .on("mouseover", function(d) {
          d3.select(this).style("stroke", "orange");
          _this.nodeAttr = d.nodeAttr;
        })
        .on("mouseout", function(d) {
          d3.select(this).style("stroke", "white");
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
      _this.nodeNameText = _this.g.append("g")//节点的文字显示
        .selectAll("text")
        .data(nodes)
        .join("text")
        .attr("fill","#3edb14")
        .classed("nodeName",true)
        // .attr("fill","white")
        .attr("font-size",20)
        .attr("class","nodeName")
        .text(function (d)
        {
          if(d.type != "stock")
          {
            return d.id
          }
          else
          {
            return d.code + d.id
          }
        })
        .attr("dx",function(){
          // -10;
          return this.getBoundingClientRect().width/2*(-1);
        })
        .attr("dy",60);
      // console.log(nodeNameText)

      _this.linkRelation = _this.g.append("g")//连接的文字显示
        .selectAll("text")
        .data(links)
        .join("text")
        // .attr("x",100)
        // .attr("y",80)
        .style("text-anchor", "middle")
        .style("fill","white")
        .style("font-size","20px")
        .style("font-weight", "bold")
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
      _this.simulation.nodes(nodes)
      _this.simulation.force("link").links(links)
      _this.simulation.alpha(1).restart()
      console.log(nodes)
      console.log(_this.nodes)
      // console.log(_this.simulation.nodes(nodes))

    },

    initGraph(data){
      var _this = this
      const links = data.links
      const nodes = data.nodes
      // const nodes = data.nodes.map(d => Object.create(d));
      // const links = data.links.map(d => Object.create(d));

      _this.simulation = d3.forceSimulation(nodes)
        .force("link", d3.forceLink(links).id(d => d.id).distance(200))
        .force("charge", d3.forceManyBody().strength(-30))
        .force("center", d3.forceCenter(_this.width / 2, _this.height / 2));

      const svg = d3.select(".container")
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
        .attr("r", 40)
        .attr("fill", function (d) {
          return _this.colorList[d.group%10];
        })
        .on("mouseover", function(d) {
          d3.select(this).style("stroke", "orange");
          _this.nodeAttr = d.nodeAttr;
        })
        .on("mouseout", function(d) {
          d3.select(this).style("stroke", "white");
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

        // .on("click", queryTest())

      _this.nodeNameText = _this.g.append("g")//节点的文字显示
      .selectAll("text")
      .data(nodes)
      .join("text")
      .attr("fill","#3edb14")
      .classed("nodeName",true)
      .attr("font-size",20)
      .attr("class","nodeName")
      .text(function (d)
      {
        if(d.type != "stock")
        {
          return d.id
        }
        else
        {
          return d.code + d.id
        }
      })
      .attr("dx",function(){
        return this.getBoundingClientRect().width/2*(-1);
      })
      .attr("dy",60)
      ;

      _this.linkRelation = _this.g.append("g")//连接的文字显示
      .selectAll("text")
      .data(links)
      .join("text")
      .style("text-anchor", "middle")
      .style("fill","white")
      .style("font-size","20px")
      .style("font-weight", "bold")
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
      const positiveMarker = svg.append("marker")//连接线的箭头效果，正反方向
        .attr("id","positiveMarker")
        .attr("orient","auto")
        .attr("stroke-width",2)
        .attr("markerUnits","strokeWidth")
        .attr("markerUnits", "userSpaceOnUse")
        .attr("viewBox", "0 -5 10 10 ")
        .attr("refX",45)
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
        .attr("refX",135)
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

        _this.linkRelation
          .attr("font-size",20)
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

    searchStock(SearchInput){
      //对搜索框结果进行查询，暂时合并用户筛选功能
      //如果与后端整合，需将MsgBoard.vue 中 mounted 中searchName函数中的调用改为this.$parent.searchStock(data);
      this.axios.post("/searchStock",{
              searchInput:SearchInput//string,搜索框输入信息
            })
            .then(function(response)
            {
              if(response.status === 200)
              {
                let StockId=response.data;//后端返回股票id
                this.search("stock",StockId);//向后端查询股票id，并更新画布
              }
            })
            .catch(function(error)
            {
              console.log(error)
            })
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
    // }

    search(nodeType,nodeMsg)//点击/查询事件，向后端请求数据
    {
      let _this = this
      if (nodeType == "stock")
      //点击节点为股票，输入股票代码（nodeMsg），返回股票信息node
      {
        this.axios.post("/"+nodeType,{
          code:nodeMsg
        })
        .then(function(response)
        {
          if(response.status === 200)
          {
            let stockNode = []
            let stockLink = []
            let res=response.data.node;
            _this.stockGraph.conceptionList = res.conception//获取概念列表
            _this.stockGraph.conceptionExtension = false
            // res = {"shorthand": "广东鸿图","stockcode":"002101","industry":"行业名称",
            // "location":"广东","plate":"主板","conception":["c1","c2"],"chinesename":"广东鸿图公司","nodeattr":"node"}
            stockNode.push({"id": res.shorthand, "group": 1,"type":"stock","code":res.code,"nodeAttr":res.chesename})//信息写入临时节点
            stockNode.push({"id": res.industry, "group": 2,"type":"industry"})
            stockNode.push({"id": res.location, "group": 2,"type":"location"})
            stockNode.push({"id": res.plate, "group": 2,"type":"plate"})
            stockNode.push({"id": "所属概念", "group": 2,"type":"conceptionTag"})
            _this.stockGraph.nodes = stockNode
            stockLink.push({"source": res.id, "target": res.industry, "value": 3,"relation":"行业"})//写入关系
            stockLink.push({"source": res.id, "target": res.location, "value": 3,"relation":"地域"})
            stockLink.push({"source": res.id, "target": res.plate, "value": 3,"relation":"板块"})
            stockLink.push({"source": res.id, "target": "所属概念", "value": 3,"relation":"概念"})
            _this.stockGraph.links = stockLink
          }
        })
        .catch(function(error)
        {
          console.log(error)
        })
        this.updateGraph(this.stockGraph)
      }
      else if (nodeType == "conceptionTag")
      //展开概念信息
      {
        if (this.stockGraph.conceptionExtension)//概念已展开，弹出所有概念节点和关系，并将概念展开记为false
        {
          for(let i=0;i<this.stockGraph.conceptionList.length;i++)
          {
            this.stockGraph.links.pop()
            this.stockGraph.nodes.pop()
          }
          this.stockGraph.conceptionExtension = false
        }
        else//概念未展开，添加节点和关系
        {
          for(let i=0;i<this.stockGraph.conceptionList.length;i++)
          {
            this.stockGraph.nodes.push({"id":this.stockGraph.conceptionList[i], "group": 2,"type":"conception"})
            this.stockGraph.links.push({"source": "所属概念", "target": this.stockGraph.conceptionList[i],
              "value": 2,"relation":"概念"+(i+1)},)
          }
          this.stockGraph.conceptionExtension = true
        }
        this.updateGraph(this.stockGraph)
      }
      else//根据关系查询股票,nodeMsg为关系名称,nodeType为关系类型（plate/location/conception）
      {
        this.axios.post("/" + nodeType,{
          limit:25,
          relation:nodeMsg,
        })
        .then(function(response)
        {
          if(response.status === 200)
          {
            let relationNode = []//存放生成的link信息
            //stockNode.push({"id": res.shorthand, "group": 1,"type":"stock","code":res.code,"nodeAttr":res.chesename})//信息写入临时节
            for(let i = 0;i <response.data.length;i++)
            {
              relationNode.push({"id": response.data[i].node.shorthand, "group": 1,"type":"stock",
                "code":response.data[i].code,"nodeAttr":response.data[i].chinesename})
            }
            this.Graph4Update.nodes=relationNode;//将返回节点加入Graph

            this.Graph4Update.nodes.push({"id": nodeMsg, "group": 1,"type":nodeType})//增放中心节点
            let relationLink = []//存放生成的link信息
            for(let i = 0;i <response.data.length;i++)
            {
              relationLink.push({"source": response.data[i].id, "target": nodeMsg, "value": 3,"relation":" "})
            }
            this.Graph4Update.links = relationLink
            this.updateGraph(this.Graph4Update)
          }
        })
        .catch(function(error)
        {
          console.log(error)
        })
        this.initGraph(this.Graph4Update)
      }
    }

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
  .container
  {
    width:72%;
    float: left;
    height: auto;
    border: 1px solid #2c3e50;
    border-radius: 8px;
    /*margin-top: 40px;*/
    margin-left:auto;
    margin-right:auto;
    background: #154360 repeating-linear-gradient(30deg,
    hsla(0,0%,100%,.1),hsla(0,0%,100%,.1) 15px,
    transparent 0, transparent 30px);
  }
  .right{
    height: auto;
    width:25%;
    background: #5cd9cf;
    float: right;
  }
  #searchPage
  {
    height: 250px;
  }
  #searchResult
  {
    display: None;
  }
  #boardPage
  {
    height:310px;
  }
</style>
