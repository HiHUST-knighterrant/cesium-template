<template>
  <div class="container">
    <div id="cesiumContainer"></div>
  </div>
</template>
 
<script>
export default {
  data() {
    return {
      viewer: null,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const Cesium = this.cesium;
      Cesium.Ion.defaultAccessToken = process.env.VUE_APP_TOKEN;
      const tiandituTk = process.env.VUE_APP_TIANDITU_KEY
      this.viewer = new Cesium.Viewer("cesiumContainer", {
        imageryProvider: new Cesium.ArcGisMapServerImageryProvider({
          url: "https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer",
        }),
        terrainProvider: new Cesium.CesiumTerrainProvider({
          //加载火星在线地形
          url: "http://data.marsgis.cn/terrain",
        }),
        // imageryProvider: new Cesium.UrlTemplateImageryProvider({
        //   url: "https://t7.tianditu.gov.cn/img_w/wmts?service=WMTS&version=1.0.0&request=GetTile&tilematrix={z}&layer=img&style=default&tilerow={y}&tilecol={x}&tilematrixset=w&format=tiles&tk=" + tiandituTk,
        // }),
        shouldAnimate: true,
        infoBox: false,
        selectionIndicator: false,
      });
      // 添加地形数据
      // this.viewer.terrainProvider = Cesium.createWorldTerrain();
      //设置贴地效果
      this.viewer.scene.globe.depthTestAgainstTerrain = false;
      this.start();
    },
    /**
     * 开始
     */
    start() {
      const Cesium = this.cesium;
      var tileset = this.viewer.scene.primitives.add(new Cesium.Cesium3DTileset({
        // url: process.env.VUE_APP_PUBLIC_URL + "/Vue/Models/3DTiles/monomerEditing/tileset.json", //数据地址
        url: '//data.mars3d.cn/3dtiles/qx-xuexiao/tileset.json',
        maximumScreenSpaceError: 2,  //最大的屏幕空间误差
        maximumNumberOfLoadedTiles: 1000, //最大加载瓦片个数
        // modelMatrix: m,//形状矩阵
      }));
      this.viewer.flyTo(tileset);

      const _url = process.env.VUE_APP_PUBLIC_URL + "/Vue/Models/3DTiles/monomerEditing/dth-xuexiao-fd.json"
      Cesium.Math.setRandomNumberSeed(0); //设置随机数种子
      const promise = Cesium.GeoJsonDataSource.load(_url, {
        // stroke: Cesium.Color.WHITE,
        // fill: Cesium.Color.BLUE.withAlpha(0.3), //注意：颜色必须大写，即不能为blue
        // strokeWidth: 5,
        clampToGround: true
      });
      promise.then((dataSource) => {
        this.viewer.dataSources.add(dataSource);
        let entities = dataSource.entities.values;
        let colorHash = {};
        for (let i = 0; i < entities.length; i++) {
          let entity = entities[i];
          console.log(entity)
          // console.log(entity)
          // let name = entity.name; //geojson里面必须得有一个name属性，entity.name对应
          // let color = colorHash[name]; //可以使两个同名要素使用同一种颜色。。。
          // if (!color) {
          //   color = Cesium.Color.fromRandom({
          //     alpha: 1.0
          //   });
          //   colorHash[name] = color;
          // }
          // entity.polygon.material = color; //设置要素颜色
          // entity.polygon.outline = false;
          // entity.polygon.height = 10000; //要素距离地面的高度
          // entity.polygon.heightReference = Cesium.HeightReference.CLAMP_TO_GROUND
          // entity.polygon.extrudedHeightReference = Cesium.HeightReference.RELATIVE_TO_GROUND
          // entity.polygon.height = 400.0
          // entity.polygon.extrudedHeight = 0.0;
          entity.polygon.heightReference = Cesium.HeightReference.RELATIVE_TO_GROUND;  // 贴地
          entity.polygon.height = 0;  // 距地高度0米
          entity.polygon.extrudedHeightReference = Cesium.HeightReference.RELATIVE_TO_GROUND; //拉伸
          // entity.polygon.extrudedHeight = entity.properties[exHeightFieldName]; // 拉伸高度
          entity.polygon.extrudedHeight = 100.0; // 拉伸高度
          entity.polygon.outline = true;
          entity.polygon.outlineColor = Cesium.Color.BLACK;
          entity.polygon.classificationType = Cesium.ClassificationType.TERRAIN
        }
      });
      // Cesium.GeoJsonDataSource.load(_url, {
      //   stroke: Cesium.Color.WHITE,
      //   fill: Cesium.Color.BLUE.withAlpha(0.3), //注意：颜色必须大写，即不能为blue
      //   strokeWidth: 5,
      //   clampToGround: true
      // }).then(res => {
      //   console.log("res", res);
      //   this.viewer.dataSources.add(res);
      // });

    },
  },
};
</script>
  
<style scoped lang="scss">
.container {
  width: 100%;
  height: 100%;

  #cesiumContainer {
    width: 100%;
    height: 100%;
  }
}
</style>