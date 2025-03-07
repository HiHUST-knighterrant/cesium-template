<template>
  <el-drawer
    append-to-body
    destroy-on-close
    :visible.sync="viewCode"
    direction="rtl"
    :before-close="handleClose"
    size="700px"
    custom-class="highlight"
  >
    <template #title>
      <div class="title-view">
        <div class="title">源代码</div>
        <el-radio-group
          size="mini"
          v-model="codeLanguage"
          @change="codeLanguageChange"
        >
          <el-radio-button
            v-for="(item, index) in codeComputed"
            :key="index"
            :label="item.codeLanguage"
            >{{ item.codeLanguageLable }}</el-radio-button
          >
        </el-radio-group>
      </div>
    </template>
    <div class="code-view">
      <div class="subCodeLanguage">
        <el-radio-group size="mini" v-model="subCodeLanguage">
          <el-radio-button
            v-for="(item, index) in subCodeComputed.code"
            :key="index"
            :label="item.codeLanguage"
            >{{ item.codeLanguage }}</el-radio-button
          >
        </el-radio-group>
        <div>
          <el-button
            icon="el-icon-copy-document"
            class="copy"
            size="mini"
            type="primary"
            @click="doCopy"
            plain
            >一键复制</el-button
          >
          <el-dropdown
            class="dropdown"
            @command="handleCommand"
            v-if="subCodeComputed.relyOn && subCodeComputed.relyOn.length != 0"
          >
            <el-button size="mini" type="primary">
              依赖文件<i class="el-icon-connection el-icon--right"></i>
            </el-button>
            <el-dropdown-menu class="dropdown-menu" slot="dropdown">
              <el-dropdown-item disabled
                >当前示例依赖文件（注意顺序）</el-dropdown-item
              >
              <!-- <el-dropdown-item
                v-if="subCodeComputed.relyOn.length > 6"
                command="all"
                >下载全部</el-dropdown-item
              > -->
              <el-dropdown-item
                :command="item"
                v-for="(item, index) in subCodeComputed.relyOn"
                :key="index"
                >{{ item.label }}</el-dropdown-item
              >
            </el-dropdown-menu>
          </el-dropdown>
        </div>
      </div>
      <div class="code-main">
        <div v-for="(_item, _index) in codeComputed" :key="_index">
          <div v-if="_item.codeLanguage == codeLanguage">
            <div v-for="(item, index) in subCodeComputed.code" :key="index">
              <div
                class="code-main-view"
                v-if="item.codeLanguage == subCodeLanguage"
                v-code
              >
                <pre>
                  <code :class="item.codeLanguage | languageFilter">
                      {{item.content}}
                  </code>
                </pre>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </el-drawer>
</template>
 
<script>
import { mapState } from "vuex";
import { downloadLocalFile } from "@/utils";
export default {
  props: {
    option: {
      type: Object,
      default: () => {
        return {};
      },
    },
  },
  filters: {
    languageFilter(value) {
      if (value == "js") {
        return "javascript";
      }
      if (value == "css") {
        return "scss";
      }
      if (value == "html") {
        return "html";
      }
      return value;
    },
  },
  computed: {
    ...mapState({
      code: (state) => state.highlight.code,
      viewCode: (state) => state.highlight.viewCode,
    }),
    //一级计算
    codeComputed() {
      function get_subCodeLanguage(type) {
        if (type == "VUE") {
          return "Vue";
        }
        if (type == "JS") {
          return "原生JS";
        }
      }
      if (this.code.length == 0) {
        return [];
      } else {
        return this.code.map((item) => {
          return {
            ...item,
            codeLanguageLable: get_subCodeLanguage(item.codeLanguage),
          };
        });
      }
    },
    //二级计算
    subCodeComputed() {
      const _filter = this.code.filter(
        (item) => this.codeLanguage == item.codeLanguage
      );
      return _filter.length != 0 ? _filter[0] : [];
    },
    /**
     * 拷贝的数据
     */
    copyData() {
      return 123;
    },
  },
  data() {
    return {
      codeLanguage: "VUE",
      subCodeLanguage: "html",
    };
  },
  created() {},
  mounted() {
    if (this.code && this.code.length != 0) {
      this.codeLanguage = this.code[0].codeLanguage;
      this.subCodeLanguage = this.code[0].code[0].codeLanguage;
    }
  },
  methods: {
    // 复制
    doCopy() {
      const copyDataFilter = this.subCodeComputed.code.filter(
        (item) => item.codeLanguage == this.subCodeLanguage
      );
      if (copyDataFilter.length == 0) return;
      const copyData = copyDataFilter[0].content;
      this.$copyText(copyData)
        .then((message) => {
          this.$notify({
            title: "成功",
            message: "复制成功",
            type: "success",
          });
        })
        .catch((err) => {
          this.$notify.error({
            title: "错误",
            message: "复制失败",
          });
        });
    },
    /**
     * 窗口关闭
     */
    handleClose() {
      this.$store.dispatch("highlight/set_view_code", false);
    },
    /**
     * 点击菜单
     */
    handleCommand(e) {
      //废弃 测试最多一次性下载10个 后期修改成zip压缩包
      if (e == "all") {
        const f_externalLinks = this.subCodeComputed.relyOn.filter(
          (item) => !item?.externalLinks || !item.externalLinks
        );
        if (f_externalLinks.length != this.subCodeComputed.relyOn.length) {
          this.$notify({
            title: "警告",
            message: "存在 npm包 需要单独点击下载",
            type: "warning",
          });
        }
        for (let i = 0; i < f_externalLinks.length; i++) {
          downloadLocalFile(f_externalLinks[i].url);
        }
        return;
      }
      
      if (e?.externalLinks && e.externalLinks) {
        window.open(e.url);
      } else {
        downloadLocalFile(e.url);
      }
    },
    /**
     * 一级语言切换
     */
    codeLanguageChange(e) {
      this.subCodeLanguage = this.subCodeComputed.code[0].codeLanguage;
    },
  },
};
</script>
  
<style scoped lang="scss">
.dropdown-menu {
  max-height: 300px;
  overflow-y: auto;
}
.highlight {
  .title-view {
    display: flex;
    align-items: center;
    .title {
      margin-right: 20px;
      font-weight: bold;
      font-size: 16px;
    }
  }
  .dropdown {
    margin-left: 15px;
  }
  .code-view {
    border-top: 1px solid #dcdfe6;
    height: calc(100vh - 80px);
    display: flex;
    flex-direction: column;
    .subCodeLanguage {
      padding: 10px 20px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      box-sizing: border-box;
      height: 68px;
    }
    .code-main {
      width: 100%;
      background: #282c34;
      box-sizing: border-box;

      height: calc(100% - 68px);
      position: relative;
      .code-main-view {
        height: 100%;
        width: 100%;
        position: absolute;
        top: 0;
        left: 0;
        padding: 10px;
        pre {
          width: 100%;
          height: 100%;
          margin: 0;
          padding: 0;
          display: flex;
          // white-space: pre-wrap;
          // tab-size: 2;
          code {
            width: 100%;
            margin: 0;
            height: 100%;
            box-sizing: border-box;
            //  white-space: pre;
            padding: 0;
          }
        }
      }
    }
  }
}
</style>