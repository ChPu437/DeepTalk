<template>
  <div class="live2d-view-container" ref="pixiContainerRef">
    <!-- PIXI Canvas 将被动态添加到这里 -->
    <Live2DModel 
      v-if="pixiAppInstance"
      ref="userModelRef" 
      :app="pixiAppInstance as PIXI.Application"
      type="user"
      modelPath="/live2d/miku/runtime/miku.model3.json" 
      :initialX="200" 
      :initialY="300" 
      :scale="0.2"
    />
    <Live2DModel 
      v-if="pixiAppInstance"
      ref="partnerModelRef"
      :app="pixiAppInstance as PIXI.Application"
      type="partner"
      modelPath="/live2d/miku/runtime/miku.model3.json"   homens diferentes
      :initialX="600" 
      :initialY="300" 
      :scale="0.2"
    />
  </div>
  <!-- 可以添加一些按钮来测试模型控制 -->
  <div class="controls">
    <button @click="triggerUserMotion">User Motion</button>
    <button @click="triggerPartnerExpression">Partner Expression</button>
  </div>
</template>

<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref, nextTick } from 'vue';
import * as PIXI from 'pixi.js';
import Live2DModel from '../components/Live2DModel.vue';

// 为子组件引用创建类型别名
type Live2DModelComponent = InstanceType<typeof Live2DModel>;

const pixiContainerRef = ref<HTMLDivElement | null>(null);
const pixiAppInstance = ref<PIXI.Application | null>(null);

const userModelRef = ref<Live2DModelComponent | null>(null);
const partnerModelRef = ref<Live2DModelComponent | null>(null);

let resizeObserver: ResizeObserver | null = null;

onMounted(async () => {
  await nextTick(); 
  if (pixiContainerRef.value) {
    const app = new PIXI.Application({
        width: pixiContainerRef.value.clientWidth,
        height: pixiContainerRef.value.clientHeight,
        backgroundAlpha: 0, 
        autoStart: true,
        resolution: window.devicePixelRatio || 1,
        autoDensity: true,
    });

    pixiContainerRef.value.appendChild(app.view as unknown as Node);
    pixiAppInstance.value = app;

    resizeObserver = new ResizeObserver(entries => {
      for (let entry of entries) {
        const { width, height } = entry.contentRect;
        if (pixiAppInstance.value) {
          pixiAppInstance.value.renderer.resize(width, height);
        }
      }
    });
    resizeObserver.observe(pixiContainerRef.value);
  }
});

onBeforeUnmount(() => {
  if (resizeObserver && pixiContainerRef.value) {
    resizeObserver.unobserve(pixiContainerRef.value);
  }
  if (pixiAppInstance.value) {
    pixiAppInstance.value.destroy(true, { children: true, texture: true }); // 移除了 basePath
    pixiAppInstance.value = null;
  }
});

const triggerUserMotion = () => {
  userModelRef.value?.playMotion('Flick', undefined); 
};

const triggerPartnerExpression = () => {
  partnerModelRef.value?.setExpression('F01'); 
};

</script>

<style scoped>
.live2d-view-container {
  width: 100%;
  height: 80vh; 
  position: relative;
  border: 1px solid #ccc; 
  overflow: hidden; 
}

.controls {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  gap: 10px;
}

</style> 