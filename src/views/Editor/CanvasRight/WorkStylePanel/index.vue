<template>
  <div class="slide-design-panel">
    
    <div class="title"><b>画布尺寸</b></div>
    <div class="mb-10">
      <el-row>
        <el-col :span="11">
          <el-input v-model="canvasWidth" :value="(Math.round(canvasWidth * 100/100))" @change="changeTemplateWidth" oninput="value=value.replace(/[^\d]/g,'')">
            <template #prepend>宽</template>
          </el-input>
        </el-col>
        <el-col :span="2" class="fixed-ratio">
          <el-tooltip effect="dark"  placement="top" content="解除宽高比" v-if="isFixed">
            <IconLock class="icon-btn" @click="changeFixedRatio(false)"/>
          </el-tooltip>
          <el-tooltip effect="dark" placement="top" content="锁定宽高比" v-else>
            <IconUnlock class="icon-btn" @click="changeFixedRatio(true)"/>
          </el-tooltip>
        </el-col>
        <el-col :span="11">
          <el-input v-model="canvasHeight" :value="(Math.round(canvasHeight * 100/100))" @change="changeTemplateHeight" oninput="value=value.replace(/[^\d]/g,'')">
            <template #prepend>高</template>
          </el-input>
        </el-col>
      </el-row>
    </div>
    <div class="mb-10">
      <el-row>
        <el-col :span="11">
          <el-input v-model="clip" @change="changeTemplateClip" oninput="value=value.replace(/[^\d]/g,'')" :disabled="unitMode === 1">
            <template #prepend>
              <el-tooltip placement="top" :hide-after="0" content="出血线">
                <IconCuttingOne/>
              </el-tooltip>
            </template>
          </el-input>
        </el-col>
        <el-col :span="2" class="fixed-ratio">
          <el-tooltip effect="dark" placement="top" content="圆角" v-if="isRound">
            <IconRound class="icon-btn" @click="changeWorkRound(false)"/>
          </el-tooltip>
          <el-tooltip effect="dark"  placement="top" content="直角" v-else>
            <IconRightAngle class="icon-btn" @click="changeWorkRound(true)"/>
          </el-tooltip>
        </el-col>
        <el-col :span="11">
          <el-input v-model="safe" @change="changeTemplateSafe" oninput="value=value.replace(/[^\d]/g,'')" :disabled="unitMode === 1">
            <template #prepend>
              <el-tooltip placement="top" :hide-after="0" content="安全线">
                <IconShield/>
              </el-tooltip>
            </template>
          </el-input>
        </el-col>
      </el-row>
    </div>
    <div class="mt-10">
      <el-row>
        <el-col :span="11">
          <el-select v-model="unitMode" @change="changeUnitMode">
            <template #prefix>
              <el-tooltip placement="top" :hide-after="0" content="单位">
                <IconRuler/>
              </el-tooltip>
            </template>
            <el-option v-for="item in DesignUnitMode" :key="item.id" :label="item.name" :value="item.id"></el-option>
          </el-select>
        </el-col>
        <el-col :span="2"></el-col>
        <el-col :span="11">
          <el-select v-model="sizeMode">
            <template #prefix>
              <el-tooltip placement="top" :hide-after="0" content="模板">
                <IconIdCard/>
              </el-tooltip>
            </template>
            <el-option v-for="item in DesignSizeMode" :key="item.id" :label="item.name" :value="item.id" :disabled="item.disabled"></el-option>
          </el-select>
        </el-col>
      </el-row>
    </div>

    <el-divider />

    <div class="title"><b>画布填充</b></div>
    <div class="row">
      <el-button class="full-row" @click="changeAllBackgroud">应用画布到全部</el-button>
    </div>
    <Backgrounds />

    <el-divider />

    <div class="title"><b>画布蒙版</b></div>
    <el-row>
      <el-col :span="7" class="slider-name">不透明度：</el-col>
      <el-col :span="13">
        <el-slider :min="0.1" :max="1" :step="0.01" v-model="opacity" @change="changeMaskOpacity"></el-slider>
      </el-col>
      <el-col :span="4" class="slider-num">{{ opacity }}</el-col>
    </el-row>

    <!-- <GridFill v-model:visible="gridColorDialog" @close="hideGridColorSelf" @save="saveGridColorSelf"></GridFill> -->
  </div>
</template>

<script lang="ts" setup>
import { Rect } from 'fabric'
import { storeToRefs } from 'pinia'
import { ElMessage } from 'element-plus'
import { ref, watch, onMounted } from 'vue'
import { mm2px, px2mm } from '@/utils/image'
import { useFabricStore, useMainStore, useTemplatesStore } from '@/store'
import { WorkSpaceClipType, WorkSpaceMaskType } from '@/configs/canvas'
import { DesignUnitMode, DesignSizeMode, MinSize, MaxSize } from '@/configs/background'
import useCanvas from '@/views/Canvas/useCanvas'
import Backgrounds from '../Backgrounds/index.vue'

const mainStore = useMainStore()
const templatesStore = useTemplatesStore()
const fabricStore = useFabricStore()

const { sizeMode, unitMode } = storeToRefs(mainStore)
const { currentTemplate } = storeToRefs(templatesStore)
const { clip, safe, zoom, opacity } = storeToRefs(fabricStore)

const canvasWidth = ref<number>(px2mm(currentTemplate.value.width / currentTemplate.value.zoom))
const canvasHeight = ref<number>(px2mm(currentTemplate.value.height / currentTemplate.value.zoom))


watch(currentTemplate, () => {
  const [ canvas ] = useCanvas()
  if (!canvas) return
  const workSpaceClip = canvas.getObjects(WorkSpaceClipType)[0]
  if (!workSpaceClip) return
  const workWidth = workSpaceClip.width ? workSpaceClip.width : 0, workHeight = workSpaceClip.height ? workSpaceClip.height : 0
  if (unitMode.value === 0) {
    canvasWidth.value = px2mm(workWidth)
    canvasHeight.value = px2mm(workHeight)
  } 
  else {
    canvasWidth.value = workWidth
    canvasHeight.value = workHeight
  }
}, { deep: true, immediate: true })

// 宽高固定比例
const isFixed = ref(false)

// 直角圆角
const isRound = ref(false)



// 网格 预定义 参数
const RECENT_GRIDS = 'RECENT_GRIDS'
const gridColorRecent = ref<[string[]]>([[]])


// 获取画布尺寸
const getCanvasSize = () => {
  let width = unitMode.value === 0 ? mm2px(canvasWidth.value) : canvasWidth.value
  let height = unitMode.value === 0 ? mm2px(canvasHeight.value) : canvasHeight.value
  width = width * zoom.value
  height = height * zoom.value
  return { width, height }
}

// 修改画布宽度
const changeTemplateWidth = () => {
  const ratio = currentTemplate.value.height / currentTemplate.value.width
  let { width, height } = getCanvasSize()
  if ((width / zoom.value) < mm2px(MinSize)) {
    ElMessage({
      message: '尺寸限制最小为' + MinSize,
      type: 'warning',
    })
    width = mm2px(MinSize) * zoom.value
  }
  if ((width / zoom.value) > mm2px(MaxSize)) {
    ElMessage({
      message: '尺寸限制最大为' + MaxSize,
      type: 'warning',
    })
    width = mm2px(MaxSize) * zoom.value
  }
  height = isFixed.value ?  (width * ratio) : height
  templatesStore.setSize(width, height, zoom.value)
  sizeMode.value = 2
  templatesStore.renderTemplate()
}

// 修改画布高度
const changeTemplateHeight = () => {
  const ratio = currentTemplate.value.height / currentTemplate.value.width
  let { width, height } = getCanvasSize()
  if ((height / zoom.value) < mm2px(MinSize)) {
    ElMessage({
      message: '尺寸限制最小为' + MinSize,
      type: 'warning',
    })
    height = mm2px(MinSize) * zoom.value
  }
  if ((height / zoom.value) > mm2px(MaxSize)) {
    ElMessage({
      message: '尺寸限制最大为' + MaxSize,
      type: 'warning',
    })
    height = mm2px(MaxSize) * zoom.value
  }
  width = isFixed.value ? (height / ratio) : width
  templatesStore.setSize(width, height, zoom.value)
  sizeMode.value = 2
  templatesStore.renderTemplate()
}

// 修改出血尺寸
const changeTemplateClip = () => {
  templatesStore.setClip(clip.value)
  templatesStore.renderTemplate()
}

// 修改安全尺寸
const changeTemplateSafe = () => {
  safe.value = Number(safe.value)
  templatesStore.renderTemplate()
}

// 修改固定宽高比
const changeFixedRatio = (fixedStatus: boolean) => {
  isFixed.value = fixedStatus
}

// 修改直角圆角
const changeWorkRound = (roundStatus: boolean) => {
  const [ canvas ] = useCanvas()
  const workSpaceclip = canvas.getObjects(WorkSpaceClipType)[0] as Rect
  isRound.value = roundStatus
  if (isRound.value) {
    workSpaceclip.rx = 10
    workSpaceclip.ry = 10
  } 
  else {
    workSpaceclip.rx = 0
    workSpaceclip.ry = 0
  }
  canvas.renderAll()
}

// 修改尺寸单位
const changeUnitMode = () => {
  if (unitMode.value === 0) {
    canvasWidth.value = px2mm(currentTemplate.value.width / currentTemplate.value.zoom)
    canvasHeight.value = px2mm(currentTemplate.value.height / currentTemplate.value.zoom)
    clip.value = 2
    safe.value = 3
    changeTemplateClip()
    changeTemplateSafe()
  } 
  else {
    canvasWidth.value = currentTemplate.value.width / currentTemplate.value.zoom
    canvasHeight.value = currentTemplate.value.height / currentTemplate.value.zoom
    clip.value = safe.value = 0
    changeTemplateClip()
    changeTemplateSafe()
  }
}

// 应用背景到所有页面
const changeAllBackgroud = () => {
  templatesStore.templates.forEach(item => item.workSpace = currentTemplate.value.workSpace)
}

// 加载缓存最近添加的网格 
onMounted(() => {
  const recentGridCache = localStorage.getItem(RECENT_GRIDS)
  if (recentGridCache) gridColorRecent.value = JSON.parse(recentGridCache)
})

// 保存缓存最近添加的网格 
watch(gridColorRecent, () => {
  const recentGridCache = JSON.stringify(gridColorRecent.value)
  localStorage.setItem(RECENT_GRIDS, recentGridCache)
}, {deep: true})

const changeMaskOpacity = () => {
  const [ canvas ] = useCanvas()
  const workMask = canvas.getObjects(WorkSpaceMaskType)[0]
  if (!workMask) return
  workMask.set('opacity', opacity.value)
  canvas.renderAll()
}
</script>

<style lang="scss" scoped>
.icon-btn {
  cursor: pointer;
}
.slide-design-panel {
  user-select: none;
}
.row {
  width: 100%;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.title {
  margin-bottom: 10px;
}

.fixed-ratio {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.slider-name {
  display: flex;
  align-items: center;
}

.mb-10 {
  margin-bottom: 10px;
}
.full-row {
  flex: 1;
  width: 100%;
}

.full-group {
  display: flex;
  flex: 1;
  .el-button {
    width: 50%;
  }
}

.full-ratio {
  display: flex;
  flex: 1;
  .el-radio-button {
    width: 50%;
  }
  .el-radio-button__inner {
    width: 100%
  }
}


.background-image {
  height: 0;
  padding-bottom: 56.25%;
  border: 1px dashed var(--el-border-color);
  border-radius: $borderRadius;
  position: relative;
  transition: all $transitionDelay;

  &:hover {
    border-color: var(--el-color-primary);
    color: var(--el-color-primary);
  }

  .content {
    @include absolute-0();

    display: flex;
    justify-content: center;
    align-items: center;
    background-position: center;
    background-size: contain;
    background-repeat: no-repeat;
    cursor: pointer;
  }
}

.theme-list {
  @include flex-grid-layout();
}
.theme-item {
  @include flex-grid-layout-children(2, 48%);

  padding-bottom: 30%;
  border-radius: $borderRadius;
  position: relative;
  cursor: pointer;

  .theme-item-content {
    @include absolute-0();

    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 8px;
    border: 1px solid $borderColor;
  }

  .text {
    font-size: 16px;
  }
  .colors {
    display: flex;
  }
  .color-block {
    margin-top: 8px;
    width: 12px;
    height: 12px;
    margin-right: 2px;
  }

  &:hover .btns {
    display: flex;
  }

  .btns {
    @include absolute-0();

    flex-direction: column;
    justify-content: center;
    align-items: center;
    display: none;
    background-color: rgba($color: #000, $alpha: .25);
  }
  .btn {
    width: 72px;
    padding: 5px 0;
    text-align: center;
    background-color: $themeColor;
    color: #fff;
    font-size: 12px;
    border-radius: $borderRadius;

    &:hover {
      background-color: #c42f19;
    }

    & + .btn {
      margin-top: 5px;
    }
  }
}

.mt-10 {
  margin-top: 10px;
}

.slider-num{
  display: flex;
  align-items: center;
  justify-content: center;
}

</style>

<style scoped>
:deep(.el-input .el-input-group__prepend) {
  padding: 0 5px;
}
:deep(.el-input .el-input-group__append) {
  padding: 0 5px;
}
:deep(.full-ratio .el-radio-button__inner) {
  width: 100%;
}
</style>