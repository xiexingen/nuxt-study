<template>
  <v-virtual-scroll
    :height="virtual.height"
    :bench="virtual.bench"
    :item-height="virtual.itemHeight"
    :items="virtualScrollItems"
  >
    <template #default="{item,index}">
      <v-list-item
        :key="item[itemKey]"
        :style="{
          paddingLeft: indent * (item.level - 1) + 40 + 'px'
        }"
      >
        <v-btn
          icon
          v-if="item[itemChildren] && item[itemChildren].length"
          @click="toggleExpand($event, item, index)"
        >
          <v-icon>{{
            item.expand ? "mdi-chevron-down" : "mdi-chevron-right"
          }}</v-icon>
        </v-btn>
        <span
          v-if="!item[itemChildren] || !item[itemChildren].length"
          :style="{
            width: `${indent*2}px`
          }"
        >
        </span>
        <slot :item="item" :index="index">
          {{ item.name }}
        </slot>
      </v-list-item>
    </template>
  </v-virtual-scroll>
</template>

<script>
export default {
  name: "TreeSelect",
  props: {
    // 树的数据源（树形结构）
    items: {
      type: Array,
      required: true
    },
    // 将鼠标悬停在节点上时应用悬停类
    hoverable: {
      type: Boolean,
      required: false,
      default: true
    },
    // 查询字符串
    search: {
      type: String,
      required: false
    },
    // 过滤函数
    filter: {
      type: Function,
      required: false,
      default: (search, itemText) => true
    },
    // 控制树形视图如何选择节点。 有两种模式可用：‘leaf’(关联叶子节点) ‘independent’(互不影响)、reject(层级互斥)
    selectionType: {
      type: String,
      required: false,
      default: "leaf"
    },
    // value:{},
    // children字段属性名
    itemChildren: {
      type: String,
      required: false,
      default: "children"
    },
    // key字段属性名
    itemKey: {
      type: String,
      required: false,
      default: "id"
    },
    // 值字段属性名
    itemValue: {
      type: String,
      required: false,
      default: "value"
    },
    // 文本字段属性名
    itemText: {
      type: String,
      required: false,
      default: "label"
    },
    // 禁用字段属性名
    itemDisabled: {
      type: String,
      required: false,
      default: "disabled"
    },
    // 默认展开所有
    defaultExpandAll: {
      type: Boolean,
      required: false,
      default: false
    },
    // 点击选中
    selectOnClick: {
      type: Boolean,
      required: false,
      default: true
    },
    // 返回对象
    returnObject: {
      type: Boolean,
      required: false,
      default: false
    },
    //是否多选
    multiple: {
      type: Boolean,
      required: false,
      default: false
    },
    // 是否纸片形式显示(可删除)
    chip: {
      type: Boolean,
      required: false,
      default: false
    },
    // 虚拟滚动,配置项参考 v-virtual-scroll组件
    virtual: {
      type: Object,
      required: false,
      default: () => ({
        bench: 0,
        height: 300,
        itemHeight: 32,
        width: 300
      })
    },
    // 父子节点之间的缩进
    indent: {
      type: Number,
      required: false,
      default: 18
    }
  },
  data() {
    return {};
  },
  computed: {
    // 将tree形数据转为平面化数据,并处理显示/折叠
    flattenTree() {
      const flatten = (
        tree,
        list = [],
        level = 0,
        parent = { [this.itemChildren]: null }
      ) => {
        const itemChildKey = this.itemChildren;
        if (tree && tree instanceof Array) {
          ++level;
          tree.forEach(item => {
            if (item.expand === undefined) {
              item.expand = this.defaultExpandAll;
            }
            if (item.visible === undefined) {
              if (this.defaultExpandAll || level === 1) {
                item.visible = true;
              } else {
                item.visible = false;
              }
            }
            item.level = level;
            item.parent = parent;
            list.push(item);
            if (item[itemChildKey]) {
              flatten(item[itemChildKey], list, level, item);
            }
          });
        }
        return list;
      };
      return flatten(this.items);
    },
    virtualScrollItems(){
      return this.flattenTree.filter(m=>m.visible)
    }
  },
  methods: {
    // 点击节点
    nodeClick(item) {
      debugger;
      const recursionSelect = function(children, value) {
        children &&
          children.forEach(node => {
            node.select = value;
            recursionSelect(node.children, value);
          });
      };
      recursionSelect(this.flattenTree, false);
      recursionSelect([item], true);
      this.updateView();
      this.$emit("node-click", item);
    },
    // 展开全部
    toggleExpandAll(state, level = 1) {
      debugger;
      const that = this;
      let expandNodes = []; // 待展开/折叠的节点
      let rootNodes = []; // 父级节点（直到根节点）
      // 1. 找到对应节点
      this.flattenTree.forEach(item => {
        if (item.level === level) {
          expandNodes.push(item);
        }
        if (item.level < level) {
          rootNodes.push(item);
        }
      });
      // 2. 展开/折叠节点
      expandNodes.forEach(item => {
        if (state) {
          that.expand(item, false);
        } else {
          that.collapse(item, false);
        }
      });
      // 3.展开父级节点
      rootNodes.forEach(item => {
        that.expand(item, true);
      });
      this.updateView();
    },
    // 切换节点展开/折叠
    toggleExpand(e, item) {
      e && e.stopPropagation();
      const isExpand = item.expand;
      if (isExpand) {
        this.collapse(item, true); // 折叠
      } else {
        this.expand(item, true); // 展开
      }
    },
    // 展开节点
    expand(item, isKeep = true) {
      debugger;
      const itemChildKey = this.itemChildren;
      const recursionVisible = function(children) {
        children.forEach(node => {
          if (!isKeep) {
            node.expand = true;
          }
          node.visible = true;
          if (node.expand && node.children) {
            recursionVisible(node.children);
          }
        });
      };
      item.expand = true;
      item[itemChildKey] &&
        item[itemChildKey].forEach(node => {
          if (!isKeep) {
            node.expand = true;
          }
          node.visible = true;
          node.expand &&
            node[itemChildKey] &&
            recursionVisible(node[itemChildKey]);
        });
    },
    // 折叠节点
    collapse(item, isKeep = true) {
      debugger
      const itemChildKey = this.itemChildren;
      const recursionVisible = function(children) {
        children.forEach(node => {
          if (!isKeep) {
            node.expand = false;
          }
          node.visible = false;
          if (node[itemChildKey]) {
            recursionVisible(node[itemChildKey]);
          }
        });
      };
      item.expand = false;
      item[itemChildKey] && recursionVisible(item[itemChildKey]);
    }
  }
};
</script>

<style></style>
