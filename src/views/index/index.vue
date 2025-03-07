<template>
  <div class="container">
    <viewer :images="images" :options="options" @inited="inited" class="viewer" ref="viewer">
      <template #default="scope">
        <img class="hide" v-for="(item, index) in scope.images" :src="item" :key="index" />
        <!-- {{ scope.options }} -->
      </template>
    </viewer>

    <el-card class="box-card" style="margin-bottom: 16px">
      <div slot="header" class="clearfix">
        <span>参考文档</span>
      </div>
      <div>
        <Button-linear-color v-for="(item, index) in computed_private(doc)" :key="index" :title="item.title"
          :index="index" identifier="doc" @trigger="link" />
      </div>
    </el-card>
    <el-card class="box-card" style="margin-bottom: 16px">
      <div slot="header" class="clearfix">
        <span>参考效果</span>
      </div>
      <div>
        <Button-linear-color v-for="(item, index) in computed_private(effect)" :key="index" :title="item.title"
          :index="index" identifier="effect" @trigger="link" />
      </div>
    </el-card>
    <el-card class="box-card" style="margin-bottom: 16px">
      <div slot="header" class="clearfix">
        <span>三方依赖（根据需求酌情使用）</span>
      </div>
      <div class="card-main">
        <div v-for="(item, index) in computed_private(tripartite)" :key="index">
          <el-tooltip class="item" effect="dark" :content="item.describe" placement="top-start">
            <Button-linear-color :title="item.title" :index="index" identifier="tripartite" @trigger="link" />
          </el-tooltip>
        </div>
      </div>
    </el-card>

    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>工具</span>
      </div>
      <div class="card-main">
        <div v-for="(item, index) in computed_private(tools)" :key="index">
          <el-tooltip class="item" effect="dark" :content="item.describe" placement="top-start">
            <Button-linear-color :title="item.title" :index="index" identifier="tools" @trigger="link" />
          </el-tooltip>
        </div>
      </div>
    </el-card>
  </div>
</template>
 
<script>
import { doc, effect, tripartite, tools } from "./module/data";
import "viewerjs/dist/viewer.css";
import { component as Viewer } from "v-viewer";
import ButtonLinearColor from "@/components/Button-linear-color";
export default {
  components: { Viewer, ButtonLinearColor },
  name: "Files",
  data() {
    return {
      doc: doc,
      effect: effect,
      tripartite: tripartite,
      tools: tools,
      images: [],
      options: {
        inline: false,
        button: true,
        navbar: false,
        title: true,
        toolbar: false,
        tooltip: true,
        movable: true,
        zoomable: true,
        rotatable: true,
        scalable: true,
        transition: true,
        fullscreen: true,
        keyboard: true,
      },
    };
  },
  computed: {
    //计算是否是私有显示
    computed_private() {
      return (arr) => {
        const show = process.env.NODE_ENV === "development";
        if (!show) {
          return arr.filter((item) => !(item?.private && item.private));
        } else {
          return arr;
        }
      };
    },
  },
  methods: {
    inited(viewer) {
      this.$viewer = viewer;
    },
    show() {
      this.$viewer.show();
    },
    /**
     * 跳转
     */
    link(data) {
      let item;
      if (data.identifier == "doc") {
        item = this.doc[data.data];
      }
      if (data.identifier == "effect") {
        item = this.effect[data.data];
      }
      if (data.identifier == "tripartite") {
        item = this.tripartite[data.data];
      }
      if (data.identifier == "tools") {
        item = this.tools[data.data];
      }
      if (item?.externalLinks && item.externalLinks) {
        window.open(item.link);
      } else if (item?.innerChain && item.innerChain) {
        this.$router.push({ path: item.link });
      } else {
        this.images = [item.link];
        this.$viewer.show();
      }
    },
  },
};
</script>
  
<style scoped lang="scss">
.container {
  width: 100%;
  height: 100%;
  padding: 16px;
  overflow-y: auto;

  .pan-btn {
    margin: 5px;
  }

  .card-main {
    display: flex;
    flex-wrap: wrap;
  }
}

.hide {
  display: none;
}
</style>