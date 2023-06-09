<template>
  <div class="layer-content">
    <div 
      class="element-content" 
      :class="handleElement && handleElement.id === element.id ? 'layer-active' : ''" 
      @click.stop="selectElement(element.id)"
      @mousemove.stop="mouseoverElement(element.id)"
      @mouseleave.stop="mouseleaveElement"
      :style="{ marginLeft: `${props.index * 10}px` }"
      >
      <div class="element-info">
        <div v-if="element.type === ElementNames.GROUP">
          <el-tooltip placement="top" :hide-after="0" :content="element.isShow ? '收回' : '展开'">
            <IconExpandDownOne v-if="element.isShow" class="common-icon" @click.stop="showElement(element)"/>
            <IconFoldUpOne v-else class="common-icon" @click.stop="showElement(element)"/>
          </el-tooltip>
        </div>
        <el-tooltip placement="top" :hide-after="0" content="拖拽" v-if="!element.group && element.type !== ElementNames.GROUP">
          <IconApplicationMenu class="common-icon" v-if="!element.group && element.type !== ElementNames.GROUP"/>
        </el-tooltip>
        <div class="element-type">{{ element.name }}</div>
        <div class="element-text" v-if="element.type === ElementNames.TEXTBOX || element.type === ElementNames.TEXT">{{ (element as TextboxElement).text }}</div>
        
      </div>
      
      <div class="element-handler">
        <el-tooltip placement="top" :hide-after="0" :content="element.visible ? '隐藏' : '显示'">
          <IconPreviewOpen class="common-icon" v-if="element.visible" @click.stop="visibleElement(element.id, false)"/>
          <IconPreviewClose class="common-icon" v-else @click.stop="visibleElement(element.id, true)"/>
        </el-tooltip>
        <el-tooltip placement="top" :hide-after="0" :content="element.lockMovementX && element.lockMovementY ? '解锁' : '锁定'">
          <IconLock class="common-icon" v-if="element.lockMovementX && element.lockMovementY" @click.stop="lockElement(element.id, false)"/>
          <IconUnlock class="common-icon" v-else @click.stop="lockElement(element.id, true)"/>
        </el-tooltip>
        <el-tooltip placement="top" :hide-after="0" content="删除">
          <IconDelete class="common-icon" @click.stop="deleteElement(element.id)"/>
        </el-tooltip>
        <div v-if="element.type === ElementNames.TEXTBOX || element.type === ElementNames.TEXT">
          <el-tooltip placement="top" :hide-after="0" :content="element.isCheck ? '取消可变' : '可变数据'">
            <IconCheckOne class="common-icon" v-if="element.isCheck" @click.stop="checkElement(element.id, false)"/>
            <IconRound class="common-icon" v-else  @click.stop="checkElement(element.id, true)"/>
          </el-tooltip>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, PropType } from 'vue'
import { CanvasOption } from '@/types/option'
import { CanvasElement, TextboxElement } from '@/types/canvas'
import { ElementNames } from '@/types/elements'

import { useMainStore } from '@/store'
import { storeToRefs } from 'pinia'
import useHandleElement from "@/hooks/useHandleElement"

const { 
  selectElement, 
  visibleElement, 
  lockElement, 
  deleteElement, 
  showElement, 
  mouseoverElement, 
  mouseleaveElement,
  checkElement,
} = useHandleElement()

const props = defineProps({
  element: {
    type: Object as PropType<CanvasElement>,
    required: true,
  },
  index: {
    type: Number,
    required: true,
  }
})

const mainStore = useMainStore()
const { canvasObject } = storeToRefs(mainStore)

const handleElement = computed(() => {
  return canvasObject.value as CanvasElement
})
</script>
<style lang="scss" scoped>
.layout-search {
  margin: 0 auto;
  width: 68%;
  padding: 20px 10px 10px;
}
.layer-content {
  padding: 0 10px;
  .element-content {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 30px;
    padding: 5px;
    border: 1px solid $borderColor;
    border-radius: 5px;
    margin-bottom: 5px;

    .element-info {
      display: flex;
      align-items: center;
    }

    .element-handler {
      display: flex;
    }

    &:not(.group-btn):hover {
      border: 1px solid $themeColor;
    }
  }
  .layer-active {
    border: 1px solid $themeColor;
  }
}
.element-type {
  width: 58px;
  margin-left: 10px;
  font-size: 12px;
}

.element-text {
  width: 50px;
  font-size: 12px;
  text-align: center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.common-icon {
  width: 24px;
  height: 24px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: $borderRadius;

  &:not(.group-btn):hover {
    background-color: #f1f1f1;
  }
}
</style>