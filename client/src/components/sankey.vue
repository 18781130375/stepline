<template>
  <div id="chart" style="width:100%;height:100%">
  </div>
</template>
<script>
import axios from 'axios';
import $ from 'jquery'
import { mapGetters } from 'vuex'
import * as d3 from 'd3v3'
import back from '@/assets/chart.png'

export default {
  name: "sankey",
  data() {
    return {
      back:back
    }
  },
  mounted() {
    this.sankey();
    // axios.get('http://127.0.0.1:5000/getMsg')
    //   .then(function (response) {
    //     console.log(response);
    //   })
    //   .catch(function (error) {
    //     console.log(error);
    //   }
    var units = "Widgets";


     /*height 修改 改 nodePadding*/
    var margin = { top: 50, right: 10, bottom: 10, left: 150 },
      width = 1450- margin.left - margin.right,
      height = 800 - margin.top - margin.bottom;

    var formatNumber = d3.format(",.0f"), // 小数点零点
      format = function(d) { return formatNumber(d) + " " + units; },
      color = "#add8e6";


    // 添加画布
    var svg = d3.select("#chart").append("svg")
      .attr("width","100%" )        //width + margin.left + margin.right + 150
      .attr("height","100%")       // height + margin.top + margin.bottom + 100
      .append("g")
      .attr("transform",
        "translate(" + margin.left + "," + margin.top + ")");


    // 设置sankey图标属性
    var sankey = d3.sankey()
      .nodeWidth(30) //节点宽度
      .nodePadding(30) //表示矩形的垂直方向的间距；
      .size([width, height]); //表示整个sankey图占用的空间大小

    var path = sankey.link(); //路径生产器

    // 加载数据 (using the timelyportfolio csv method)
    d3.json("http://127.0.0.1:5000/getMsg/", function(error, all_data) {
      //console.log(data);
      //set up graph in same style as original example but empty，与原始示例设置相同的图形，但空
      //console.log(all_data);
      var nodesColor=["#388712","#60DD49","#FCE639","#FFA91A","#FF3030"];   //绿到红
      var nodetext=["威胁等级低","威胁等级较低","威胁等级中等","威胁等级较高","威胁等级高"];

      //色块等级说明
        var colorsexplain=svg.append("g").selectAll(".explain")
          .data(nodesColor)
          .enter()
          .append("g")
          .attr("class","explain")

      colorsexplain.append("rect")
        .attr("x","70%")
        .attr("y",function (d){
          for (var i=0;i<nodesColor.length;i++)
          if(nodesColor[i]==d)
          {
            return i*40
          }
        })
        .attr("height",20)
        .attr("width",20)
        .style("fill",function (d) {return d})
        .style("filter", "url(#drop-shadow)") //添加阴影

      colorsexplain.append("text")
        .data(nodetext)
        .attr("x","72%")
        .attr("y",function (d) {
          for(var i=0;i<nodetext.length;i++)
          {
            if(nodetext[i]==d)
            {
              return i*43
            }
          }
        })
        .attr("dy", ".35em")
        .attr("text-anchor", "start")
         .text(function (d) { return d })
        .attr("fill","#fff")
        .style("filter", "url(#drop-shadow)");


      var graph = { "nodes": [], "links": []  };
      graph.nodes = all_data['2'];
      graph.links = all_data['0'];
      var attackpath =all_data['1'];
      //console.log(graph.nodes);

      sankey
        .nodes(graph.nodes)
        .links(graph.links)
        .layout(32); //layout中的参数表示桑基布局用来优化流布局的时间。

     //拖拽和点击的区分
     //  function distinguish() {
     //    var flag=0;
     //
     //   var element=xxxx;
     //
     //  element.addEventListener("mousedown",function () {
     //    flag=0;
     //  },false);
     //  element.addEventListener("mousemove", function(){
     //   flag = 1;
     //  }, false);
     //  element.addEventListener("mouseup", function(){
     //    if(flag === 0){
     //      alert("click");
     //    }
     //    else if(flag === 1){
     //      alert("drag");
     //    }
     //  }, false);
     //
     //  }


      // 添加节点
      var node = svg.append("g").selectAll(".node")
        .data(graph.nodes)
        .enter().append("g")
        .attr("class", "node")
        .attr("transform", function(d) {
          return "translate(" + d.x + "," + d.y +")";
        })

       .on("click",function () {
            alert("click")
        })

        .call(d3.behavior.drag()
          .origin(function(d) { return d; })
          .on("dragstart", function() {
            this.parentNode.appendChild(this);
          })
         .on("drag", dragmove))



     // var a = d3.rgb(175, 238, 238, .2); //左侧
      //var b = d3.rgb(76, 165, 246, .7); //右侧

      var a=d3.rgb(255,255,0);   //黄色
      var b = d3.rgb(255,0,0);   //绿色


      var compute=d3.interpolate(a,b);


      //定义一个线性渐变
       var defs = svg.append("defs");
      // var linearGradient = defs.append("linearGradient")
      //   .attr("id", "linearColor")
      //   .attr("x1", "0%")
      //   .attr("y1", "0%")
      //   .attr("x2", "100%")
      //   .attr("y2", "0%");
      //
      // var stop1 = linearGradient.append("stop")
      //   .attr("offset", "0%")
      //   .style("stop-color", a.toString());
      //
      // var stop2 = linearGradient.append("stop")
      //   .attr("offset", "100%")
      //   .style("stop-color", b.toString());

      //添加阴影
      var filter = defs.append("filter")
        .attr("id", "drop-shadow")
        .attr("width", "200%")
        .attr("height", "200%");

      filter.append("feGaussianBlur")
        //  .attr("in", "SourceAlpha")
        .attr("stdDeviation", 5)

        .attr("result", "blur");

      // translate output of Gaussian blur to the right and downwards with 2px
      // store result in offsetBlur
      filter.append("feOffset")
        .attr("in", "blur")
        .attr("dx", 0)
        .attr("dy", 0)
        .attr("result", "offsetBlur");

      var feMerge = filter.append("feMerge");

      feMerge.append("feMergeNode")

        .attr("in", "offsetBlur")
      feMerge.append("feMergeNode")

        .attr("in", "SourceGraphic");

     // console.log(graph.nodes[0].name[graph.nodes[0].name.length-1])
    // console.log(Math.max(d.sourceLinks.length,d.targetLinks.length));
      // 为节点添加矩形
      node.append("rect")
        .attr("height", function(d) {  return d.dy; })
        .attr("width", function (){ return sankey.nodeWidth()/2; })
        //.attr("rx", "9px") //圆角设置
        //.attr("ry", "9px")
        //.style("fill", "url(#" + linearGradient.attr("id") + ")") //添加线性渐变
        //.style("fill",function()  { return compute(Math.random()); })
        .style("fill",function (d) {
          return nodesColor[d.name[d.name.length-1]-1];
        })
       // .style("opacity",.7)
        //.style("stroke","#CDC9C9")      //描边颜色设置
        .style("stroke-width", "1.5px")
        .style("filter", "url(#drop-shadow)") //添加阴影
        .on("mouseover", function(d) {
          var name = d.name;
          var datapath = attackpath[name];
         // console.log(datapath);
          for (var item in datapath) {
            var dom = d3.select("." + datapath[item]['0'] + "→" + datapath[item]['1']);
            dom.style("stroke-opacity", 1) //连线高亮设置
          }
        })


        .on("mouseout", function(d) {
          var name = d.name;
          var datapath = attackpath[name];
          for (var item in datapath) {
            //nsole.log(datapath[item][0]);
            var dom = d3.select("." + datapath[item]['0'] + "→" + datapath[item]['1']);
            dom.style("stroke-opacity", .5);
          }
        })
        .append("title")
        .text(function(d) {
          return d.name + "\n" + format(d.value);
        });

      //添加节点的标题
      // node.append("text")
      //   .attr("x", sankey.nodeWidth() * 1.1) //右侧标题的坐标
      //   .attr("y", sankey.nodeWidth() / 3.5)
      //   .attr("fill", "#fff")
      //   .attr("dy", ".35em")
      //   .attr("text-anchor", "start") //文本相对基点（x，y）的位置
      //   .attr("transform", null)
      //   .text(function(d) { return d.name; })
      //   .filter(function(d) { return d.x < width / 3; })
      //   .attr("x", sankey.nodeWidth()-25) //左侧及中间的文本坐标
      //   .attr("y",sankey.nodeWidth()-15)
      //   .attr("text-anchor", "end")
      //   .attr("fill", "#fff");

     // console.log(graph.nodes[0].name(0,graph.nodes[0].name.length));
     node.append("text")
      .attr("x", function (d) { return d.dx/1.5})
      .attr("y", function(d) { return d.dy*1.3; })
      .attr("dy", ".35em")
      .attr("text-anchor", "end")
      .attr("transform", null)
      .text(function(d) { return d.name.substr(0,d.name.length-1); })
      .filter(function(d) { return d.x < width / 2; })
      .attr("x",  function (){ return sankey.nodeWidth()/6 })   //左侧
      .attr("y", function (d) { return d.dy*1.3})
      .attr("text-anchor", "start")
      .attr("fill", "#fff");

      // //新建箭头
      // var arrowMarker = defs.append("marker")
      //   .attr("id", "arrow")
      //   .attr("markerUnits", "strokeWidth")
      //   .attr("markerWidth", 6)
      //   .attr("markerHeight", 6)
      //   .attr("viewBox", "0 0 12 12")
      //   .attr("refX", 8)
      //   .attr("refY", 6)
      //   .attr("orient", "auto")
      // var arrow_path = "M2,2 L10,6 L2,10 L6,6 L2,2";
      // arrowMarker.append("path")
      //   .attr("d", arrow_path)
      //   .attr("fill", "#fff")
      //   .attr("opacity", 0.5);

      //添加链接
      var link = svg.append("g").selectAll(".link")
        .data(graph.links)
        .enter().append("path")
        .attr("class", function(d) {
          return "link " + d.source.name + "→" + d.target.name;
        })
        //.attr("marker-end", "url(#arrow)")
        .attr("d", path)
        .style("stroke-width", function(d) { return d.dy; }) //连线宽度
        .style("strock-opacity",.5)
        .sort(function(a, b) { return b.dy - a.dy; });
        // .on("click",function (d) {
        //   d3.selectAll(".link")
        //     .style("stroke-width", function(d) { return d.dy/2; })
        // });

      // 添加链接标题
      link.append("title")
        .text(function(d) {
          return d.source.name + "→" + d.target.name + "\n" + format(d.value);
        })


      // 拖拽功能
      function dragmove(d) {
        {
          d3.select(this).attr("transform",
            "translate(" + (
              d.x = Math.max(0, Math.min(width - d.dx, d3.event.x))
            ) + "," + (
              d.y = Math.max(0, Math.min(height - d.dy, d3.event.y))
            )  + ")");
          sankey.relayout();
          link.attr("d", path);
        }
      }

    });

  },
  methods: {
    sankey() {

      //sankey插件
      d3.sankey = function() {
        var sankey = {},
          nodeWidth = 24,
          nodePadding = 50,
          size = [1, 1],
          nodes = [],
          links = [];
         var margin = { top: 30, right: 10, bottom: 10, left: 50 },
      width = 950 - margin.left - margin.right,
      height = 300 - margin.top - margin.bottom;


        sankey.nodeWidth = function(_) {
          if (!arguments.length) return nodeWidth;
          nodeWidth = +_;
          return sankey;
        };

        sankey.nodePadding = function(_) {
          if (!arguments.length) return nodePadding;
          nodePadding = +_;
          return sankey;
        };

        sankey.nodes = function(_) {
          if (!arguments.length) return nodes;
          nodes = _;
          return sankey;
        };

        sankey.links = function(_) {
          if (!arguments.length) return links;
          links = _;
          return sankey;
        };
     
        sankey.size = function(_) {
          if (!arguments.length) return size;
          size = _;
          return sankey;
        };

        sankey.layout = function(iterations) {
          computeNodeLinks();
          computeNodeValues();
          computeNodeBreadths();
          computeNodeDepths(iterations);
          computeLinkDepths();
          return sankey;
        };

        sankey.relayout = function() {
          computeLinkDepths();
          return sankey;
        };

        sankey.link = function() {
          var curvature = .5;

          function link(d) {
            var x0 = d.source.x + d.source.dx -15,
              x1 = d.target.x,
              xi = d3.interpolateNumber(x0, x1),
              x2 = xi(curvature),
              x3 = xi(1 - curvature),
              y0 = d.source.y + d.sy + d.dy/2,
              y1 = d.target.y + d.ty + d.dy/2;
            return "M" + x0 + "," + y0 +
              "C" + x2 + "," + y0 +
              " " + x3 + "," + y1 +
              " " + x1 + "," + y1;
          }

          link.curvature = function(_) {
            if (!arguments.length) return curvature;
            curvature = +_;
            return link;
          };

          return link;
        };

        // Populate the sourceLinks and targetLinks for each node.
        // Also, if the source and target are not objects, assume they are indices.
        function computeNodeLinks() {
          nodes.forEach(function(node) {
            node.sourceLinks = [];
            node.targetLinks = [];
          });
          links.forEach(function(link) {
            var source = link.source,
              target = link.target;
            if (typeof source === "number") source = link.source = nodes[link.source];
            if (typeof target === "number") target = link.target = nodes[link.target];
            source.sourceLinks.push(link);
            target.targetLinks.push(link);
          });
        }

        // Compute the value (size) of each node by summing the associated links.
        function computeNodeValues() {
          nodes.forEach(function(node) {
            node.value = Math.max(
              d3.sum(node.sourceLinks, value),
              d3.sum(node.targetLinks, value)
            );
          });
        }

        // Iteratively assign the breadth (x-position) for each node.
        // Nodes are assigned the maximum breadth of incoming neighbors plus one;
        // nodes with no incoming links are assigned breadth zero, while
        // nodes with no outgoing links are assigned the maximum breadth.
        function computeNodeBreadths() {
          var remainingNodes = nodes,
            nextNodes,
            x = 0;

          while (remainingNodes.length) {
            nextNodes = [];
            remainingNodes.forEach(function(node) {
              node.x = x;
              node.dx = nodeWidth;
              node.sourceLinks.forEach(function(link) {
                nextNodes.push(link.target);
              });
            });
            remainingNodes = nextNodes;
            ++x;
          }

          //
          moveSinksRight(x);
          scaleNodeBreadths((width - nodeWidth) / (x - 1));
        }

        function moveSourcesRight() {
          nodes.forEach(function(node) {
            if (!node.targetLinks.length) {
              node.x = d3.min(node.sourceLinks, function(d) {
                return d.target.x;
              }) - 1;
            }
          });
        }

        function moveSinksRight(x) {
          nodes.forEach(function(node) {
            if (!node.sourceLinks.length) {
              node.x = x - 1;
            }
          });
        }

        function scaleNodeBreadths(kx) {
          nodes.forEach(function(node) {
            node.x *= kx;
          });
        }

        function computeNodeDepths(iterations) {
          var nodesByBreadth = d3.nest()
            .key(function(d) {
              return d.x;
            })
            .sortKeys(d3.ascending)
            .entries(nodes)
            .map(function(d) {
              return d.values;
            });


          initializeNodeDepth();
          resolveCollisions();
          for (var alpha = 1; iterations > 0; --iterations) {
            relaxRightToLeft(alpha *= .99);
            resolveCollisions();
            relaxLeftToRight(alpha);
            resolveCollisions();
          }

          function initializeNodeDepth() {
            var ky = d3.min(nodesByBreadth, function(nodes) {
              return (size[1] - (nodes.length - 1) * nodePadding) / d3.sum(nodes, value);
            });

            nodesByBreadth.forEach(function(nodes) {
              nodes.forEach(function(node, i) {
                node.y = i;
                node.dy = node.value * ky;
              });
            });

            links.forEach(function(link) {
              link.dy = link.value * ky;
            });
          }

          function relaxLeftToRight(alpha) {
            nodesByBreadth.forEach(function(nodes, breadth) {
              nodes.forEach(function(node) {
                if (node.targetLinks.length) {
                  var y = d3.sum(node.targetLinks, weightedSource) / d3.sum(node.targetLinks, value);
                  node.y += (y - center(node)) * alpha;
                }
              });
            });

            function weightedSource(link) {
              return center(link.source) * link.value;
            }
          }

          function relaxRightToLeft(alpha) {
            nodesByBreadth.slice().reverse().forEach(function(nodes) {
              nodes.forEach(function(node) {
                if (node.sourceLinks.length) {
                  var y = d3.sum(node.sourceLinks, weightedTarget) / d3.sum(node.sourceLinks, value);
                  node.y += (y - center(node)) * alpha;
                }
              });
            });

            function weightedTarget(link) {
              return center(link.target) * link.value;
            }
          }

          function resolveCollisions() {
            nodesByBreadth.forEach(function(nodes) {
              var node,
                dy,
                y0 = 0,
                n = nodes.length,
                i;

              // Push any overlapping nodes down.
              nodes.sort(ascendingDepth);
              for (i = 0; i < n; ++i) {
                node = nodes[i];
                dy = y0 - node.y;
                if (dy > 0) node.y += dy;
                y0 = node.y + node.dy + nodePadding;
              }

              // If the bottommost node goes outside the bounds, push it back up.
              dy = y0 - nodePadding - size[1];
              if (dy > 0) {
                y0 = node.y -= dy;

                // Push any overlapping nodes back up.
                for (i = n - 2; i >= 0; --i) {
                  node = nodes[i];
                  dy = node.y + node.dy + nodePadding - y0;
                  if (dy > 0) node.y -= dy;
                  y0 = node.y;
                }
              }
            });
          }

          function ascendingDepth(a, b) {
            return a.y - b.y;
          }
        }

        function computeLinkDepths() {
          nodes.forEach(function(node) {
            node.sourceLinks.sort(ascendingTargetDepth);
            node.targetLinks.sort(ascendingSourceDepth);
          });
          nodes.forEach(function(node) {
            var sy = 0,
              ty = 0;
            node.sourceLinks.forEach(function(link) {
              link.sy = sy;
              sy += link.dy;
            });
            node.targetLinks.forEach(function(link) {
              link.ty = ty;
              ty += link.dy;
            });
          });

          function ascendingSourceDepth(a, b) {
            return a.source.y - b.source.y;
          }

          function ascendingTargetDepth(a, b) {
            return a.target.y - b.target.y;
          }
        }

        function center(node) {
          return node.y + node.dy / 2;
        }

        function value(link) {
          return link.value;
        }

        return sankey;
      };
    }
  }
}

</script>
<style scoped>
.node rect {
  cursor: move;
  fill-opacity: .5;
  shape-rendering: crispEdges;
}

.node text {
  pointer-events: none;
  text-shadow: 2px 1px 7px #fff;
}

#chart{
  background:#020D1F;/*url("../assets/chart1300.png");*/
  background-size: 100% 100%; 
}
</style>
