<template>
<!-- cesium容器 -->
<div id="cesiumContainer" style="position:relative;">
  <!-- 左右上下卷帘的分割条 -->
  <div id="lrSlider" ref="lrSlider" class="shutter-slider" style="display:none"></div>
  <div id="tbSlider" ref="tbSlider" class="shutter-slider" style="display:none"></div>
</div>
</template>
 
<script>
// https://www.freesion.com/article/14861291934/
export default {
  data() {
    return {
      viewer: null,
      handler: null,
      moveActive: false,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const Cesium = this.cesium;
      this.viewer = new Cesium.Viewer("cesiumContainer", {
        shouldAnimate: true,
        infoBox: false,
        selectionIndicator: false,
      });
      // 添加图层
      const layer1 = new Cesium.UrlTemplateImageryProvider({
        url: "http://webrd02.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}",
        minimumLevel: 1,
        maximumLevel: 18,
      });
      const layer2 = new Cesium.UrlTemplateImageryProvider({
        url: "https://webst02.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}",
        minimumLevel: 1,
        maximumLevel: 18,
      });

      const layers = this.viewer.imageryLayers;
      const earthAtRight = layers.addImageryProvider(layer1);
      const earthAtLeft = layers.addImageryProvider(layer2);

      earthAtLeft.splitDirection = Cesium.SplitDirection.LEFT;
      earthAtRight.splitDirection = Cesium.SplitDirection.RIGHT;

      this.initRollerShutter();
    },
    /**
     * 初始化卷帘地球位置
     */
    initRollerShutter() {
      const Cesium = this.cesium;
      if (this.handler) {
        this.handler.destroy();
        this.handler = null;
      }
      const slider = this.$refs.slider;
      const container = this.$refs.container;
      const line = this.$refs.line;
      if (!slider && !container) return;
      this.handler = new Cesium.ScreenSpaceEventHandler(slider);
      this.viewer.scene.splitPosition =
        slider.offsetLeft / container.offsetWidth;

      const move = (movement) => {
        if (!this.moveActive) return;
        const relativeOffset = movement.endPosition.x;
        const splitPosition =
          (slider.offsetLeft + relativeOffset) / container?.offsetWidth;
        slider.style.left = `${100.0 * splitPosition}%`;
        line.style.left = `${100.0 * splitPosition}%`;
        
        this.viewer.scene.splitPosition = splitPosition; // 设置卷帘左右分区范围(0-1)之间
      };

      this.handler.setInputAction(() => {
        this.moveActive = true;
      }, Cesium.ScreenSpaceEventType.LEFT_DOWN);
      this.handler.setInputAction(() => {
        this.moveActive = true;
      }, Cesium.ScreenSpaceEventType.PINCH_START);

      this.handler.setInputAction(move, Cesium.ScreenSpaceEventType.MOUSE_MOVE);
      this.handler.setInputAction(move, Cesium.ScreenSpaceEventType.PINCH_MOVE);

      this.handler.setInputAction(() => {
        this.moveActive = false;
      }, Cesium.ScreenSpaceEventType.LEFT_UP);
      this.handler.setInputAction(() => {
        this.moveActive = false;
      }, Cesium.ScreenSpaceEventType.PINCH_END);
    },
  },
};
</script>
  
<style scoped lang="scss">
$color: #409eff;
.container {
  width: 100%;
  height: 100%;
  position: relative;
  #cesiumContainer {
    width: 100%;
    height: 100%;
  }
  .slider-view {
    .slider-line {
      position: absolute;
      top: 0;
      left: 50%;
      transform: translateX(-50%);
      background-color: $color;
      width: 5px;
      height: 100%;
      z-index: 2;
      pointer-events: none;
    }
    .slider {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background-color: $color;
      display: flex;
      align-items: center;
      justify-content: center;
      color: #fff;
      z-index: 3;
      font-size: 24px;
      &:hover {
        cursor: ew-resize;
      }
    }
  }
}
</style>