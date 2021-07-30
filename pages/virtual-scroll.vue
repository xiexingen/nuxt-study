<template>
  <v-card elevation="16" max-width="400" class="mx-auto">
    <v-virtual-scroll :items="items" :height="650" :bench="1" item-height="65">
      <template #default="{item}">
        <v-list-item :key="item">
          <v-list-item-action>
            <v-btn fab small depressed color="primary">
              {{ item }}
            </v-btn>
          </v-list-item-action>

          <v-list-item-content>
            <v-list-item-title>
              User Database Record <strong>ID {{ item }}</strong>
            </v-list-item-title>
          </v-list-item-content>

          <v-list-item-action>
            <v-icon small>
              mdi-open-in-new
            </v-icon>
          </v-list-item-action>
        </v-list-item>

        <v-divider></v-divider>
      </template>
    </v-virtual-scroll>
  </v-card>
</template>

<script>
export default {
  data() {
    return {
      items: Array.from({ length: 1000 }, (k, v) => v + 1)
    };
  },
  methods: {},
  mounted() {}
};
</script>

<style lang="css" scoped></style>
