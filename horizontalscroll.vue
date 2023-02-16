<template>
  <div class="text-container w-[calc(100%-32px)] bg-transparent rounded-6 text-sm scroll-smooth" ref="container">
    <div
      class="flex flex-nowrap bg-white px-1.5 h-14 scroll-smooth"
      ref="textBlocks"
      @touchstart="handleTouchStart"
      @touchmove="handleTouchMove"
    >
      <div
        class="flex-[0_0_auto] py-1.5 px-3 mr-4 self-center"
        :class="activeTab === text ? 'text-secondary bg-purpleLightBg rounded-4 ease-linear font-medium' : ''"
        v-for="(text, index) in props.items"
        :key="index"
        @click="centerTextBlock(index, text)"
      >
        {{ text }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
export interface Props {
  items: string[];
}
const props = defineProps<Props>();
const emits = defineEmits(['chosenTab']);

const activeTab = ref(props.items[0]);

let startX = 0;
let scrollLeft = 0;

const textBlocks = ref(null);
const container = ref(null);

const handleTouchStart = (event) => {
  startX = event.touches[0].clientX;
  scrollLeft = textBlocks.value.scrollLeft;
};

const handleTouchMove = (event) => {
  const x = event.touches[0].clientX;
  const walk = (x - startX) * 3;
  textBlocks.value.scrollLeft = scrollLeft - walk;
};

const centerTextBlock = (index: number, text: string) => {
  const containerWidth = textBlocks.value.clientWidth;
  const block = textBlocks.value.children[index];
  const blockWidth = block.offsetWidth;
  const blockLeft = block.offsetLeft;
  const centerOffset = containerWidth / 2 - blockWidth / 2;
  const targetScrollLeft = blockLeft - centerOffset;
  slowScroll(targetScrollLeft);
  activeTab.value = text;
  emits('chosenTab', activeTab.value);
};

const slowScroll = (targetScrollLeft: any) => {
  const start = container.value.scrollLeft;
  const change = targetScrollLeft - start;
  const increment = 20;
  let currentTime = 0;

  const animateScroll = () => {
    currentTime += increment;
    const easing = easeInOutQuad(currentTime, start, change, 1000);
    container.value.scrollLeft = easing;
    if (currentTime < 1000) {
      window.requestAnimationFrame(animateScroll);
    }
  };

  animateScroll();
};

const easeInOutQuad = (increment: number, start: any, change: number, duration: number) => {
  increment /= duration / 2;
  if (increment < 1) return (change / 2) * increment * increment + start;
  increment--;
  return (-change / 2) * (increment * (increment - 2) - 1) + start;
};
</script>
<style scoped>
.text-container {
  overflow-x: scroll;
  -webkit-overflow-scrolling: touch;
  box-shadow: 0px 4px 35px rgba(18, 11, 29, 0.06);
}

.text-container::-webkit-scrollbar {
  display: none;
}
</style>
