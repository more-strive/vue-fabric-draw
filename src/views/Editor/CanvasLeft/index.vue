<template>
  <div>
    <div class="left-top-tabs">
      <div 
        class="top-tab" 
        :class="{ 'left-active': tab.key === poolType }"
        v-for="tab in topTabs"
        :key="tab.key"
        @click="setPoolType(tab.key)"
        >
        <div><SvgIcon :icon-class="tab.icon" className="svg-size"/></div>
        <div class="left-name">{{tab.label}}</div>
      </div>
    </div>
    <div class="left-bottom-tabs">
      <el-affix position="bottom" :offset="110" style="width: calc(100%); height: 0;">
        <div 
          class="bottom-tab" 
          :class="{ 'left-active': 'layer' === poolType }"
          @click="setPoolType('layer')"
          >
          <div><SvgIcon icon-class="layer" className="svg-size"/></div>
          <div class="left-name">图层</div>
        </div>
      </el-affix>
      <el-affix position="bottom" :offset="60" style="width: calc(100%); height: 0;">
        <div 
          class="bottom-tab" 
          :class="{ 'left-active': 'help' === poolType }"
          ref="helpRef"
          @click="setPoolType('help')"
          >
          <div><SvgIcon icon-class="help" className="svg-size"/></div>
          <div class="left-name">帮助</div>
        </div>
      </el-affix>

      <el-popover placement="right" trigger="click" :popper-style="{padding: 0}" @before-enter="showHelp" @hide="hideHelp" ref="helpPopoverRef" :virtual-ref="helpRef" virtual-triggering>
        <el-row class="help-pop-row">
          <IconGuideBoard class="help-pop-icon"/>
          <span class="help-pop-text">新手入门</span>
        </el-row>
        <el-row class="help-pop-row">
          <IconVideoTwo class="help-pop-icon"/>
          <span class="help-pop-text">使用教程</span>
        </el-row>
        <el-row class="help-pop-row" @click="hasHotkey = true">
          <IconKeyboardOne class="help-pop-icon"/>
          <span class="help-pop-text">快捷键</span>
        </el-row>
        <el-row class="help-pop-row">
          <IconEdit class="help-pop-icon"/>
          <span class="help-pop-text">反馈建议</span>
        </el-row>
        <el-row class="help-pop-row">
          <IconHeadsetOne class="help-pop-icon"/>
          <span class="help-pop-text">在线客服</span>
        </el-row>
      </el-popover>

      <el-drawer v-model="hasHotkey" :with-header="false" size="320">
        <HotkeyDoc/>
      </el-drawer>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { useMainStore } from '@/store'
import { PoolType } from '@/types/common'
import { storeToRefs } from 'pinia'
import { ref } from 'vue'
import HotkeyDoc from './components/HotkeyDoc.vue'

const mainStore = useMainStore()

const { poolType, poolShow } = storeToRefs(mainStore)

const hasHelp = ref(false)
const helpRef = ref()
const helpPopoverRef = ref()
const hasHotkey = ref(false)

interface TabItem {
  key: PoolType
  label: string
  icon: string
  index: number
}

const topTabs: TabItem[] = [
  { key: 'editor', label: '编辑', icon: `editor`, index: 0},
  { key: 'template', label: '模板', icon: `template`, index: 1},
  { key: 'material', label: '素材', icon: `material`, index: 2 },
  { key: 'text', label: '文字', icon: 'text', index: 3 },
  { key: 'image', label: '图片', icon: 'picture', index: 4 },
  { key: 'toolkit', label: '工具', icon: 'toolkit', index: 5 },
]

const setPoolType = (tab: PoolType) => {
  if (poolShow.value && tab === poolType.value) {
    poolShow.value = false
  } 
  else {
    poolShow.value = tab !== 'editor' && tab !== 'help' ? true : false
  }
  mainStore.setPoolType(tab)
}

const showHelp = () => {
  hasHelp.value = true
}

const hideHelp = () => {
  hasHelp.value = false
}

</script>

<style lang="scss" scoped>
.left-top {
  display: flex;
  height: 40px;
}
.left-top-tabs {
  display: flex;
  flex-direction: column;
  width: 100%;
}
.top-tab {
  width: 100%;
  height: 60px;
  text-align: center;
  font-size: 12px;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  justify-content: center;

  .left-icon {
    font-size: 20px;
  }
}
.left-active {
  color: $themeColor
}
.left-name {
  font-size: 14px;
  line-height: 1.2;
}
.svg-size {
  font-size: 20px;
}
.left-active::before {
  background-color: $themeColor;
  border-radius: 4px;
  content: "";
  height: 41px;
  left: -3px;
  position: absolute;
  top: calc(var(--index) * 72px + 23px);
  transition: top .2s;
  width: 6px;
  z-index: 20;
}
.left-content {
  position: relative;
  width: 300px;
  left: 50px;
  top: -360px;
  height: 100%;
  z-index: 1;
  background: #fff;
  border-left: 1px solid $borderColor;
  border-right: 1px solid $borderColor;
  transition: left 0.5s linear, right 0.5s linear;
}
.left-close {
  cursor: default;
  left: -320px;
  position: relative;;
  // z-index: 1;
}
.layout-toggle {
  background: #fff;
  cursor: pointer;
  height: 88px;
  position: absolute;
  right: -16px;
  top: 50%;
  transform: translateY(-50%);
  transition: right .1s linear;
  width: 16px;
  z-index: 1;
  border-top-right-radius: 20px;
  border-bottom-right-radius: 20px;
  display: flex;
  align-items: center;
  border-top: 1px solid $borderColor;
  border-bottom: 1px solid $borderColor;
  border-right: 1px solid $borderColor;
}
.bottom-tab {
  height: 50px;
  padding-bottom: 10px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  background: #fff;

  .help-handle {
    font-size: 20px;
  }
}
.has-help {
  color: $themeColor;
}
.help-pop-row {
  font-size: 15px;
  padding: 10px 25px;
  cursor: pointer;
  .help-pop-icon {
    font-size: 20px;
  }

  .help-pop-text {
    padding-left: 10px;
  } 
}
.help-pop-row:hover {
  background-color: $hoverColor;
}
</style>