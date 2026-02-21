<template>
  <div
    class="virtual-scroller"
    ref="scrollContainer"
    :style="{ height: containerHeight + 'px' }"
    @scroll.passive="onScroll"
  >
    <div class="virtual-scroller__spacer" :style="{ height: totalHeight + 'px' }">
      <div
        class="virtual-scroller__content"
        :style="{ transform: `translateY(${offsetY}px)` }"
      >
        <div
          v-for="item in visibleItems"
          :key="item.id"
          class="virtual-scroller__item"
          :style="{ height: itemHeight + 'px' }"
        >
          <slot :item="item" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
  itemHeight: {
    type: Number,
    required: true,
  },
  containerHeight: {
    type: Number,
    default: 500,
  },
  overscan: {
    type: Number,
    default: 5,
  },
})

const scrollContainer = ref(null)
const scrollTop = ref(0)

const totalHeight = computed(() => props.items.length * props.itemHeight)

const startIndex = computed(() =>
  Math.floor(scrollTop.value / props.itemHeight)
)
const endIndex = computed(() =>
  Math.ceil((scrollTop.value + props.containerHeight) / props.itemHeight)
)
const renderStart = computed(() =>
  Math.max(0, startIndex.value - props.overscan)
)
const renderEnd = computed(() =>
  Math.min(props.items.length, endIndex.value + props.overscan)
)

const visibleItems = computed(() =>
  props.items.slice(renderStart.value, renderEnd.value)
)

const offsetY = computed(() => renderStart.value * props.itemHeight)

function onScroll() {
  scrollTop.value = scrollContainer.value.scrollTop
}
</script>

<style scoped>
.virtual-scroller {
  overflow: auto;
  overscroll-behavior: contain;
  position: relative;
}

.virtual-scroller__spacer {
  position: relative;
}

.virtual-scroller__content {
  position: absolute;
  left: 0;
  right: 0;
}

.virtual-scroller__item {
  display: flex;
  align-items: center;
  padding: 0 20px;
  border-bottom: 1px solid #f0f0f0;
}
</style>
