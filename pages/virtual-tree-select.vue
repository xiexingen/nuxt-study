<template>
  <div class="virtual-tree-demo">
    <v-btn color="primary" @click="toggleExpandAll">(展开/折叠)全部</v-btn>
    <!-- <VirtualTreeSelect
      ref="vtree"
      class="vtree"
      v-model="tree"
      :option="virtualOption"
      :defaultExpand="expandAll"
      @node-click="handleNodeClick"
    >
      <template v-slot="{ item }">
        {{ item.name }}
        <i class="el-icon-delete" @click.stop="handleDelete(item)" />
      </template>
    </VirtualTreeSelect> -->
    <tree-select :items="tree" :defaultExpandAll="false"></tree-select>
  </div>
</template>

<script>
import TreeSelect from '../components/TreeSelect.vue';
function generateLevel(level) {
  return {
    id:`${level}`,
    name: `Application${level}`,
    children: Array.from({ length: 20 }, (v, index) => {
      return {
        id: `${level}-${index}`,
        name: `Application-${level}-${index}`,
        children: Array.from(
          { length: Math.ceil(Math.random() * 20) },
          (_, sIndex) => ({
            id: `${level}-${index}-${sIndex}`,
            name: `Application-${level}-${index}-${sIndex}`
          })
        )
      };
    })
  };
}
export default {
  components: { TreeSelect },
  data() {
    return {
      tree: Array.from({ length: 10 }, (v, index) => generateLevel(index)),
      virtualOption: {
        visibleHeight: 500,     // 设置可视区的高度，也可以通过样式强行覆盖高度
        itemHeight: 25,         // 单个item的高度
        bench: 1          // 如果滚动有白屏，就适当添加非可视区冗余数量
      },
      expandAll: true,
    }
  },
  methods: {
    toggleExpandAll() {
      this.expandAll = !this.expandAll
      this.$refs.vtree.toggleExpandAll(this.expandAll)
    },
    handleNodeClick(node) {
      console.log(`点击了${node.label}`)
    },
    handleDelete(node) {
      this.$refs.vtree.remove(node)
    }
  }
}
</script>

<style>
.vtree {
  margin: 50px;
  height: 300px !important;
}
</style>
