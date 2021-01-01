<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      node-key="catId"
      ref="menuTree"
      @node-click="nodeClick"
    >
    </el-tree>
  </div>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  // import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    // 这里存放数据
    return {
      menus: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  // 计算属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 方法集合
  methods: {
     getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
        params: this.$http.adornParams({})
      }).then(({ data }) => {
        console.log("getMenus", data.data);
        this.menus = data.data;
      })
    },
    nodeClick(data, node, component) {
      console.log("分类组件被点击", data, node, component)
      this.$emit("tree-node-click", data, node, component);
    }
  },
  // 生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  // 生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, // 生命周期 - 创建之前
  beforeMount() {}, // 生命周期 - 挂载之前
  beforeUpdate() {}, // 生命周期 - 更新之前
  updated() {}, // 生命周期 - 更新之后
  beforeDestroy() {}, // 生命周期 - 销毁之前
  destroyed() {}, // 生命周期 - 销毁完成
  activated() {} // 如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>