<template>
  <div class="layout-pool">
    <el-row class="layout-search">
      <el-col :span="5">
        <FileInput @change="files => drawMaterial(files)">
          <el-tooltip placement="top" :hide-after="0" content="上传素材">
            <el-button type="primary">
              <IconUpload />
            </el-button> 
          </el-tooltip>
        </FileInput>
      </el-col>
      <el-col :span="19">
        <el-input :prefix-icon="Search" placeholder="搜索素材"></el-input>
      </el-col>
    </el-row>
    <el-tabs v-model="activeMaterial" class="layout-tabs">
      <el-tab-pane label="推荐素材" name="data">
        <LinePool @select="line => drawLine(line)"/>
        <PathPool @select="path => drawPath(path)"/>
      </el-tab-pane>
      <el-tab-pane label="我的收藏" name="self">我的模板</el-tab-pane>
      <el-tab-pane label="我的购买" name="team">团队模板</el-tab-pane>
    </el-tabs>
    
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { Search } from '@element-plus/icons-vue'
import { nanoid } from 'nanoid'

import { PathPoolItem, LinePoolItem, ElementNames } from '@/types/elements'
import { loadSVGFromURL, Group } from 'fabric'
import { getImageDataURL } from '@/utils/image'
import { useTemplatesStore } from '@/store'
import useCanvas from '@/views/Canvas/useCanvas'
import useCenter from '@/views/Canvas/useCenter'
import PathPool from './MaterialComponents/PathPool.vue'
import LinePool from './MaterialComponents/LinePool.vue'
import useCreateElement from '@/hooks/useCreateElement'
import { CanvasElement, TextboxElement } from '@/types/canvas'


const { createLineElement, createPathElement } = useCreateElement()
const activeMaterial = ref('data')

const drawLine = (line: LinePoolItem) => {
  createLineElement(line.path)
}

const drawPath = (shape: PathPoolItem) => {
  createPathElement(shape.path, shape.viewBox)
}

const svgRevier = (option: any) => {
  // console.log("option:", option.type)

}

const svgCallback = (objects: CanvasElement[], options: any, elements: any, allElements: any) => {
  const templatesStore = useTemplatesStore()
  const [ canvas ] = useCanvas()
  const { centerPoint } = useCenter()
  objects.forEach(item => {
    item.id = nanoid(10)
    if (item.type === ElementNames.TEXT) {
      item.type = ElementNames.TEXTBOX
    }
    item.name = item.type
  })
  const svgGroup = new Group(objects, { 
    id: nanoid(10),
    name: ElementNames.GROUP, 
    // interactive: true, 
    // subTargetCheck: true, 
    left: centerPoint.x,
    top: centerPoint.y
  })
  const left = svgGroup.left - svgGroup.width / 2, top = svgGroup.top - svgGroup.height / 2
  svgGroup.set({left, top})
  canvas.add(svgGroup)
  canvas.setActiveObject(svgGroup)
  templatesStore.modifedElement()
}

const drawMaterial = async (files: FileList) => {
  const materialFile = files[0]
  if (!materialFile) return
  const dataURl = await getImageDataURL(materialFile)
  loadSVGFromURL(dataURl, svgCallback, svgRevier, {})
  // getImageDataURL(imageFile).then(dataURL => createImageElement(dataURL))
}

</script>

<style lang="scss" scoped>
.layout-search {
  margin: 0 auto;
  width: 80%;
  padding: 20px 10px 10px;
}
.layout-upload {
  justify-content: center;
}
.layout-tabs {
  width: 90%;
  margin: 0 auto;
}
.layout-templates {
  display: flex;
  flex-wrap: wrap;
  padding: 2px;
  .thumbnail {
    display: flex;
    width: 124px;
    margin: 2px;
  }
  .thumbnail img {
    outline: 1px solid $borderColor;
    margin: 0 5px;
    cursor: pointer;
    &:hover {
      outline-color: $themeColor;
    }
  }
}
</style>
<style>
.el-tabs .el-tabs__nav {
  display: flex;
  flex-wrap: nowrap;
  justify-content: space-around;
  width: 100%;
}
</style>