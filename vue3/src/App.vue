<script setup>
import { computed } from 'vue'
import VirtualScroller from './components/VirtualScroller.vue'

const TOTAL_ITEMS = 100000

const items = computed(() =>
  Array.from({ length: TOTAL_ITEMS }, (_, i) => ({
    id: i,
    name: `Item #${i} — 這是第 ${i.toLocaleString()} 筆資料`,
  }))
)
</script>

<template>
  <div class="wrapper">
    <h1>Vue 3 Virtual Scroller</h1>

    <VirtualScroller
      :items="items"
      :item-height="50"
      :container-height="500"
      :overscan="5"
      class="scroller"
    >
      <template #default="{ item }">
        <div class="item-index">#{{ item.id }}</div>
        <div class="item-text">{{ item.name }}</div>
        <div class="item-badge" :class="item.id % 2 === 0 ? 'even' : 'odd'">
          {{ item.id % 2 === 0 ? '偶數' : '奇數' }}
        </div>
      </template>
    </VirtualScroller>
  </div>
</template>

<style scoped>
.wrapper {
  width: 500px;
}

h1 {
  font-size: 22px;
  margin-bottom: 16px;
  color: #1a1a1a;
}

.scroller {
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.item-index {
  width: 60px;
  font-weight: 600;
  color: #1a73e8;
  flex-shrink: 0;
}

.item-text {
  flex: 1;
  font-size: 14px;
  color: #333;
}

.item-badge {
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 500;
}

.item-badge.even {
  background: #e8f0fe;
  color: #1a73e8;
}

.item-badge.odd {
  background: #fce8e6;
  color: #d93025;
}
</style>
