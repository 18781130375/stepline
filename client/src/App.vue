<template>
  <div id="app">
    <div id="Inf_view1">
      <ui ref="ui"></ui>
    </div>
    <div id="main_view">
      <!--<Ping ref="Ping"  @nodeselect="nodeselect"></Ping>-->
      <!--<haddata ref="haddata" @nodeselect="nodeselect">-->
      <sankey></sankey>
    </div>
  </div>
</template>
<script>
import * as d3 from 'd3v4'
//import maindiv from './components/main'
import lineS from './components/line'
import Ping from './components/Ping'
import haddata from './components/haddata'
import ui from './components/GUI'
import sankey from './components/sankey'
export default {
  name: 'App',
  data() {
    return {
    }
  },
  components: {
    //  maindiv,
    ui,
    haddata,
    Ping,
    sankey
  },
  methods: {

    Inf_select(node) {
      console.log(node);
    },
    nodeselect(node) {
      //根据传入参数的node.seriesId不同调用不同的更新图表函数，并将参数传入实现图表连动

      if (node == "nodetype") {
        this.$refs.Ping.daga("run");
      } else if (node == "wtf") {
        this.$refs.haddata.qwe(this.$store.state.shuju, this.$store.state.ifcl)
      } else if (node == "wtfs") {
        this.$refs.Ping.daga("run");
      } else if (node == "folder_scan") {
        this.$refs.sunburst.change("folder_scan");
      }
    }
  },
  beforeCreate() {},
  created() {},
  beforeMount() {
    var that = this;

  },
  mounted() {}
}

</script>
<style>
html {

  height: 100%;
  width: 100%;
}

body {
  background-color: #020D1F;
  height: 100%;
  width: 100%;
}

* {

  margin: 0px;
  padding: 0px;
  box-sizing: border-box;
}

#app {

  width: 100%;
  height: 100%;
}

#main_view {
  width: 90%;
  height: 100%;
  border: 1px solid #9A2EFE;
  float: left;
}

#Inf_view1 {
  width: 10%;
  height: 100%;
  border: 1px solid #9A2EFE;
  float: left;
}

/*#sys_email{
  width: 100%;
  height: 100%;
}

.Double_view{
  width: 100%;
  height: 40%;
  border: 1px solid  #9A2EFE;
  border-left: none;
}*/

</style>
