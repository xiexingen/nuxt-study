<template>
  <div
    class="virtual-tree"
    :style="{ height: option.visibleHeight + 'px' }"
    @scroll="handleScroll"
  >
    <div class="vt-phantom" :style="{ height: contentHeight + 'px' }"></div>
    <div
      ref="content"
      class="vt-content"
      :style="{ transform: `translateY(${offset}px)` }"
    >
      <div
        v-for="(item, index) in visibleData"
        :key="item.id"
        :level="item.level"
        :class="{
          'vt-item': true,
          'item-selected': item.select,
          ...(itemProps(item).class || {})
        }"
        :style="{
          lineHeight: option.itemHeight + 'px',
          height: option.itemHeight + 'px',
          paddingLeft: indent * (item.level - 1) + 40 + 'px',
          ...(itemProps(item).style || {})
        }"
        v-bind="itemProps(item)"
        @click="nodeClick(item)"
      >
        <v-btn icon v-if="item.children&&item.children.length" @click="toggleExpand($event, item, index)">
          <v-icon>{{item.expand?'mdi-chevron-down':'mdi-chevron-right'}}</v-icon>
        </v-btn>
        <span v-if="!item.children||!item.children.length" class="placehold-btn">
        </span>
        <slot :item="item" :index="index"></slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  inheritAttrs: false,
  name: "VirtualTreeSelect",
  model: {
    prop: "tree",
    event: "update"
  },
  props: {
    tree: {
      // 树的数据源（树形结构）
      type: Array,
      required: true
    },
    defaultExpand: {
      // 是否默认展开
      type: Boolean,
      required: false,
      default: false
    },
    option: {
      // 配置对象
      type: Object,
      required: true
      // default: {
      //   visibleHeight: 0,     // 可视区域的高度
      //   itemHeight: 25,       // 单个item的高度
      //   // 若滚动元素不是自身，需要提供下面两个属性
      //   flexHeight: 0,        // 顶部浮动的高度
      //   scrollDom: null       // 滚动元素的节点
      // }
    },
    itemProps: {
      // item属性扩展
      type: Function,
      required: false,
      default: () => ({})
    }
  },
  data() {
    return {
      bench: 0, // 修正个数
      indent: 18, // 缩进
      offset: 0, // translateY偏移量
      visibleData: [], // 可视数据列表
      contentHeight: 0 // 容器高度
    };
  },
  computed: {
    flattenTree() {
      debugger
      const flatten = (
        tree,
        list = [],
        level = 0,
        parent = { children: tree }
      ) => {
        if (tree && tree instanceof Array) {
          ++level;
          tree.forEach(item => {
            if (item.expand === undefined) {
              item.expand = this.defaultExpand;
            }
            if (item.visible === undefined) {
              if (this.defaultExpand || level === 1) {
                item.visible = true;
              } else {
                item.visible = false;
              }
            }
            item.level = level;
            item.parent = parent;
            list.push(item);
            if (item.children) {
              flatten(item.children, list, level, item);
            }
          });
        }
        return list;
      };
      const result= flatten(this.tree);
      return result;
    }
  },
  mounted() {
    debugger
    this.updateView();
    window.addEventListener("resize", this.resize);
  },
  methods: {
    // 处理滚动
    handleScroll() {
      // 如果当前滚动元素不是当前组件，则需要传入scrollDom
      if (this.option.scrollDom) {
        const scrollTop = this.option.scrollDom.scrollTop;
        const diff = scrollTop - (this.$el.offsetTop - this.option.flexHeight);
        this.updateVisibleData(diff > 0 ? diff : 0);
      } else {
        const scrollTop =
          this.$el.scrollTop <= this.contentHeight
            ? this.$el.scrollTop
            : this.contentHeight;
        this.updateVisibleData(scrollTop);
      }
    },
    // 更新可视数据
    updateVisibleData(scrollTop = 0) {
      console.log(scrollTop);
      const that = this;
      const visibleHeight = this.option.visibleHeight
        ? this.option.visibleHeight
        : this.$el.clientHeight;
      const bench = this.option.bench ? this.option.bench : this.bench;
      const start = Math.floor(scrollTop / this.option.itemHeight);
      const end =
        start + Math.ceil(visibleHeight / this.option.itemHeight) + bench;
      const allVisibleData = (this.flattenTree || []).filter(
        item => item.visible
      );
      that.visibleData = allVisibleData.slice(start, end);
      that.offset = start * that.option.itemHeight;
      this.$nextTick(() => {
        // 避免动画渲染问题
        that.$emit("update-visible-data", {
          visibleData: that.visibleData,
          offset: that.offset
        });
      });
    },
    // 更新容器高度
    updateContentHeight() {
      this.contentHeight =
        (this.flattenTree || []).filter(item => item.visible).length *
        this.option.itemHeight;
    },
    // 窗口变化，需要做的处理
    resize(duration) {
      if (!this.timer) {
        // 避免频繁触发
        this.timer = true;
        const that = this;
        setTimeout(function() {
          that.$emit("resize");
          that.handleScroll();
          that.timer = false;
        }, duration);
      }
    },
    // 强制刷新
    updateView() {
      const that = this;
      this.updateContentHeight();
      this.$emit("update", this.tree);
      this.$nextTick(() => {
        that.handleScroll();
      });
    },
    // 点击节点
    nodeClick(item) {
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
    toggleExpand(e, item, index) {
      e && e.stopPropagation();
      const isExpand = item.expand;
      if (isExpand) {
        this.collapse(item, true); // 折叠
      } else {
        this.expand(item, true); // 展开
      }
      this.updateView();
      !isExpand && this.$emit("node-expand", e, item, index);
    },
    // 展开节点
    expand(item, isKeep = true) {
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
      item.children &&
        item.children.forEach(node => {
          if (!isKeep) {
            node.expand = true;
          }
          node.visible = true;
          node.expand && node.children && recursionVisible(node.children);
        });
    },
    // 折叠节点
    collapse(item, isKeep = true) {
      const recursionVisible = function(children) {
        children.forEach(node => {
          if (!isKeep) {
            node.expand = false;
          }
          node.visible = false;
          if (node.children) {
            recursionVisible(node.children);
          }
        });
      };
      item.expand = false;
      item.children && recursionVisible(item.children);
    },
    /** 对树节点的操作 **/
    // 添加子节点
    append(item, parent) {
      if (!parent.children) {
        this.$set(parent, "children", []);
      }
      parent.children.push(item);
      this.updateView();
    },
    // 添加兄弟节点
    insertAfter(item, node, isUpdate = true) {
      const index = node.parent.children.indexOf(node);
      node.parent.children.splice(index + 1, 0, item);
      isUpdate && this.updateView();
    },
    // 删除节点
    remove(item) {
      if (item.parent) {
        const index = item.parent.children.indexOf(item);
        item.parent.children.splice(index, 1);
        this.updateView();
      }
    }
  }
};
</script>

<style>
.virtual-tree {
  height: 100%;
  overflow: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
}
.vt-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}
.vt-content {
  left: 0;
  right: 0;
  top: 0;
  position: absolute;
  min-height: 100px;
}
.vt-item {
  padding: 5px;
  box-sizing: border-box;
  display: flex;
  position: relative;
  align-items: center;
  cursor: pointer;
}
.vt-item:hover,
.item-selected {
  background-color: #d7d7d7;
}
.virtual-tree-icon {
  position: absolute;
  left: 0;
  color: #c0c4cc;
  z-index: 10;
}
.placehold-btn{
  width:36px;
}
</style>
