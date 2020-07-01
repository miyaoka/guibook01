<template>
  <div>
    <h3>ドラッグ移動で入れ替わり</h3>

    <ul class="items" :class="{ isDragging }">
      <li
        v-for="(item, i) in list"
        :key="i"
        class="item"
        :style="draggingStyles[i]"
        :class="{ isDraggingItem: fromIndex === i }"
        @mousedown="(e) => onDragStart(e, i)"
      >
        item {{ item }} ({{ i }})
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { mv } from '@/utils/list'

type DragItemStyle = Record<string, string> | null

export default Vue.extend({
  data() {
    return {
      list: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split(''),
      fromIndex: -1,
      toIndex: -1,
      dragStartY: 0,
      deltaY: 0,
      selectedIndex: -1,
      draggingStyle: null as DragItemStyle,
      draggingRect: {} as { width: number; height: number },
      isDragging: false,
    }
  },
  computed: {
    draggingStyles(): DragItemStyle[] {
      if (this.fromIndex < 0) return []

      const getTranslate = (index: number) => {
        // ドラッグ要素
        if (this.fromIndex === index) return this.deltaY
        // ドラッグ要素より前の要素はindex加算
        const adjustedIndex = index < this.fromIndex ? index + 1 : index
        // ドラッグ先より後ろの要素はずらす
        if (adjustedIndex > this.toIndex) return this.draggingRect.height
        // それ以外はstyle設定しない
        return null
      }

      const styles = this.list.map(
        (_item, index): DragItemStyle => {
          const diff = getTranslate(index)
          return diff ? { transform: `translateY(${diff}px)` } : null
        }
      )
      styles[this.fromIndex] = {
        ...this.draggingStyle,
        ...styles[this.fromIndex],
      }
      return styles
    },
  },
  methods: {
    getTranslate(index: number) {
      if (this.fromIndex === index)
        return Math.min(
          this.draggingRect.height * (this.list.length - this.fromIndex - 1),
          Math.max(-this.draggingRect.height * this.fromIndex, this.deltaY)
        )

      const i = index < this.fromIndex ? index + 1 : index
      if (i > this.toIndex) return this.draggingRect.height
      return null
    },

    getStyle(index: number) {
      if (index === this.fromIndex) {
        return { transform: `translateY(${this.deltaY}px)` }
      }
      return null
    },
    onDragStart(e: MouseEvent, index: number) {
      this.fromIndex = this.toIndex = index
      this.dragStartY = e.pageY
      const target = e.target as HTMLElement

      const { offsetWidth: width, offsetHeight: height } = target

      this.draggingRect = {
        width,
        height,
      }
      this.draggingStyle = {
        width: `${width}px`,
        height: `${height}px`,
      }
      document.addEventListener('mousemove', this.onDrag)
      document.addEventListener('mouseup', this.onDragEnd)
    },
    onDrag(e: MouseEvent) {
      this.isDragging = true
      this.deltaY = e.pageY - this.dragStartY
      this.toIndex =
        this.fromIndex +
        Math.floor(this.deltaY / this.draggingRect.height + 0.5)
    },
    onDragEnd() {
      this.isDragging = false
      this.list = mv(this.list, this.fromIndex, this.toIndex)
      this.fromIndex = this.toIndex = -1
      this.deltaY = 0
      document.removeEventListener('mousemove', this.onDrag)
      document.removeEventListener('mouseup', this.onDragEnd)
    },
    select(index: number) {
      if (this.selectedIndex < 0) {
        this.selectedIndex = index
        return
      }
      this.list = mv(this.list, this.selectedIndex, index)
      this.selectedIndex = -1
    },
  },
})
</script>

<style lang="scss">
.container {
  display: flex;
  flex-direction: column;
}
.items {
  list-style: none;
  margin: 0;
  padding: 0;
  width: 160px;
  &.isDragging {
    .item {
      transition: ease-out 0.2s;
    }
  }
  .item {
    display: flex;
    flex: 1;
    align-items: center;
    justify-content: center;
    border: 1px solid #666;
    padding: 10px;
    margin: 0;
    height: 100px;
    box-sizing: border-box;
    cursor: pointer;
    user-select: none;
    background: #fff;
    z-index: 1000;

    &.isDraggingItem {
      transition: none;
      position: absolute;
      background: #ffc;
      z-index: 1;
      box-shadow: 4px 4px 4px rgba(#666, 0.2);
    }
  }
}
</style>
