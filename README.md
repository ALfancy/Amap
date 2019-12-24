# Amap
高德地图demo


//百度坐标转高德（传入经度、纬度）
function bgps_gps(bd_lng, bd_lat) {
    var X_PI = Math.PI * 3000.0 / 180.0;
    var x = bd_lng - 0.0065;
    var y = bd_lat - 0.006;
    var z = Math.sqrt(x * x + y * y) - 0.00002 * Math.sin(y * X_PI);
    var theta = Math.atan2(y, x) - 0.000003 * Math.cos(x * X_PI);
    var gg_lng = z * Math.cos(theta);
    var gg_lat = z * Math.sin(theta);
    return {lng: gg_lng, lat: gg_lat}
}
//高德坐标转百度（传入经度、纬度）
function gps_bgps(gg_lng, gg_lat) {
    var X_PI = Math.PI * 3000.0 / 180.0;
    var x = gg_lng, y = gg_lat;
    var z = Math.sqrt(x * x + y * y) + 0.00002 * Math.sin(y * X_PI);
    var theta = Math.atan2(y, x) + 0.000003 * Math.cos(x * X_PI);
    var bd_lng = z * Math.cos(theta) + 0.0065;
    var bd_lat = z * Math.sin(theta) + 0.006;
    return {
        bd_lat: bd_lat,
        bd_lng: bd_lng
    };
}







/**
 * Created by Wandergis on 2015/7/8.
 * 提供了百度坐标（BD09）、国测局坐标（火星坐标，GCJ02）、和WGS84坐标系之间的转换
 */
//UMD魔法代码
// if the module has no dependencies, the above pattern can be simplified to
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD. Register as an anonymous module.
    define([], factory);
  } else if (typeof module === 'object' && module.exports) {
    // Node. Does not work with strict CommonJS, but
    // only CommonJS-like environments that support module.exports,
    // like Node.
    module.exports = factory();
  } else {
    // Browser globals (root is window)
    root.coordtransform = factory();
  }
}(this, function () {
  //定义一些常量
  var x_PI = 3.14159265358979324 * 3000.0 / 180.0;
  var PI = 3.1415926535897932384626;
  var a = 6378245.0;
  var ee = 0.00669342162296594323;
  /**
   * 百度坐标系 (BD-09) 与 火星坐标系 (GCJ-02)的转换
   * 即 百度 转 谷歌、高德
   * @param bd_lon
   * @param bd_lat
   * @returns {*[]}
   */
  var bd09togcj02 = function bd09togcj02(bd_lon, bd_lat) {
    var bd_lon = +bd_lon;
    var bd_lat = +bd_lat;
    var x = bd_lon - 0.0065;
    var y = bd_lat - 0.006;
    var z = Math.sqrt(x * x + y * y) - 0.00002 * Math.sin(y * x_PI);
    var theta = Math.atan2(y, x) - 0.000003 * Math.cos(x * x_PI);
    var gg_lng = z * Math.cos(theta);
    var gg_lat = z * Math.sin(theta);
    return [gg_lng, gg_lat]
  };

  /**
   * 火星坐标系 (GCJ-02) 与百度坐标系 (BD-09) 的转换
   * 即谷歌、高德 转 百度
   * @param lng
   * @param lat
   * @returns {*[]}
   */
  var gcj02tobd09 = function gcj02tobd09(lng, lat) {
    var lat = +lat;
    var lng = +lng;
    var z = Math.sqrt(lng * lng + lat * lat) + 0.00002 * Math.sin(lat * x_PI);
    var theta = Math.atan2(lat, lng) + 0.000003 * Math.cos(lng * x_PI);
    var bd_lng = z * Math.cos(theta) + 0.0065;
    var bd_lat = z * Math.sin(theta) + 0.006;
    return [bd_lng, bd_lat]
  };

  /**
   * WGS84转GCj02
   * @param lng
   * @param lat
   * @returns {*[]}
   */
  var wgs84togcj02 = function wgs84togcj02(lng, lat) {
    var lat = +lat;
    var lng = +lng;
    if (out_of_china(lng, lat)) {
      return [lng, lat]
    } else {
      var dlat = transformlat(lng - 105.0, lat - 35.0);
      var dlng = transformlng(lng - 105.0, lat - 35.0);
      var radlat = lat / 180.0 * PI;
      var magic = Math.sin(radlat);
      magic = 1 - ee * magic * magic;
      var sqrtmagic = Math.sqrt(magic);
      dlat = (dlat * 180.0) / ((a * (1 - ee)) / (magic * sqrtmagic) * PI);
      dlng = (dlng * 180.0) / (a / sqrtmagic * Math.cos(radlat) * PI);
      var mglat = lat + dlat;
      var mglng = lng + dlng;
      return [mglng, mglat]
    }
  };

  /**
   * GCJ02 转换为 WGS84
   * @param lng
   * @param lat
   * @returns {*[]}
   */
  var gcj02towgs84 = function gcj02towgs84(lng, lat) {
    var lat = +lat;
    var lng = +lng;
    if (out_of_china(lng, lat)) {
      return [lng, lat]
    } else {
      var dlat = transformlat(lng - 105.0, lat - 35.0);
      var dlng = transformlng(lng - 105.0, lat - 35.0);
      var radlat = lat / 180.0 * PI;
      var magic = Math.sin(radlat);
      magic = 1 - ee * magic * magic;
      var sqrtmagic = Math.sqrt(magic);
      dlat = (dlat * 180.0) / ((a * (1 - ee)) / (magic * sqrtmagic) * PI);
      dlng = (dlng * 180.0) / (a / sqrtmagic * Math.cos(radlat) * PI);
      var mglat = lat + dlat;
      var mglng = lng + dlng;
      return [lng * 2 - mglng, lat * 2 - mglat]
    }
  };

  var transformlat = function transformlat(lng, lat) {
    var lat = +lat;
    var lng = +lng;
    var ret = -100.0 + 2.0 * lng + 3.0 * lat + 0.2 * lat * lat + 0.1 * lng * lat + 0.2 * Math.sqrt(Math.abs(lng));
    ret += (20.0 * Math.sin(6.0 * lng * PI) + 20.0 * Math.sin(2.0 * lng * PI)) * 2.0 / 3.0;
    ret += (20.0 * Math.sin(lat * PI) + 40.0 * Math.sin(lat / 3.0 * PI)) * 2.0 / 3.0;
    ret += (160.0 * Math.sin(lat / 12.0 * PI) + 320 * Math.sin(lat * PI / 30.0)) * 2.0 / 3.0;
    return ret
  };

  var transformlng = function transformlng(lng, lat) {
    var lat = +lat;
    var lng = +lng;
    var ret = 300.0 + lng + 2.0 * lat + 0.1 * lng * lng + 0.1 * lng * lat + 0.1 * Math.sqrt(Math.abs(lng));
    ret += (20.0 * Math.sin(6.0 * lng * PI) + 20.0 * Math.sin(2.0 * lng * PI)) * 2.0 / 3.0;
    ret += (20.0 * Math.sin(lng * PI) + 40.0 * Math.sin(lng / 3.0 * PI)) * 2.0 / 3.0;
    ret += (150.0 * Math.sin(lng / 12.0 * PI) + 300.0 * Math.sin(lng / 30.0 * PI)) * 2.0 / 3.0;
    return ret
  };

  /**
   * 判断是否在国内，不在国内则不做偏移
   * @param lng
   * @param lat
   * @returns {boolean}
   */
  var out_of_china = function out_of_china(lng, lat) {
    var lat = +lat;
    var lng = +lng;
    // 纬度3.86~53.55,经度73.66~135.05 
    return !(lng > 73.66 && lng < 135.05 && lat > 3.86 && lat < 53.55);
  };

  return {
    bd09togcj02: bd09togcj02,
    gcj02tobd09: gcj02tobd09,
    wgs84togcj02: wgs84togcj02,
    gcj02towgs84: gcj02towgs84
  }
}));





/*
 * @Author: Fred
 * @Date: 2019-08-15 15:16:51
 * @LastEditors: Fred
 * @LastEditTime: 2019-08-15 16:55:10
 * @Description: file content
 */
import axios from '../../http/axios';

export default {
  state: {
    map: {},
    wsView: {
      center: [106.24585, 30.349281],
      zoom: 11.6,
    },
    mapStyle: 'amap://styles/28006c60645861718fa097bcca30116a',
    tileLayer: new AMap.TileLayer({
      visible: true,
      zIndex: 0,
    }),
    building: new AMap.Buildings({
      zooms: [16, 18],
      zIndex: 10,
      heightFactor: 2,
      zIndex: 1,
    }),
  },
  mutations: {
    mapInit(state, domID = 'map-container', option = state.wsView) {
      state.map = new AMap.Map(domID, {
        zoom: option.zoom,
        center: option.center,
        resizeEnable: true,
        rotateEnable: true,
        pitchEnable: true,
        viewMode: '3D',
        buildingAnimation: true,
        expandZoomRange: true,
        layers: [state.tileLayer, state.building],
        mapStyle: state.mapStyle,
        pitch: 57,
      });
    },
    // 返回初始位置
    zoomToOrigin(state) {
      state.map.setZoomAndCenter(state.wsView.zoom, state.wsView.center);
    },
    // 清除图层
    clearMap(state) {
      state.map.clearMap();
      this.commit('zoomToOrigin');
      this.dispatch('getBorderData');
      state.map.setZoom(11.6);
    },
    // 到指定Zoom
    GoZoom(state, zoom) {
      state.map.setZoom(zoom);
    },
    // 添加行政区边框
    addBorder(state, geoData) {
      const geojson = new AMap.GeoJSON({
        geoJSON: geoData,
        getPolygon(geojson, lnglats) {
          return new AMap.Polygon({
            bubble: true,
            strokeWeight: 1,
            path: lnglats,
            fillOpacity: 0.15,
            fillColor: '#1b9ea9',
            strokeColor: '#1b9ea9',
            strokeWeight: 4,
          });
        },
      });
      geojson.setMap(state.map);
    },
  },
  actions: {
    getBorderData({ commit }) {
      axios.get('./geo/511622.json').then((res) => {
        commit('addBorder', res.data);
      });
    },
  },
};


this.ssMarker = ssMarker.map(item => {
        let jd = '';
        let wd = '';
        if (item.jdLocation) {
          jd = item.jdLocation.split(',')[0];
          wd = item.jdLocation.split(',')[1];
          return new AMap.Marker({
            map: this.map,
            icon: new AMap.Icon({
              size: new AMap.Size(50, 60),
              imageSize: new AMap.Size(50, 60),
              image: require('@img/map/tx_ss_point.png')
            }),
            topWhenClick: true,
            position: new AMap.LngLat(jd, wd),
            extData: item
          })
        }
      })
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
  # Cesium简易开发攻略
> 编者语：其实我对于Cesium、Three.js、WebGL的理解还是非常短浅的。但是这个手册也只能说是我做Cesium项目的感受和经验而已，希望这个手册能为你减少一些开发上的负担。
## 什么是Cesium
[Cesium](https://cesiumjs.org/)是一个由微软开发的基于`WebGL`的`开源三维可视化GIS库`。但是[官方API](https://cesium.com/docs/cesiumjs-ref-doc/)对于开发者而言非常的不友好。一方面是因为作为一名中国开发者看英文API本身就有点麻烦，另一方面是因为Cesium是一个`GIS`库，因此在API中渗透了许多有关`地理信息技术`的名词。当然目前Cesium官方已经做出了一份[官方的教学文档](https://cesium.com/docs/tutorials/getting-started/)，这比我们当初自己啃着API方便了很多，由于我个人没有去研读过这份新出的文档，所以也不能对此做过多描述，但是我还是认为你可以去研读一下这篇文档，对于你的开发一定会有所帮助！当然在国内有个叫`Cesiumlab`的工作室，他们是专门研究三维地理可视化的，你可以关注一下他们的[简书帐号](https://www.jianshu.com/u/c5b90bde7370)。同时他们自己也有做一些有利于Cesium开发的小工具和软件，如果感兴趣也可以下载。
## 友情链接
* [Cesium官方API](https://cesium.com/docs/cesiumjs-ref-doc/)
* [Cesium官方教学](https://cesium.com/docs/tutorials/getting-started/)
* [Cesiumlab教程](https://www.jianshu.com/p/31c3b55a21eb)
* [geojson获取](http://datav.aliyun.com/tools/atlas/)
## 开发攻略
### 开发之前
在项目确定需要使用Cesium之后，需要确认以下事项：
* 分辨率  
Cesium是基于WebGL开发的，因此实际呈现的形式将会是`canvas`，因此我不建议在使用Cesium的页面做任何有关`自适应`的操作，这将会导致Cesium中的摄像机位置产生偏移；更不建议做类似`transform:scaleX(0.8)`这样的操作。
* 显卡性能  
canvas的性能是由其像素点个数决定的，canvas像素点越多，所需要的显卡性能越高（主要是`显存`）。当然项目经理很可能向你要一个Cesium的`显卡配置`，由于我也没有对Cesium所需的性能做过精确的测试，这里给出一个我们`之前项目测试的结果`供你参考。

| 分辨率 | 显卡 | 显存 | 结果 |
|:----:|:----:|:----:|:----:|
|1920*1080|NVIDA GTX650（前端组现有的开发显卡）|1G|基本流畅|
|1920*1080|NVIDA GTX1050Ti（公司大屏和华硕笔记本）|4G|流畅|
|4800*1080（[广西大屏](http://10.1.20.110:8050/#/curing)）|NVIDA GTX650（前端组现有的开发显卡）|1G|卡顿（甚至会造成崩溃）|
|4800*1080（[广西大屏](http://10.1.20.110:8050/#/curing)）|NVIDA GTX1050Ti（公司大屏和华硕笔记本）|4G|流畅（有时会卡一下）|
> 当你开发`高分辨率`的Cesium项目时，建议`不要使用`GTX650这块显卡，因为这块显卡的显存只有1G，当系统显存负荷过高会导致`显卡崩溃`，从而导致`浏览器崩溃`，非常影响开发效率。但是如果使用集成显卡，会使vscode和系统反应速度很慢，因为他实际消耗的是CPU性能。
* 图层服务  
Cesium原生图层的服务来自于`国外`，因此实际请求图层效率较低，如果客户无法提供图层服务，建议使用其他的第三方图层，但是第三方图层只能`应用他们的配色`，并不能像高德地图那样自由搭配。这里提供几个我们之前收集的图层，当然可能有遗漏，你也可以在这里做补充。

| 图层名称 | 图层类型 | 地址 | Cesium图层实例 |
|:----:|:----:|:----:|:----:|
|蓝黑色底图图层|ArcGis|https://map.geoq.cn/arcgis/rest/services/ChinaOnlineStreetPurplishBlue/MapServer|new Cesium.ArcGisMapServerImageryProvider
|谷歌地图图层|瓦片图层|http://www.google.cn/maps/vt?lyrs=s&x={x}&y={y}&z={z}| new Cesium.UrlTemplateImageryProvider|
### 开始开发
#### 开发顺序
1. 初始化Cesium（创建viewer）
  Cesium中有一个很重要的概念叫做`viewer`，你可以认为它是一个地图实例，当`viewer`被创建的时候就意味着Cesium初始化成功了。  
  下面提供一种方式能够直接建立没有Cesium自带控件的地图。  
  你可以在创建地图实例时直接引入`imageryProvider`和`terrainProvider`，也可以在viewer生成之后利用`addImageryProvider`和`addTerrainProvider`方法来实现
  ```
  let viewer = new Cesium.Viewer('dom',{
    scene3DOnly: true,
    selectionIndicator: false,
    baseLayerPicker: false,
    shouldAnimate: true,
    fullscreenButton: false,
    animation: false,
    baseLayerPicker: false,
    homeButton: false,
    geocoder: false,
    timeline: false,
    sceneModePicker: false,
    navigationHelpButton: false,
    infoBox: false,
    skyBox: false,
  })
  ```
  2. 初始化摄像机位置  
  摄像机是Cesium中一个很重要的概念，即表示你的视角位于什么位置。移动摄像机可以用`flyTo`、`setView`、`lookAt`等方法，其中`flyTo`会有一个比较圆滑的转场动画。  
  下面就是一个flyTo的例子
  ```
  let homeCameraView = {
    destination: Cesium.Cartesian3.fromDegrees(100.70660021415233, 26.412919435895407, 4232.785085240076),
    orientation: {
      heading: 5.979873817691248,
      pitch: -0.32513264287588606,
      roll: 6.282342823490078,
    },
    duration: 6,
    pitchAdjustHeight: 6000,
  }
  viewer.camera.flyTo(homeCameraView)
  ```  
  当然有时候你需要旋转到某个特定的角度，这里提供一个我自己写的方法，能够获取到相机位置，并打印到控制台。
  ```
    // 其中viewer需要使用地图创建出来的viewer
    let getCamera = () => {
      console.log("已开启Cesium相机监控");
      let container = document.querySelector(".cesium-viewer");
      let button = document.createElement("button");
      button.appendChild(document.createTextNode("获取相机位置"));
      button.onclick = () => {
        console.log("经度(lng):" + Cesium.Math.toDegrees(this.viewer.camera.positionCartographic.longitude));
        console.log("纬度(lat):" + Cesium.Math.toDegrees(this.viewer.camera.positionCartographic.latitude));
        console.log("高度(height):" + this.viewer.camera.positionCartographic.height);
        // 输出的是弧度
        console.log("roll:" + this.viewer.camera.roll);
        // 输出的是弧度
        console.log("pitch:" + this.viewer.camera.pitch);
        // 输出的是弧度
        console.log("heading:" + this.viewer.camera.heading);
      }
      button.style.position = "absolute";
      button.style.top = "20px";
      button.style.left = "20px";
      button.style.padding = "5px 10px";
      console.log(container);
      container.appendChild(button);
    },
  ```
  3. 设置Entity  
  Cesium中所有的覆盖物都被叫做Entity，这也是我们平时开发过程中用到最多的API。这里列举几个常用的Entitiy。  
  * polyline
  * polygon
  * label
  * billboard  
  由于Entity种类和使用方式过多，其余的Entity还请你去API中寻找使用的方法。这里举一个最简单的例子。
  ```
    let entitiy = {
      id:'0',
      name:'test',
      position:[120,30,2000]
    }
    let billboard = this.viewer.entities.add({
      id: entitiy.id,
      name: entitiy.name,
      type:"Camera",
      position: Cesium.Cartesian3.fromDegrees(
        entitiy.position[0], entitiy.position[1], entitiy.position[2]
      ),
      billboard: {
        // heightReference: Cesium.HeightReference.CLAMP_TO_GROUND,
        image: require("@img/camera.png"),
        verticalOrigin: 0,
        width: 32,
        height: 32,
        pixelOffset:new Cesium.Cartesian2(0,-10),
      },
      label: {
        // heightReference:Cesium.HeightReference.CLAMP_TO_GROUND,
        text: entitiy.name,
        font: '140px Microsoft YaHei',
        scale: 0.1,
        style: Cesium.LabelStyle.FILL,
        outlineWidth: 0,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
        pixelOffset: new Cesium.Cartesian2(0, -22),
      }
    })
  ```  
  我个人比较建议将设置Entity的方法以`业务`来封装，而不是以功能来封装，这样能够大大提高代码可读性。就像上方的代码一样，我只将设置`Camera`作为一个方法进行封装，而不是把它认为是一个billboard.  
  4. 设置事件  
  WebGL中的所有事件都是通过`射线(Ray)`来实现的。当你鼠标点击canvas上的一个点之后，以这个点为起点向屏幕内部发射一条`射线`，然后就能获取所有触碰到的物体。Cesium也不例外，在Cesium中这个“射线”被称为picker。
  ```
  // 这个方法需要绑定到Cesium的容器Dom中
  onMapClick(e) {
    this.pick = null;
    let windowPosition = new Cesium.Cartesian2(e.offsetX, e.offsetY);
    var picked = this.viewer.scene.pick(windowPosition);
    if (Cesium.defined(picked)) {
      var id = Cesium.defaultValue(picked.id, picked.primitive.id);
      if (id instanceof Cesium.Entity) {
        this.pick = id;
        if (id.type) {
          this[`on${id.type}Click`]();
        } else {
          console.error("Cesium点击事件未绑定");
        }
        return;
      }
    }
  },
  ```  
  5. CZML  
  CZML是Cesium对于Entity的整合文件，你可以在CZML中制作所有Entity API能够达到的效果。  
  CZML是一个类JSON文件，实际处理过程中，你可以直接当做JSON处理。所以从理论上说，你甚至可以让后台拼接出一个CZML加入到dataSource中，这样也可以实现效果。  
  CZML相较于直接通过js写Entity有个巨大的区别。CZML可以记录Entity在每个时间点的状态，用高端的话讲就是将三维转化成了四维。通过记录不同的时间点的状态，你就可以制作酷炫的动画。  
  CZML的格式及语法规范可以在[这篇文章](https://www.cnblogs.com/laixiangran/p/4998529.html)中查看，但是这篇文章是全英文教程，而且可能不够新了，如果你觉得不满足你的开发，你可以去[Cesium的官网](https://cesiumjs.org/)或者[Cesium的官方github](https://github.com/AnalyticalGraphicsInc/czml-writer/wiki)中去寻找。

  以上是对于Cesium的简单介绍，其中的所有例子都可以在[程海的DEMO](http://10.1.20.209:8090/chenghaiin)中找到,你也可以在git上找到[源代码](http://10.1.70.185/yunnan/chenghai)。  
  我在这个文档中所说到的开发流程只是Cesium中的冰山一角，Cesium可能还有许多有意思的妙用，比如将`Three.js`混入其中之类的，还需要机智的你去发现。
