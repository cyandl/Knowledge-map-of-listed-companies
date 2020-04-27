<template>
  <div style="width: 100%">
    <div class="container" >
    </div>
    <div id="right">
      <h2>股票代码/股票名称: <input type="text" name="firstname"></h2>
      <h2>查询关系：
        <select>
        <option value ="volvo">所属行业</option>
        <option value ="saab">所属概念</option>
        <option value="audi">高管信息</option>
        </select>
      </h2>
      <p>公司信息：xxx</p>
      <input type="submit" value="查询信息">

    </div>
  </div>
<!--  <div class="container">-->
<!--  </div>-->
</template>

<script>
  import * as d3 from 'd3'
  import * as axios from 'axios'
  export default {
  name: 'HelloWorld',
  data () {
    return {
      width:800,
      height:600,
      testGraph:{
        "nodes":[
          {"id": "东方财富", "group": 1},
          {"id": "证券", "group": 30},
          {"id": "上海", "group": 30},
          {"id": "300059", "group": 30}
        ],
        "links":[
          {"source": "东方财富", "target": "证券", "value": 1,"relation":"行业"},
          {"source": "东方财富", "target": "上海", "value": 10,"relation":"地域"},
          {"source": "东方财富", "target": "300059", "value": 8,"relation":"股票代码"},
        ]
      },
      shGraph:{
        "nodes":[
          {"id": "上海", "group": 1},
          {"id": "浦发银行", "group": 2},
          {"id": "上海机场", "group": 3},
          {"id": "东方财富", "group": 4}
        ],
        "links":[
          {"source": "上海", "target": "浦发银行", "value": 5,"relation":"地域"},
          {"source": "上海", "target": "上海机场", "value": 5,"relation":"地域"},
          {"source": "上海", "target": "东方财富", "value": 5,"relation":"地域"},
        ]
      }
    }
  },
  mounted(){
    this.initGraph(this.testGraph)
  },
  methods:{
    updateGraph(data){
    },
    initGraph(data){
      var _this = this
      const links = data.links;
      const nodes = data.nodes;

      const simulation = d3.forceSimulation(nodes)
        .force("link", d3.forceLink(links).id(d => d.id).distance(200))
        .force("charge", d3.forceManyBody().strength(-30))
        .force("center", d3.forceCenter(_this.width / 2, _this.height / 2));

      const  svg = d3.select(".container")
        .append("svg")
        .attr("viewBox", [0, 0, _this.width, _this.height]);
      svg.call(d3.zoom().on("zoom",function(){
        // svg.attr("transform",d3.event.transform)
        g.attr("transform",d3.event.transform)
      }))

      const g = svg.append("g")

      const link = g.append("g")//节点的关系连线
        .attr("marker-end","url(#positiveMarker)")
        .attr("stroke", "#999")
        .attr("stroke-opacity", 0.6)
        .selectAll("path")
        .data(links)
        .join("path")
        .attr("stroke-width", d => Math.sqrt(d.value))
        .attr("id",d => d.source + "_" + d.relation+"_"+d.target);

      const node = g.append("g")//节点声明
        .attr("stroke", "#ffffff")
        .attr("stroke-width", 1.5)
        .selectAll("circle")
        .data(nodes)
        .join("circle")
        .attr("r", 40)
        .attr("fill", function (d) {
          const scale = d3.scaleOrdinal(d3.schemeCategory10);
          return scale(100+d.group);
        })
        .attr("class","node")
        .on("mouseover", function(d) {
          d3.select(this).style("stroke", "orange");})
        .on("mouseout", function(d) {
          d3.select(this).style("stroke", "white");
        })
        .call(_this.drag(simulation))
        .on("click",d=>//鼠标监听
        {
          if (d.id == "上海")
          {
            this.testGraph.nodes = this.shGraph.nodes
            this.testGraph.links = this.shGraph.links
            this.initGraph(this.shGraph)
            // _this.link=globalData.shGraph.links
            // _this.updateGraph(globalData.shGraph)
            console.log(data.nodes);
          }
        });

      node.append("title")
        .text(d => d.id)

        // .on("click", queryTest())

      const nodeNameText = g.append("g")//节点的文字显示
      .selectAll("text")
      .data(nodes)
      .join("text")
      .attr("fill","#3edb14")
      .attr("dx",function(){
        return -30
      })
      .attr("dy",60)
      .classed("nodeName",true)
      // .attr("fill","white")
      .attr("font-size",20)
      .attr("class","nodeName")
      .text(function (d)
      {
        return d.id
      })
      // console.log(nodeNameText)

      const linkRelation = g.append("g")//连接的文字显示
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

      simulation.on("tick", () => {//坐标控制
        link
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

        node
          .attr("cx", d => d.x)
          .attr("cy", d => d.y);

        nodeNameText
          .attr("x",d => d.x)
          .attr("y",d => d.y);

        linkRelation
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
    height: 600px;
    border: 1px solid #2c3e50;
    border-radius: 8px;
    /*margin-top: 40px;*/
    margin-left:auto;
    margin-right:auto;
    background: #154360 repeating-linear-gradient(30deg,
    hsla(0,0%,100%,.1),hsla(0,0%,100%,.1) 15px,
    transparent 0, transparent 30px);
  }
  #right{
    height: 600px;
    width:25%;
    background: #5cd9cf;
    float: right;
  }
</style>
