<template>
  <div
    class="bubble"
    :style="bubbleStyle"
    :title="bubble.title || ''"
  >
    <img v-if="bubble.img" :src="bubble.img" alt="avatar" />
    <div v-else class="placeholder"></div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
const props = defineProps({
  bubble: { type: Object, required: true }
})

const bubbleStyle = computed(() => ({
  width: props.bubble.size + 'px',
  height: props.bubble.size + 'px',
  left: (props.bubble.leftPercent || 0) + '%',
  animationDuration: '15s',
  animationDelay: (props.bubble.delay || 0) + 's',
  zIndex: props.bubble.zIndex ?? 1,
}))
</script>

<style scoped>
</style>
