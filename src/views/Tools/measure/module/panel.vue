<template>
    <el-card class="panel-card">
        <div slot="header" class="clearfix">
            <span>操作面板</span>
            <el-button style="float: right; padding: 3px 0" type="text" @click="clears">清除</el-button>
        </div>
        <div class="panel-content">
            <el-button type="primary" v-for="(item, index) in clickCreateFor" :key="index" size="mini"
                @click="clickCreate(item.value)">{{ item.label }}</el-button>
        </div>
    </el-card>
</template>
 
<script>
export default {
    props: {
        option: {
            type: Object,
            default: () => {
                return {};
            },
        },
    },
    data() {
        return {
            clickCreateFor: [
                {
                    label: "距离测量",
                    value: "distance"
                },
                // {
                //     label: "测量高度",
                //     value: "height"
                // },
                {
                    label: "面积测量",
                    value: "area"
                }
            ]
        };
    },
    created() { },
    mounted() {

    },
    methods: {
        clickCreate(name) {
            this.$emit('measure', name)
        },
        clears() {
            this.$emit('clears')
        }
    },
};
</script>
  
<style scoped lang="scss">
.panel-card {
    position: fixed;
    bottom: 10px;
    right: 10px;
    width: 300px;

    .panel-content {
        >* {
            margin: 0 10px 10px 0;
        }
    }
}
</style>