<template>
  <v-card elevation="16" max-width="600" class="mx-auto">
    <v-btn @click="toggle">toggle</v-btn>
    <v-treeview
      v-model="values"
      ref="treeview"
      :items="items"
      activatable
      dense
      hoverable
      item-key="id"
      item-text="name"
      item-children="children"
      open-all
      selectable
      selection-type="independent"
      expand-icon="mdi-chevron-down"
      @input="input"
      @update:active="updateActive"
    >
      <template #prepend>--</template>
      <template #append>
        <v-btn icon>
          <v-icon>mdi-asterisk</v-icon>
        </v-btn>
      </template>
      <template #label="{item}">{{ item.name }}</template>
    </v-treeview>
  </v-card>
</template>

<script>
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
  data() {
    return {
      openAll: false,
      values: [],
      items: Array.from({ length: 10 }, (v, index) => generateLevel(index))
    };
  },
  methods: {
    input(v) {
      console.log('--------------input-----------')
      console.log(v, this.values);
    },
    updateActive([v]) {
      const index=this.values.indexOf(v)
      if(index!==-1){
        this.values.splice(index,1)
      }
      else{
        this.values.push(v)
      }
      console.log('--------------update:active-----------')
      console.log(v, this.values);
    },
    toggle() {
      this.openAll = !this.openAll;
      this.$refs.treeview.updateAll(this.openAll);
    }
  },
  mounted() {}
};
</script>

<style lang="css" scoped></style>
