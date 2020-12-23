<template>
  <div>
    <el-switch v-model="draggable" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedKey"
      node-key="catId"
      show-checkbox
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level<3" type="text" size="mini" @click="() => append(data)">新增</el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">编辑</el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >删除</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" width="30%">
      <span>
        <el-form :model="category">
          <el-form-item label="分类名称">
            <el-input v-model="category.name" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="图标">
            <el-input v-model="category.icon" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="计量单位">
            <el-input v-model="category.productUnit" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
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
      },
      expandedKey: [],
      dialogVisible: false,
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productUnit: "",
        icon: "",
        catId: null
      },
      dialogTitle: "",
      dialogType: "",
      maxLevel: 0,
      updateNodes: [],
      draggable: false,
      pCids: []
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
      });
    },
    append(data) {
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
      this.dialogType = "append";
      this.dialogTitle = "新增分类";
      this.dialogVisible = true;
      console.log("append", data);
    },
    edit(data) {
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
        params: this.$http.adornParams({})
      }).then(({ data }) => {
        console.log("请求到的数据", data);
        this.category.catId = data.data.catId;
        this.category.name = data.data.name;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.dialogType = "edit";
        this.dialogTitle = "编辑分类";
        this.dialogVisible = true;
      });
    },
    remove(node, data) {
      this.$confirm("是否删除?", "删除提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          var ids = [data.catId];
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.getMenus();
            this.expandedKey = [node.parent.data.catId];
            this.$message({
              type: "success",
              message: "删除成功!"
            });
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });

      console.log("remove", node, data);
    },
    editCategory() {
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "修改成功!"
        });
        this.dialogVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },
    appendCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "添加成功!"
        });
        this.dialogVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });

      console.log("添加的分类", this.category);
    },
    submitData() {
      if (this.dialogType == "edit") {
        this.editCategory();
      } else {
        this.appendCategory();
      }
    },
    getMaxLevel(node) {
      if (node.childNodes != null && node.childNodes.length != 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          this.getMaxLevel(node.childNodes[i]);
        }
      } else {
        if (node.level > this.maxLevel) {
          this.maxLevel = node.level;
        }
      }
    },
    allowDrop(draggingNode, dropNode, type) {
      console.log(
        "draggingNode",
        draggingNode,
        "dropNode",
        dropNode,
        "type",
        type
      );
      this.getMaxLevel(draggingNode);
      var draggingNode_depth = this.maxLevel - draggingNode.level + 1;
      var sum_depth = draggingNode_depth;
      if (type == "inner") {
        sum_depth += dropNode.level;
      } else {
        // 'prev'或者 'next'
        sum_depth += dropNode.level - 1;
      }
      this.maxLevel = 0;
      console.log(
        "sum_depth",
        sum_depth,
        "draggingNode_depth",
        draggingNode_depth
      );
      return sum_depth <= 3;
    },
    /**
     * 共四个参数，依次为：
     * 被拖拽节点对应的 Node、
     * 结束拖拽时最后进入的节点、
     * 被拖拽节点的放置位置（before、after、inner）、
     * event
     */
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log(draggingNode, dropNode, dropType, ev);
      // 父id
      let pCid = 0;
      let siblings = null;
      if (dropType == "inner") {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      } else {
        // before、after
        pCid = dropNode.data.parentCid;
        siblings = dropNode.parent.childNodes;
      }
      // 排序
      for (let i = 0; i < siblings.length; i++) {
        let catId = siblings[i].data.catId;
        if (catId == draggingNode.data.catId) {
          let catLevel = siblings[i].level;
          if (catLevel != draggingNode.level) {
            this.updateChildLevels(siblings[i]);
          }
          this.updateNodes.push({
            catId: catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel
          });
        } else {
          this.updateNodes.push({ catId: catId, sort: i });
        }
      }

      this.pCids.push(pCid);
      console.log("updateNodes", this.updateNodes);
    },
    updateChildLevels(node) {
      for (let i = 0; i < node.childNodes.length; i++) {
        this.updateNodes.push({
          catId: node.childNodes[i].data.catId,
          catLevel: node.childNodes[i].data.catLevel
        });
        if (node.childNodes[i].childNodes.length != 0) {
          this.updateChildLevels(node.childNodes[i]);
        }
      }
    },
    batchSave() {
      // 更新数据库数据
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.getMenus();
        this.expandedKey = this.pCids;
        this.updateNodes = [];
        this.pCids = [];
      });
    },
    batchDelete() {
      // 收集id
      let deleteIds = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      for (let i = 0; i < checkedNodes.length; i++) {
        deleteIds.push(checkedNodes[i].catId);
      }
      this.$confirm("是否删除?", "删除提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(deleteIds, false)
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "删除成功!"
            });
            this.getMenus();
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "取消删除!"
          });
        });

      console.log("checkedNodes", checkedNodes);
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