<template>
  <div class="image-style-panel">
    <div class="title">码样式：</div>
    <el-carousel type="card" :height="QRSize + 'px'" :autoplay="false" trigger="click" indicator-position="none" ref="carousel">
      <el-carousel-item v-for="item in QRCodeStyleLibs" :key="item.index">
        <div justify="center" @click="generateQRCode(item.name)">
          <img v-if="item.name !== 'C2'" :src="`data:image/svg+xml;base64,` + Base64.encode(generateQRCodeMap[item.name](getEncodeData()))" :alt="item.name">
        </div>
      </el-carousel-item>
    </el-carousel>
    <div class="title">码内容：</div>
    <div class="row">
      <el-input v-model="handleElement.codeContent" @change="updateCodeContent"></el-input>
    </div>
    <el-divider />
    <div class="title">码边距：</div>
    <div class="row">
      <el-radio-group class="full-ratio" v-model="handleElement.codeSpace" @change="updateCodeSpace">
        <el-radio-button :value="true" :label="true">无边距</el-radio-button>
        <el-radio-button :value="false" :label="false">标准边距</el-radio-button>
      </el-radio-group>
    </div>
    <div class="title">容错率：</div>
    <div class="row">
      <el-radio-group class="full-ratio" v-model="handleElement.codeError" @change="updateCodeError">
        <el-radio-button label="0">7%</el-radio-button>
        <el-radio-button label="1">15%</el-radio-button>
        <el-radio-button label="2">25%</el-radio-button>
        <el-radio-button label="3">30%</el-radio-button>
      </el-radio-group>
    </div>
    <el-divider />
    <ElementOutline />
    <el-divider />
    <ElementShadow />
    <el-divider />
  </div>
</template>

<script lang="ts" setup>
import { computed, onMounted, ref, watch } from 'vue'
import { storeToRefs } from 'pinia'
import { useMainStore, useTemplatesStore } from '@/store'
import { QRCodeStyleLibs } from '@/configs/codeStyles'
import { 
  encodeData,
  renderer25D,
  rendererRect,
  rendererRound,
  rendererRandRound,
  rendererDSJ,
  rendererRandRect,
  rendererImage,
  rendererResImage,
  rendererCircle,
  rendererLine,
  rendererLine2,
  rendererFuncA,
  rendererFuncB,
} from 'beautify-qrcode'
import { Base64 } from 'js-base64'
import { QRCodeElement } from '@/types/canvas'
import useCanvas from '@/views/Canvas/useCanvas'
import ElementOutline from '../Components/ElementOutline.vue'
import ElementShadow from '../Components/ElementShadow.vue'
const carousel = ref<HTMLFormElement>()
const QRSize = ref(118)
const mainStore = useMainStore()
const templatesStore = useTemplatesStore()
const [ canvas ] = useCanvas()
const { canvasObject } = storeToRefs(mainStore)

const generateQRCodeMap = {
  'A1': rendererRect,
  'A2': rendererRound,
  'A3': rendererRandRound,
  'SP1': rendererDSJ,
  'SP2': rendererRandRect,
  'SP3': rendererCircle,
  'B1': renderer25D,
  'C1': rendererImage,
  'C2': rendererResImage,
  'A_a1': rendererLine,
  'A_a2': rendererLine2,
  'A_b1': rendererFuncA,
  'A_b2': rendererFuncB,
}

const handleElement = computed(() => canvasObject.value as QRCodeElement)

onMounted(() => {
  if (!handleElement.value) return
  const codeItem = QRCodeStyleLibs.filter(item => item.name === handleElement.value.codeStyle)[0]
  if (codeItem.index) {
    carousel.value?.setActiveItem(codeItem.index)
  }
})

// 输入二位码内容
const updateCodeContent = () => {
  generateQRCode()
}

// 修改码边距
const updateCodeSpace = () => {
  generateQRCode()
}

// 修改容错率
const updateCodeError = () => {
  generateQRCode()
}

// 获取qrcode
const getEncodeData = (width = QRSize.value, height = QRSize.value) => {
  const codeOption = {
    text: handleElement.value.codeContent,
    width,
    height,
    correctLevel: Number(handleElement.value.codeError),
    isSpace: handleElement.value.codeSpace
  }
  return encodeData(codeOption)
}

const generateQRCode = async (style?: string) => {
  const encodeData = getEncodeData()
  if (style) handleElement.value.codeStyle = style
  if (!encodeData) return
  const src = `data:image/svg+xml;base64,` + Base64.encode(generateQRCodeMap[handleElement.value.codeStyle](encodeData))
  const qrcodeElement = canvasObject.value as QRCodeElement
  await qrcodeElement.setSrc(src)
  templatesStore.modifedElement()
  canvas.renderAll()
}
</script>

<style lang="scss" scoped>
.row {
  width: 100%;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.switch-wrapper {
  text-align: right;
}
.origin-image {
  height: 100px;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  background-color: $lightGray;
  margin-bottom: 10px;
}
.full-width-btn {
  width: 100%;
  margin-bottom: 10px;
}
.btn-icon {
  margin-right: 3px;
}

.clip {
  width: 260px;
  font-size: 12px;

  .title {
    margin-bottom: 5px;
  }
}

.title {
  margin-bottom: 10px;
}
.shape-clip {
  margin-bottom: 10px;

  @include flex-grid-layout();
}
.shape-clip-item {
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;

  @include flex-grid-layout-children(5, 16%);

  &:hover .shape {
    background-color: #ccc;
  }

  .shape {
    width: 40px;
    height: 40px;
    background-color: #e1e1e1;
  }
}
.config-margin {
  display: flex;
  flex: 1;
}
.full-ratio {
  display: flex;
  flex: 1;
  .el-radio-button {
    position: relative;
    display: inline-flex;
    outline: 0;
  }
  .el-radio-button__inner {
    width: 100%
  }
}
</style>

<style scoped>
:deep(.full-ratio .el-radio-button__inner) {
  width: 100%;
}
:deep(.full-ratio .el-radio-button) {
  position: relative;
  display: inline-flex;
  outline: 0;
  flex: 1;
  width: 25%
}
.el-carousel__item {
  border-radius: 10px;
}
.el-carousel__item div {
  color: #475669;
  opacity: 0.75;
  line-height: var(--QRSize) + 'px';
  margin: 0;
  text-align: center;
}
.el-carousel__item:nth-child(2n) {
  background-color: #99a9bf;
}
.el-carousel__item:nth-child(2n + 1) {
  background-color: #d3dce6;
}
</style>