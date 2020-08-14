<template>
  <div class="hello">
      <el-upload
        class="upload-demo"
        action=""
        :on-change="readFile"
        :auto-upload="false">
        <el-button size="small" type="primary">点击上传<i class="el-icon-upload el-icon--right"></i></el-button><i class="el-icon-loading"></i>
        <div slot="tip" class="el-upload__tip">只能上传JSON文件</div>
      </el-upload>
      <div id='container'> </div>
      <textarea disabled v-bind:value='fileContent'></textarea>
      
  
  <div id='myChart' ></div>
  </div>
</template>
<script>
import AMapLoader from '@amap/amap-jsapi-loader';
export default {
  name:'HelloWorld',
  props:{
    msg:String
  },
  data:function(){
    return{
      fileContent:'文件读取内容',
      map:null,
      AMap:null,
      lnglats:[],
    }
  },
  mounted: function () {
    AMapLoader.load({
      "key": "c456a66f90bbed06c6214bc9d5168fcb", 
      "version": "1.4.15",   
      "plugins": []      
    }).then((AMap)=>{
      this.AMap=AMap;
          var map = new AMap.Map('container',{
          center:[116.397428, 39.90923],
          zoom:11,
      });
      this.map=map
    }).catch(e => {
        console.log(e);
    })

      // Code that will run only after the
      // entire view has been rendered
  },
  
  methods:{
    drawLine(){
        // 基于准备好的dom，初始化echarts实例
        let myChart = this.$echarts.init(document.getElementById('myChart'))
        // 绘制图表
        myChart.setOption({
            title: { text: '在Vue中使用echarts' },
            tooltip: {},
            xAxis: {
                data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
            },
            yAxis: {},
            series: [{
                name: '销量',
                type: 'bar',
                data: [5, 20, 36, 10, 10, 20]
            }]
        });
    },
    readFile(file){
        if (window.FileReader && file) {
          console.log(file)
          let reader = new FileReader();
          reader.onload = (e) => {
          console.log(e.target.result);
          this.fileContent = e.target.result;
          
          let tempData=JSON.parse(this.fileContent)
          
          if(tempData instanceof Array && tempData.length>0
            && tempData[0]['Longitude']&& tempData[0]['Latitude']){
            console.log('if')
            this.lnglats=tempData
            this.drawPath()
            
          }
          else{
            console.log('else')
          }
        };
      reader.readAsText(file.raw);
      }
    },
    drawPath(){
      let gps=[]
      console.log(this.lnglats)
      for(let e of this.lnglats){
        gps.push(new this.AMap.LngLat(e['Longitude'],e['Latitude']))
      }
      console.log(gps)
      this.AMap.convertFrom(gps, 'gps',(status, result)=> {
        if (result.info === 'ok') {
          let lnglats = result.locations; // Array.<LngLat>
          console.log(lnglats)
          // 创建折线实例
          let polyline = new this.AMap.Polyline({
            path: lnglats,  
            borderWeight: 2, // 线条宽度，默认为 1
            strokeColor: 'red', // 线条颜色
            lineJoin: 'round' // 折线拐点连接处样式
          });

          // 将折线添加至地图实例
          this.map.add(polyline);
          this.map.setCenter(lnglats[500]);
        }
      });
    }
  },
}

</script>
<style scoped>
  h3{
    margin: 20px 0 0;
  }
  ul{
    list-style-type: none;
    padding: 0;
  }
  li{
    display: inline-block;
    margin: 0 10px;
  }
  a{
    color: cyan;
  }
  textarea{
    width: 60vw;
    height: 55vh;
    position: fixed;
    left: 20%;
    top:40% ;
  }
  #container {
    width: 500px;
    height: 250px;
    position:fixed;
    top:13%;
    left:30%;
  }
  #myChart {
    width: 300px;
    height: 300px;
   
  }
</style>>

