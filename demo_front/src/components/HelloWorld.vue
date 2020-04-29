<template>
  <div style="width: 100%">
    <div class="container" >
    </div>
    <div id="right">
      <h2>股票代码/股票名称: <input type="text" name="firstname"></h2>
      <input type="submit" value="查询信息">
      <h2>查询关系：
        <select>
        <option value ="volvo">行业</option>
        <option value ="saab">板块</option>
        <option value="audi">概念</option>
        <option value="audi">地域</option>
        </select>
      </h2>
      <table border="1">
        <tr>
          <td>主板</td>
        </tr>
        <tr>
          <td>创业板</td>
        </tr>
        <tr>
          <td>中小板</td>
        </tr>
        <tr>
          <td>科创板</td>
        </tr>
      </table>

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
      nodes:[],//使用全局变量以更新画布
      links:[],
      nodeNameText:[],
      linkRelation:[],
      simulation:null,
      testGraph:{
        "nodes":[
          {"id": "广东鸿图", "group": 1,"type":"stock"},
          {"id": "广东鸿图科技股份有限公司", "group": 30,"type":"company"},
          {"id": "广东", "group": 30,"type":"location"},
          {"id": "主板", "group": 2,"type":"plate"},
          {"id": "行业概念", "group": 2,"type":"conception"}
        ],
        "links":[
          {"source": "广东鸿图", "target": "广东鸿图科技股份有限公司", "value": 1,"relation":"行业"},
          {"source": "广东鸿图", "target": "广东", "value": 10,"relation":"地域"},
          {"source": "广东鸿图", "target": "主板", "value": 8,"relation":"板块"},
          {"source": "广东鸿图", "target": "行业概念", "value": 8,"relation":"概念"},
        ]
      },
      gdGraph:{
        "nodes":[
          {"id": "广东鸿图", "group": 1,"type":"stock"},
          {"id": "文化长城", "group": 1,"type":"stock"},
          {"id": "白云山", "group": 1,"type":"stock"},
          {"id": "广东", "group": 1,"type":"location"},
        ],
        "links":[
          {"source": "广东", "target": "白云山", "value": 1,"relation":" "},
          {"source": "广东", "target": "广东鸿图", "value": 10,"relation":" "},
          {"source": "广东", "target": "文化长城", "value": 8,"relation":" "},
        ]
      },
      zbGraph:{
        "nodes":[
          {"id": "广东鸿图", "group": 1,"type":"stock"},
          {"id": "文化长城", "group": 1,"type":"stock"},
          {"id": "白云山", "group": 1,"type":"stock"},
          {"id": "东方财富", "group": 1,"type":"stock"},
          {"id": "中国石油", "group": 1,"type":"stock"},
          {"id": "主板", "group": 1,"type":"plate"},
        ],
        "links":[
          {"source": "主板", "target": "白云山", "value": 1,"relation":" "},
          {"source": "主板", "target": "广东鸿图", "value": 10,"relation":" "},
          {"source": "主板", "target": "东方财富", "value": 8,"relation":" "},
          {"source": "主板", "target": "中国石油", "value": 8,"relation":" "},
          {"source": "主板", "target": "文化长城", "value": 8,"relation":" "},
        ]
      },
    }
  },

  mounted(){
    this.initGraph(this.testGraph)
  },
  methods:{
    updateGraph(data,nodeEnter,linkEnter){
      const _this = this
      // const nodes = data.nodes.map(d => Object.create(d));
      // const links = data.links.map(d => Object.create(d));
      let links = data.links
      let nodes = data.nodes

      if (nodeEnter)//绑定元素大于已有元素，使用enter占位
      {
        _this.nodes = _this.nodes
        .data(nodes)
        // .exit().remove()
        .enter()
        .append("circle")
        .attr("r", 40)
        .text(d => d.id)
        .attr("fill", function (d) {
          const scale = d3.scaleOrdinal(d3.schemeCategory10);
          return scale(100+d.group);
        })
        .attr("class","node")
        .merge(_this.nodes)
        .call(_this.drag(_this.simulation))
          .on("click",d=>//鼠标监听
          {
            let tGraph = _this.testGraph;
            if (d.type == "location")
            {
              tGraph = _this.gdGraph
            }
            if (d.type == "stock")
            {
              tGraph = _this.testGraph
            }
            if (d.type == "plate")
            {
              tGraph = _this.zbGraph
            }
            let nodeEnter = true
            let linkEnter = true
            if (_this.nodes.size() > tGraph.nodes.length)
            {nodeEnter = false}
            if (_this.links.size() > tGraph.links.length)
            {linkEnter = false}
            _this.updateGraph(tGraph,nodeEnter,linkEnter);
            console.log(_this.nodes.size())
            console.log(tGraph.nodes.length)
          })
        ;

        _this.nodeNameText = _this.nodeNameText
          .data(nodes)
          .enter()
          // .exit().remove()
          .append("text")
          .merge(_this.nodeNameText)
          .attr("fill","#3edb14")
          .classed("nodeName",true)
          .attr("font-size",20)
          .attr("class","nodeName")
          .text(function (d)
          {
            return d.id
          })
          .attr("dx",function(){
            // -10;
            return this.getBoundingClientRect().width/2*(-1);
          })
          .attr("dy",60);
      }
      else{
      _this.nodes = _this.nodes
        .data(nodes)
        // .enter()
        .exit().remove()
        .merge(_this.nodes)
        .on("click",d=>//鼠标监听
        {
          let tGraph = _this.testGraph;
          if (d.type == "location")
          {
            tGraph = _this.gdGraph
          }
          if (d.type == "stock")
          {
            tGraph = _this.testGraph
          }
          if (d.type == "plate")
          {
            tGraph = _this.zbGraph
          }
          let nodeEnter = true
          let linkEnter = true
          if (_this.nodes.size() > tGraph.nodes.length)
          {nodeEnter = false}
          if (_this.links.size() > tGraph.links.length)
          {linkEnter = false}
          _this.updateGraph(tGraph,nodeEnter,linkEnter);
          console.log(_this.nodes.size())
          console.log(tGraph.nodes.length)
        })
      ;


      _this.nodeNameText = _this.nodeNameText
        .data(nodes)
        // .enter()
        .exit().remove()
        .append("text")
        .merge(_this.nodeNameText)
        .attr("fill","#3edb14")
        .classed("nodeName",true)
        .attr("font-size",20)
        .attr("class","nodeName")
        .text(function (d)
        {
          return d.id
        })
        .attr("dx",function(){
          // -10;
          return this.getBoundingClientRect().width/2*(-1);
        })
        .attr("dy",60);
      }

      if (linkEnter)
      {
        _this.links = _this.links
          .data(links)
          .enter()
          // .exit().remove()
          .append("path")
          .attr("marker-end","url(#positiveMarker)")
          .attr("stroke", "#999")
          .attr("stroke-opacity", 0.6)
          .attr("stroke-width", d => Math.sqrt(d.value))
          .attr("id",d => d.source + "_" + d.relation+"_"+d.target)
          .merge(_this.links);

        _this.linkRelation = _this.linkRelation
          .data(links)
          .enter()
          // .exit().remove()
          .append("textPath")
          .merge(_this.linkRelation)
          .style("text-anchor", "middle")
          .style("fill","white")
          .style("font-size","20px")
          .style("font-weight", "bold")
          .attr("class","linkRelation")
          .attr(
            "xlink:href", d => "#" + d.source +"_" + d.relation + "_" +d.target
          )
          .attr("startOffset" , "50%")
          .text(function (d)
          {
            return d.relation
          })
      }
      else
      {
        _this.links = _this.links
          .data(links)
          // .enter()
          .exit().remove()
          .append("path")
          .attr("marker-end","url(#positiveMarker)")
          .attr("stroke", "#999")
          .attr("stroke-opacity", 0.6)
          .attr("stroke-width", d => Math.sqrt(d.value))
          .attr("id",d => d.source + "_" + d.relation+"_"+d.target)
          .merge(_this.links);

        _this.linkRelation = _this.linkRelation
          .data(links)
          // .enter()
          .exit().remove()
          .append("textPath")
          .merge(_this.linkRelation)
          .style("text-anchor", "middle")
          .style("fill","white")
          .style("font-size","20px")
          .style("font-weight", "bold")
          .attr("class","linkRelation")
          .attr(
            "xlink:href", d => "#" + d.source +"_" + d.relation + "_" +d.target
          )
          .attr("startOffset" , "50%")
          .text(function (d)
          {
            return d.relation
          })
      }

      _this.simulation.nodes(nodes)
      _this.simulation.force("link").links(links)
      _this.simulation.alpha(1).restart()
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

      const  svg = d3.select(".container")
        .append("svg")
        .attr("viewBox", [0, 0, _this.width, _this.height]);
      svg.call(d3.zoom().on("zoom",function(){
        // svg.attr("transform",d3.event.transform)
        g.attr("transform",d3.event.transform)
      }))

      const g = svg.append("g")

      _this.links = g.append("g")//节点的关系连线
        .attr("marker-end","url(#positiveMarker)")
        .attr("stroke", "#999")
        .attr("stroke-opacity", 0.6)
        .selectAll("path")
        .data(links)
        .join("path")
        .attr("stroke-width", d => Math.sqrt(d.value))
        .attr("id",d => d.source + "_" + d.relation+"_"+d.target);

      _this.nodes = g.append("g")//节点声明
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
        .on("mouseover", function(d) {
          d3.select(this).style("stroke", "orange");})
        .on("mouseout", function(d) {
          d3.select(this).style("stroke", "white");
        })
        .call(_this.drag(_this.simulation))
        .on("click",d=>//鼠标监听
        {
          let tGraph = _this.testGraph;
          if (d.type == "location")
          {
            tGraph = _this.gdGraph
          }
          if (d.type == "stock")
          {
            tGraph = _this.testGraph
          }
          if (d.type == "plate")
          {
            tGraph = _this.zbGraph
          }
          let nodeEnter = true
          let linkEnter = true
          if (_this.nodes.size() > tGraph.nodes.length)
          {nodeEnter = false}
          if (_this.links.size() > tGraph.links.length)
          {linkEnter = false}
          _this.updateGraph(tGraph,nodeEnter,linkEnter);
          console.log(_this.nodes.size())
          console.log(tGraph.nodes.length)
        })
      ;

      _this.nodes.append("title")
        .text(d => d.id)

        // .on("click", queryTest())

      _this.nodeNameText = g.append("g")//节点的文字显示
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
        return d.id
      })
      .attr("dx",function(){
        // -10;
        return this.getBoundingClientRect().width/2*(-1);
      })
      .attr("dy",60);
      // console.log(nodeNameText)

      _this.linkRelation = g.append("g")//连接的文字显示
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
