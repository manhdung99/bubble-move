<template>
  <div class="bubble-container">
    <img
      v-for="(bubble, index) in bubbles"
      :key="bubble.id"
      class="avatar"
      :src="bubble.img"
      :style="{
        left: bubble.x + 'vw',
        width: bubble.size + 'px',
        height: bubble.size + 'px',
        animationDuration: bubble.duration + 's',
        animationDelay: bubble.delay + 's',
      }"
      @animationend="resetBubble(index)"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";

const MAX_AVATARS = 100;
const bubbles = ref([]);
let idCounter = 0;
let refillTimer = null;

const avatarUrls = [
  "https://i.pravatar.cc/100?img=1",
  "https://i.pravatar.cc/100?img=2",
  "https://i.pravatar.cc/100?img=3",
  "https://i.pravatar.cc/100?img=4",
  "https://i.pravatar.cc/100?img=5",
  "https://i.pravatar.cc/100?img=6",
  "https://i.pravatar.cc/100?img=7",
  "https://i.pravatar.cc/100?img=8",
  "https://i.pravatar.cc/100?img=9",
  "https://i.pravatar.cc/100?img=10",
  "https://i.pravatar.cc/100?img=11",
  "https://i.pravatar.cc/100?img=12",
  "https://i.pravatar.cc/100?img=13",
  "https://i.pravatar.cc/100?img=14",
  "https://i.pravatar.cc/100?img=15",
  "https://i.pravatar.cc/100?img=16",
  "https://i.pravatar.cc/100?img=17",
  "https://i.pravatar.cc/100?img=18",
  "https://i.pravatar.cc/100?img=19",
  "https://i.pravatar.cc/100?img=20",
  "https://i.pravatar.cc/100?img=21",
  "https://i.pravatar.cc/100?img=22",
  "https://i.pravatar.cc/100?img=23",
  "https://i.pravatar.cc/100?img=24",
  "https://i.pravatar.cc/100?img=25",
  "https://i.pravatar.cc/100?img=26",
  "https://i.pravatar.cc/100?img=27",
  "https://i.pravatar.cc/100?img=28",
  "https://i.pravatar.cc/100?img=29",
  "https://i.pravatar.cc/100?img=30",
];

function getDeviceType() {
  const width = window.innerWidth;
  if (width < 768) return "mobile";
  if (width < 1024) return "tablet";
  return "desktop";
}

let deviceType = getDeviceType();
window.addEventListener("resize", () => {
  deviceType = getDeviceType();
});

function getConfigByDevice() {
  if (deviceType === "mobile") {
    return { max: 50, minSize: 20, maxSize: 60 };
  } else if (deviceType === "tablet") {
    return { max: 50, minSize: 40, maxSize: 80 };
  } else {
    return { max: 100, minSize: 40, maxSize: 120 };
  }
}

function createAvatar() {
  const { minSize, maxSize } = getConfigByDevice();
  return {
    id: idCounter++,
    x: Math.random() * 100,
    size: Math.round(minSize + Math.random() * (maxSize - minSize)),
    duration: 5 +  Math.random()*10, 
    delay: Math.random() * 5,
    img: avatarUrls[Math.floor(Math.random() * avatarUrls.length)],
  };
}

function resetBubble(index) {
  bubbles.value[index] = createAvatar();
}

onMounted(() => {
  for (let i = 0; i < MAX_AVATARS; i++) {
    bubbles.value.push(createAvatar());
  }

  refillTimer = setInterval(() => {
    const i = Math.floor(Math.random() * bubbles.value.length);
    bubbles.value[i] = createAvatar();
  }, 500);
});

onBeforeUnmount(() => {
  if (refillTimer) clearInterval(refillTimer);
});
</script>

<style scoped>
.bubble-container {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: linear-gradient(to top, #001f3f, #0074d9);
}

.avatar {
  position: absolute;
  bottom: -160px;
  border-radius: 50%;
  object-fit: cover;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.15);
  animation-name: rise;
  animation-timing-function: linear;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  will-change: transform, opacity;
}

@keyframes rise {
  0% {
    transform: translateY(0) scale(1);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 1;
  }
  100% {
    transform: translateY(-120vh) scale(1.05);
    opacity: 0;
  }
}
</style>
