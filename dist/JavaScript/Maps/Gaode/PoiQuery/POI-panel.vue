<template>
  <el-drawer
    title="操作面板"
    :visible.sync="drawer"
    size="500px"
    direction="rtl"
    :before-close="handleClose"
  >
    <div class="POI-panel">
      <el-tabs v-model="panelType" @tab-click="tabClick">
        <el-tab-pane
          v-for="(item, index) in panelTypeArr"
          :key="index"
          :label="item.label"
          :name="item.value"
        ></el-tab-pane>
      </el-tabs>
      <!-- 兴趣点 Start -->
      <section class="pois" v-if="panelType == '1'">
        <div class="form-view">
          <div class="form-item">
            <div class="form-label">范围：</div>
            <div class="form-value">
              <el-radio-group @change="search" v-model="range" size="mini">
                <el-radio label="city">城市</el-radio>
                <el-radio label="all">全国</el-radio>
              </el-radio-group>
            </div>
          </div>
          <div class="form-item">
            <div class="form-label">城市：</div>
            <div class="form-value">
              <el-cascader
                size="mini"
                clearable
                @change="search"
                :disabled="range == 'all'"
                v-model="city"
                :props="{
                  children: 'districtList',
                  label: 'name',
                  value: 'adcode',
                }"
                style="width: 100%"
                placeholder="城市选择..."
                :options="cityOptions"
              ></el-cascader>
            </div>
          </div>
          <div class="form-item multiple">
            <div class="form-label">兴趣点类别：</div>
            <div class="form-value">
              <el-select
                class="select"
                size="mini"
                v-model="searchType"
                multiple
                @change="search"
                placeholder="请选择兴趣类别（可多选）"
              >
                <el-option
                  v-for="item in placeSearchType"
                  :key="item"
                  :label="item"
                  :value="item"
                >
                </el-option>
              </el-select>
            </div>
          </div>
          <div class="form-item multiple">
            <div class="form-label">是否聚合：</div>
            <div class="form-value">
              <el-switch
                size="mini"
                v-model="polymerization"
                active-color="#13ce66"
                inactive-color="#ff4949"
                @change="search"
              >
              </el-switch>
            </div>
          </div>
          <div class="form-item search">
            <el-input
              size="mini"
              placeholder="请输入搜索关键词"
              v-model.trim="keyword"
              clearable
              @change="search"
            >
            </el-input>
            <el-button
              class="button"
              :disabled="keyword == ''"
              type="primary"
              size="mini"
              icon="el-icon-search"
              @click="search"
              >搜索</el-button
            >
          </div>
        </div>
        <div class="main">
          <div class="item-view" ref="poisList" v-loading="loading">
            <div
              class="pois-item"
              v-for="(item, index) in poisList"
              :key="item.id + index"
              @click="poisClick(item)"
            >
              <div class="pois-item-image">
                <el-image
                  class="image"
                  v-if="item.photos.length != 0"
                  :src="item.photos[0].url"
                  fit="cover"
                >
                  <div slot="error" class="image-slot">
                    <i class="el-icon-picture-outline"></i>
                  </div>
                </el-image>
                <el-image v-else class="image" src="" fit="cover">
                  <div slot="error" class="image-slot">
                    <i class="el-icon-picture-outline"></i>
                  </div>
                </el-image>
              </div>
              <div class="pois-item-text">
                <h3>{{ item.name }}</h3>
                <p>
                  电话：
                  <template v-if="item.tel">{{ item.tel }}</template>
                  <template v-else>-</template>
                </p>
                <p>
                  类目：
                  {{ item.type }}
                </p>
                <p>
                  地址：
                  {{ item.pname }}{{ item.cityname }}{{ item.adname
                  }}{{ item.address }}
                </p>
              </div>
            </div>
          </div>
          <div class="pagination">
            <el-pagination
              hide-on-single-page
              layout="prev, pager, next"
              @current-change="currentPaginationChange"
              :total="total"
              :page-size="pageSize"
              :current-page="pageIndex"
            />
          </div>
        </div>
      </section>
      <!-- 兴趣点 End -->
      <!-- 路线规划与导航 Start -->
      <section class="navigation" v-if="panelType == '2'">
        <div class="form-view">
          <el-form ref="form" label-width="100px">
            <el-form-item label="起始位置：">
              <el-input
                size="mini"
                placeholder="请输入起始位置"
                id="startPosition"
                v-model.trim="startPosition"
              >
              </el-input>
            </el-form-item>
            <el-form-item label="终点位置：">
              <el-input
                size="mini"
                placeholder="请输入终点位置"
                id="endPosition"
                v-model.trim="endPosition"
              >
              </el-input>
            </el-form-item>
            <el-form-item label="类型：">
              <el-radio-group
                v-model="navigationType"
                :disabled="
                  JSON.stringify(startPositionResult) == '{}' ||
                  JSON.stringify(endPositionResult) == '{}'
                "
                size="mini"
                @change="navigationTypeChange"
              >
                <el-radio-button
                  v-for="(item, index) in navigationTypeArr"
                  :key="index"
                  :label="item.value"
                  >{{ item.label }}</el-radio-button
                >
              </el-radio-group>
            </el-form-item>
          </el-form>
        </div>
        <div id="panel" class="steps-view"></div>
      </section>
      <!-- 路线规划与导航 End -->
    </div>
  </el-drawer>
</template>
 
<script>
// https://blog.csdn.net/u014556081/article/details/113185855 路线规划与导航
import Utils from "@/common/cesium/Utils.js";
import GaodeMap from "@/common/cesium/Map/Gaode";

export default {
  props: {
    show: {
      type: Boolean,
      default: () => {
        return false;
      },
    },
    option: {
      type: Object,
      default: () => {
        return {};
      },
    },
  },
  data() {
    return {
      drawer: false,
      _GaodeMap: null,
      _Utils: null,
      loading: false,

      panelTypeArr: [
        {
          label: "兴趣点",
          value: "1",
        },
        {
          label: "路线规划与导航",
          value: "2",
        },
      ],
      panelType: "1", //面板类型tab
      range: "city", //范围
      cityOptions: [], //城市数据
      city: ["210000", "210100"], //城市
      placeSearchType: [], //兴趣类别
      searchType: ["餐饮服务", "商务住宅", "生活服务"], //选择的类别
      polymerization: false, //是否聚合
      keyword: "", //搜索关键词

      pageIndex: 1, //页码
      pageSize: 20, //页数量
      total: 0, //总数

      poisList: [], //兴趣点搜索结果

      startPosition: "", //起始输入位置数据
      endPosition: "", //终点输入位置数据
      startPositionResult: {}, //起始结果位置数据
      endPositionResult: {}, //终点结果位置数据
      navigationTypeArr: [
        {
          label: "驾车",
          value: "1",
        },
        {
          label: "公交",
          value: "2",
        },
        {
          label: "骑行",
          value: "3",
        },
        {
          label: "步行",
          value: "4",
        },
      ],
      navigationType: "",
      //路线位置集合
      routeLocationArr: [],
    };
  },
  watch: {
    show: {
      handler(newValue, oldValue) {
        this.drawer = newValue;
        //等标签加载成功以后在调用
        if (newValue) {
          //初始化表单筛选
          this.initAutoComplete();
        }
      },
      deep: true,
      immediate: true,
    },
  },
  mounted() {
    /**
     * 设置秘钥 必须在脚本加载之前设置
     */
    window._AMapSecurityConfig = {
      securityJsCode: process.env.VUE_APP_SECURITY_JS_CODE,
    };

    this._Utils = new Utils();

    /**
     * 加载高德api
     */

    this._Utils
      .loadJs(
        `https://webapi.amap.com/maps?v=2.0&key=${
          process.env.VUE_APP_GAODE_KEY_WEB_TERMINAL
        }&plugin=${new GaodeMap().plugin}`,
        { append: "body", defer: true }
      )
      .then(() => {
        this._GaodeMap = new GaodeMap(AMap);
        this.$nextTick(() => {
          //行政列表
          this._GaodeMap.districtList({ subdistrict: 2 }).then((res) => {
            this.cityOptions = res;
          });
          //搜索数据的兴趣点类别
          this.placeSearchType = this._GaodeMap.placeSearchType;
        });
      });
  },
  methods: {
    /**
     * 类型触发 开始检索数据
     */
    navigationTypeChange(e) {
      document.getElementById("panel").innerHTML = "";
      // this.viewer.entities.removeAll();
      // console.log(e);
      const start = [
        this.startPositionResult.poi.location.lng,
        this.startPositionResult.poi.location.lat,
      ];
      const end = [
        this.endPositionResult.poi.location.lng,
        this.endPositionResult.poi.location.lat,
      ];
      //驾车
      if (e == "1") {
        this._GaodeMap
          .Driving({
            panel: "panel",
            start,
            end,
          })
          .then((res) => {
            // console.log(res.routes);
            if (res.routes.length != 0) {
              let routeLocationArr = [];
              const steps = res.routes[0].steps;
              const len = steps.length;
              for (let i = 0; i < len; i++) {
                const item = steps[i];
                routeLocationArr.push([
                  item.start_location.lng,
                  item.start_location.lat,
                ]);
                if (len - 1 == i) {
                  routeLocationArr.push([
                    item.end_location.lng,
                    item.end_location.lat,
                  ]);
                }
              }
              this.$emit("load", {
                type: "navigation",
                style: "driving",
                data: routeLocationArr,
              });
            }
          });
      }
      //公交车
      if (e == "2") {
        this._GaodeMap
          .Transfer({
            panel: "panel",
            start,
            end,
            city: "沈阳市",
          })
          .then((res) => {
            // console.log(res);
          });
      }
      //骑行
      if (e == "3") {
        this._GaodeMap
          .Riding({
            panel: "panel",
            start,
            end,
          })
          .then((res) => {
            if (res.routes.length != 0) {
              let routeLocationArr = [];
              const rides = res.routes[0].rides;
              const len = rides.length;
              for (let i = 0; i < len; i++) {
                const item = rides[i];
                routeLocationArr.push([
                  item.start_location.lng,
                  item.start_location.lat,
                ]);
                if (len - 1 == i) {
                  routeLocationArr.push([
                    item.end_location.lng,
                    item.end_location.lat,
                  ]);
                }
              }
              this.$emit("load", {
                type: "navigation",
                style: "riding",
                data: routeLocationArr,
              });
            }
          });
      }
      //步行
      if (e == "4") {
        this._GaodeMap
          .Walking({
            panel: "panel",
            start,
            end,
          })
          .then((res) => {
            if (res.routes.length != 0) {
              let routeLocationArr = [];
              const steps = res.routes[0].steps;
              const len = steps.length;
              for (let i = 0; i < len; i++) {
                const item = steps[i];
                routeLocationArr.push([
                  item.start_location.lng,
                  item.start_location.lat,
                ]);
                if (len - 1 == i) {
                  routeLocationArr.push([
                    item.end_location.lng,
                    item.end_location.lat,
                  ]);
                }
              }
              this.$emit("load", {
                type: "navigation",
                style: "walking",
                data: routeLocationArr,
              });
            }
          });
      }
    },
    /**
     * tab切换触发
     */
    tabClick(e) {
      if (e.name == "2") {
        this.initAutoComplete();
      }
    },
    /**
     * 初始化路线规划和导航的表单筛选
     */
    initAutoComplete() {
      setTimeout(() => {
        //搜素组件
        this._GaodeMap
          .AutoComplete({
            input: "startPosition",
          })
          .then((res) => {
            res.on("select", (result) => {
              if (!result.poi.location) {
                this.$notify({
                  title: "警告",
                  message: "输入的地址有误，请重新输入",
                  type: "warning",
                });
                return;
              }
              this.startPositionResult = result;
            });
          });

        this._GaodeMap
          .AutoComplete({
            input: "endPosition",
          })
          .then((res) => {
            res.on("select", (result) => {
              if (!result.poi.location) {
                this.$notify({
                  title: "警告",
                  message: "输入的地址有误，请重新输入",
                  type: "warning",
                });
                return;
              }
              this.endPositionResult = result;
            });
          });
      }, 500);
    },
    /**
     * 点击兴趣点
     * @param {*} item
     */
    poisClick(item) {
      this.$emit("poisClick", item);
    },
    /**
     * 搜索
     */
    search() {
      if (this.keyword == "") return;
      if (this.range == "city") {
        if (this.city.length == 0) {
          this.$message({
            message: "请选择城市",
            type: "warning",
          });
          return;
        }
      }
      const placeSearchData = {
        pageSize: this.pageSize,
        pageIndex: this.pageIndex,
        searchKeyword: this.keyword,
        extensions: "all",
      };
      if (this.range == "city") {
        placeSearchData.city = this.city[1] || "";
      }
      if (this.searchType.length != 0) {
        placeSearchData.type = this.searchType.join("|");
      }
      this.loading = true;
      this._GaodeMap
        .placeSearch(placeSearchData)
        .then((res) => {
          this.total = res.count;
          this.poisList = res.pois;
          this.$nextTick(() => {
            this.$refs.poisList.scrollTop = 0;
          });
          this.loading = false;
          this.$emit("load", {
            type: "pois",
            polymerization: this.polymerization,
            data: this.poisList,
          });
        })
        .catch((err) => {
          this.$notify.error({
            title: "错误",
            message: "请重新搜索",
          });
        });
    },
    /**
     * 分页
     * @param {NUmber} i 页码
     */
    currentPaginationChange(i) {
      this.pageIndex = i;
      this.search();
    },
    /**
     * 关闭抽屉窗口
     */
    handleClose() {
      this.$emit("close");
    },
  },
};
</script>
  
<style scoped lang="scss">
@import "./POI-panel.scss";
</style>