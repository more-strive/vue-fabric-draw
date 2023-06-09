<template>
  <div class="thumbnail-slide"
    :style="{
      width: props.size + 'px',
      height: height + 'px',
    }"
  >
    <div v-if="visible">
      <canvas ref="thumbnailTemplate"></canvas>
    </div>
    <div class="placeholder" v-else>加载中 ...</div>
  </div>
</template>

<script lang="ts" setup>
import { computed, onMounted, PropType, ref, watchEffect, watch } from 'vue'
import { StaticCanvas, Gradient, Pattern, Rect, Image } from 'fabric'
import { Template } from '@/types/canvas'
import { TransparentFill } from '@/configs/background'

const RectFillType = 'RectFillType'

const props = defineProps({
  template: {
    type: Object as PropType<Template>,
    required: true,
  },
  size: {
    type: Number,
    required: true,
  },
  visible: {
    type: Boolean,
    default: true,
  },
})

// const viewportRatio = ref<number>(props.template.height / props.template.width) 
const viewportRatio = computed(() => props.template.height / props.template.width)
const height = computed(() => props.size * viewportRatio.value)
const thumbnailTemplate = ref()
// let thumbnailCanvas: StaticCanvas 
const thumbCanvas = ref<StaticCanvas | undefined>(undefined)

onMounted(() => {
  thumbCanvas.value = new StaticCanvas(thumbnailTemplate.value, {
    width: props.size,
    height: props.size * viewportRatio.value,
    backgroundColor: props.template.workSpace.fillType === 0 ? props.template.workSpace.fill as string : '#fff'
  })
  setThumbnailElement()
})

watch(props ,() => {
  if (!thumbCanvas.value) return
  setThumbnailElement()
}, { deep: true, immediate: true })

const setThumbnailElement = async () => {
  const width = props.template.width / props.template.zoom
  const height = props.template.height / props.template.zoom
  if (!thumbCanvas.value) return
  await thumbCanvas.value.loadFromJSON(props.template)
  thumbCanvas.value.getObjects().forEach(obj => {
    if (typeof obj.left === 'number' && typeof obj.top === 'number') {
      obj.left += width / 2
      obj.top += height / 2
    }
  })
  thumbCanvas.value.width = props.size
  thumbCanvas.value.height = props.size * viewportRatio.value
  thumbCanvas.value.setZoom(props.size / width)
  setThumbnailBackground(width, height)
}

const setThumbnailBackground = async (width: number, height: number) => {
  const workSpace = props.template.workSpace
  if (!workSpace) return
  if (!thumbCanvas.value) return
  // 删除渐变填充矩形
  thumbCanvas.value.backgroundColor = TransparentFill
  thumbCanvas.value.remove(...thumbCanvas.value.getObjects(RectFillType))
  const left = workSpace.left, top = workSpace.top, angle = workSpace.angle 
  let scaleX = workSpace.scaleX, scaleY = workSpace.scaleY
  // 纯色
  if (workSpace.fillType === 0) {
    thumbCanvas.value.backgroundColor = workSpace.fill as string
  }
  // 图片
  else if (workSpace.fillType === 1) {
    if (!workSpace.imageURL) return
    if (workSpace.imageSize === 'repeat') {
      const patternFill = workSpace.fill as Pattern
      const patternRect = new Rect({
        type: RectFillType,
        width: width,
        height: height,
        fill: patternFill
      })
      thumbCanvas.value.add(patternRect)
      thumbCanvas.value.sendObjectToBack(patternRect)
    }
    else {
      const backgroundImage = await Image.fromURL(workSpace.imageURL)
      // let scaleX = 1, scaleY = 1
      if (workSpace.imageSize === 'cover') {
        scaleX = width / (backgroundImage.width ? backgroundImage.width : 1), scaleY = height / (backgroundImage.height ? backgroundImage.height : 1)
      }
      backgroundImage.set({left, top, angle, scaleX, scaleY})
      thumbCanvas.value.set({backgroundImage})
    }
  }
  // 渐变
  else if (workSpace.fillType === 2) {
    const gradientFill = workSpace.fill as Gradient<'linear' | 'radial'>
    const gradientRect = new Rect({
      type: RectFillType,
      width: width,
      height: height,
      fill: gradientFill
    })
    thumbCanvas.value.add(gradientRect)
    thumbCanvas.value.sendObjectToBack(gradientRect)
  }
  // 网格
  else if (workSpace.fillType === 3) {
    if (!workSpace.gaidImageURL) return
    const backgroundImage = await Image.fromURL(workSpace.gaidImageURL)
    backgroundImage.set({left, top, angle, scaleX, scaleY})
    thumbCanvas.value.set({backgroundImage})
  }
  // 底纹
  else if (workSpace.fillType === 4) {
    if (!workSpace.shadingImageURL) return
    const backgroundImage = await Image.fromURL(workSpace.shadingImageURL)
    backgroundImage.set({left, top, angle, scaleX, scaleY})
    thumbCanvas.value.set({backgroundImage})
  }
  thumbCanvas.value.renderAll()
}

</script>

<style lang="scss" scoped>
.thumbnail-slide {
  background-color: #fff;
  overflow: hidden;
  user-select: none;
}
.elements {
  transform-origin: 0 0;
}
.background {
  width: 100%;
  height: 100%;
  background-position: center;
  position: absolute;
}
.placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>