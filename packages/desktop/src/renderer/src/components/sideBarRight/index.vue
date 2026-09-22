<template>
  <div
    class="side-bar-right"
    :style="{
      width: `${finalSideBarWidth}px`,
      minWidth: `${activePanel ? 220 : 45}px`
    }"
  >
    <div
      v-show="activePanel"
      class="panel-column"
    >
      <tree
        v-if="activePanel === 'files'"
        :project-tree="projectTree"
        :tabs="tabs"
      />
      <side-bar-search v-else-if="activePanel === 'search'" />
      <toc v-else-if="activePanel === 'toc'" />
    </div>

    <div class="icon-column">
      <ul>
        <li
          v-for="(c, index) of sideBarIcons"
          :key="index"
          :title="c.name()"
          :class="{ active: c.id === activePanel }"
          @click="handleIconClick(c.id)"
        >
          <component :is="c.icon" />
        </li>
      </ul>
      <ul class="bottom">
        <li
          v-for="(c, index) of sideBarBottomIcons"
          :key="index"
          :title="c.name()"
          @click="handleBottomClick(c.id)"
        >
          <component :is="c.icon" />
        </li>
      </ul>
    </div>

    <div
      v-show="activePanel"
      class="drag-bar"
      @mousedown="handleResizeStart"
    />
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { storeToRefs } from 'pinia'
import { useProjectStore } from '@/store/project'
import { useEditorStore } from '@/store/editor'
import { sideBarIcons, sideBarBottomIcons } from '@/components/sideBar/help'
import Tree from '@/components/sideBar/tree.vue'
import SideBarSearch from '@/components/sideBar/search.vue'
import Toc from '@/components/sideBar/toc.vue'

const PANEL_KEY = 'right-side-bar-panel'
const WIDTH_KEY = 'right-side-bar-width'
const DEFAULT_WIDTH = 280
const MIN_WIDTH = 220

const readInitialPanel = (): string => {
  const value = localStorage.getItem(PANEL_KEY)
  return value === null ? 'toc' : value
}

const readInitialWidth = (): number => {
  const value = Number(localStorage.getItem(WIDTH_KEY))
  return Number.isFinite(value) && value >= MIN_WIDTH ? value : DEFAULT_WIDTH
}

const projectStore = useProjectStore()
const editorStore = useEditorStore()

const { projectTree } = storeToRefs(projectStore)
const { tabs } = storeToRefs(editorStore)

const activePanel = ref<string>(readInitialPanel())
const sideBarWidth = ref<number>(readInitialWidth())

const finalSideBarWidth = computed<number>(() => {
  return activePanel.value ? Math.max(sideBarWidth.value, MIN_WIDTH) : 45
})

const handleIconClick = (name: string): void => {
  activePanel.value = activePanel.value === name ? '' : name
  localStorage.setItem(PANEL_KEY, activePanel.value)
}

const handleBottomClick = (name: string): void => {
  if (name === 'settings') {
    projectStore.OPEN_SETTING_WINDOW()
  }
}

const handleResizeStart = (event: MouseEvent): void => {
  const startX = event.clientX
  const startWidth = sideBarWidth.value

  const handleMouseMove = (moveEvent: MouseEvent): void => {
    const nextWidth = startWidth + (startX - moveEvent.clientX)
    sideBarWidth.value = Math.max(nextWidth, MIN_WIDTH)
  }

  const handleMouseUp = (): void => {
    localStorage.setItem(WIDTH_KEY, String(sideBarWidth.value))
    document.removeEventListener('mousemove', handleMouseMove, false)
    document.removeEventListener('mouseup', handleMouseUp, false)
  }

  document.addEventListener('mousemove', handleMouseMove, false)
  document.addEventListener('mouseup', handleMouseUp, false)
}
</script>

<style scoped>
.side-bar-right {
  display: flex;
  flex-shrink: 0;
  flex-grow: 0;
  height: 100vh;
  position: relative;
  color: var(--sideBarColor);
  user-select: none;
  background: var(--sideBarBgColor);
  border-left: 1px solid var(--itemBgColor);
}

.panel-column {
  flex: 1;
  width: calc(100% - 45px);
  overflow: hidden;
}

.icon-column {
  height: 100%;
  width: 45px;
  flex: 0 0 45px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding-top: 28px;
  box-sizing: border-box;
}

.icon-column > ul {
  opacity: 1;
}

.icon-column ul {
  list-style: none;
  display: flex;
  flex-direction: column;
  margin: 0;
  padding: 0;
}

.icon-column ul > li {
  width: 45px;
  height: 45px;
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: space-around;
  align-items: center;
  cursor: pointer;
}

.icon-column ul > li > svg {
  width: 18px;
  height: 18px;
  color: var(--sideBarIconColor);
  opacity: 1;
  transition: transform 0.25s ease-in-out;
}

.icon-column ul > li.active > svg {
  color: var(--themeColor);
}

.drag-bar {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  height: 100%;
  width: 3px;
  cursor: col-resize;
}

.drag-bar:hover {
  border-left: 2px solid var(--iconColor);
}
</style>
