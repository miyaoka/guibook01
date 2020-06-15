<template>
  <div>
    <h3>2点クリックで入れ替わり</h3>

    <ul class="items">
      <li
        v-for="(item, i) in list"
        :key="i"
        class="item"
        @click="() => select(i)"
      >
        {{ item }} ({{ i }})
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { mv } from '@/utils/list'

export default Vue.extend({
  data() {
    return {
      list: 'ABCDE'.split(''),
      selectedIndex: -1,
    }
  },
  methods: {
    select(index: number) {
      // 移動元が選択されてない->移動元選択
      if (this.selectedIndex < 0) {
        this.selectedIndex = index
        return
      }
      // 移動元を選択済み->移動実行
      this.list = mv(this.list, this.selectedIndex, index)
      this.selectedIndex = -1
    },
  },
})
</script>
<style scoped>
.items {
  list-style: none;
  margin: 0;
  padding: 0;
}
.item {
  border: 1px solid #000;
  padding: 10px;
  margin: 4px;
}
</style>
