<template>
  <div class="shape-style-panel">
    <ElementFill />
    <el-divider />
    <ElementOutline />
    <el-divider />
    <ElementShadow />
    <el-divider />
    <ElementOpacity />
  </div>
</template>

<script lang="ts" setup>
import { computed, ref, watch } from 'vue'
import { storeToRefs } from 'pinia'
import { useMainStore } from '@/store'


import ElementOpacity from '../Components/ElementOpacity.vue'
import ElementOutline from '../Components/ElementOutline.vue'
import ElementShadow from '../Components/ElementShadow.vue'
import ElementFill from '../Backgrounds/ElementFill.vue'
import useCanvas from '@/views/Canvas/useCanvas'
import { PathElement } from '@/types/canvas'

const mainStore = useMainStore()
const { canvasObject } = storeToRefs(mainStore)

const handleElement = computed(() => canvasObject.value as PathElement)


// 填充类型
const fillType = ref<number>(0)


watch(handleElement, () => {
  if (!handleElement.value) return
  if (typeof handleElement.value.fill !== 'string') fillType.value = 1
})


</script>

<style lang="scss" scoped>
.shape-style-panel {
  user-select: none;
}
.row {
  width: 100%;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.font-size-btn {
  padding: 0;
}
.slider {
  flex: 3;
}
.mb-10 {
  margin-bottom: 10px;
}
.full-row {
  flex: 1;
  width: 100%;
}

.gradient-box {
  display: flex;
  flex: 1;
  .el-button {
    width: 100%;
  }
}

.mt-10 {
  margin-top: 10px;
}
.full-group {
  display: flex;
  flex: 1;
  .el-button {
    width: 50%;
  }
}

.tooltip-popover {
  .el-button {
    width: 100%;
    border-radius: 0;
  }
  .font-color {
    border-top-left-radius: 4px;
    border-bottom-left-radius: 4px;
    border-right: 0;
  }
  .high-light {
    border-right: 0;
  }
}
.font-size {
  border-top-left-radius: 0;
  border-bottom-left-radius: 0;
}
</style>