<template>
  <div class="card">
    <div class="card-header">
      <span>Markdown 组件</span>
    </div>

    <div class="describe">
      采用开源组件
      <el-link type="primary" href="https://imzbf.github.io/md-editor-v3/zh-CN">md-editor-v3</el-link>
      ，代码位置：src/views/components/markdown
    </div>

    <MdEditor
      class="md-editor-custom"
      ref="editorWrapRef"
      v-model="text"
      previewTheme="default"
      codeTheme="atom"
      :footers="['markdownTotal']"
      :theme="theme"
      :language="language"
      :toolbars="toolbars"
      @onUploadImg="onUploadImg"
      @onSave="onSave"
      @onHtmlChanged="onHtmlChanged"
    >
      <template #defToolbars>
        <Emoji />
      </template>
    </MdEditor>
  </div>
</template>

<script setup>
defineOptions({
  name: 'ComponentsMarkdown'
})
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from 'vue'
import { MdEditor } from 'md-editor-v3'
import 'md-editor-v3/lib/style.css'
import { Emoji } from '@vavt/v3-extension'
import '@vavt/v3-extension/lib/asset/Emoji.css'
import * as echarts from 'echarts'

import { useSettingStore } from '@/stores/modules/setting'
import { ElMessage } from 'element-plus'

const settingStore = useSettingStore()

const text = ref(`# 在 Markdown 中渲染 ECharts

下面是一个 ECharts 柱状图示例（使用 HTML 容器 + data-option）：

<div class="echarts" style="width: 100%; height: 320px;" data-option='{
  "tooltip": {"trigger": "axis"},
  "xAxis": {"type": "category", "data": ["Mon","Tue","Wed","Thu","Fri","Sat","Sun"]},
  "yAxis": {"type": "value"},
  "series": [{"type": "bar", "data": [120, 200, 150, 80, 70, 110, 130]}]
}'></div>

下面是一个 ECharts 折线图示例：

<div class="echarts" style="width: 100%; height: 320px;" data-option='{
  "tooltip": {"trigger": "axis"},
  "legend": {"data": ["邮件营销", "联盟广告"]},
  "grid": {"left": "3%", "right": "4%", "bottom": "3%", "containLabel": true},
  "xAxis": {"type": "category", "boundaryGap": false, "data": ["Mon","Tue","Wed","Thu","Fri","Sat","Sun"]},
  "yAxis": {"type": "value"},
  "series": [
    {"name": "邮件营销", "type": "line", "smooth": true, "data": [120, 132, 101, 134, 90, 230, 210]},
    {"name": "联盟广告", "type": "line", "smooth": true, "data": [220, 182, 191, 234, 290, 330, 310]}
  ]
}'></div>
`)

const theme = computed(() => {
  return settingStore.isDark ? 'dark' : 'light'
})
const language = computed(() => {
  return settingStore.lang === 'en' ? 'en-US' : 'zh-CN'
})

const toolbars = [
  0,
  'bold',
  'underline',
  'italic',
  'strikeThrough',
  '-',
  'title',
  'sub',
  'sup',
  'quote',
  'unorderedList',
  'orderedList',
  'task',
  '-',
  'codeRow',
  'code',
  'link',
  'image',
  'table',
  'mermaid',
  '-',
  'revoke',
  'next',
  'save',
  '=',
  'pageFullscreen',
  'fullscreen',
  'preview',
  'previewOnly',
  'htmlPreview',
  'catalog',
  'github'
]

const editorWrapRef = ref(null)
const echartsInstanceSet = new Set()

const disposeAllCharts = () => {
  echartsInstanceSet.forEach(instance => {
    instance.dispose()
  })
  echartsInstanceSet.clear()
}

const renderEchartsInPreview = async () => {
  await nextTick()
  const wrapEl = editorWrapRef?.value?.$el || editorWrapRef?.value
  if (!wrapEl) {
    return
  }
  const containers = wrapEl.querySelectorAll('.echarts')
  containers.forEach(container => {
    try {
      // 解析 data-option
      const optionStr = container.getAttribute('data-option') || '{}'
      const option = JSON.parse(optionStr)
      // 如果已有实例先销毁
      const prev = echarts.getInstanceByDom(container)
      if (prev) {
        prev.dispose()
        echartsInstanceSet.delete(prev)
      }
      const instance = echarts.init(container)
      instance.setOption(option)
      echartsInstanceSet.add(instance)
    } catch {
      // 解析或渲染失败忽略，以免影响编辑器
    }
  })
}

const onHtmlChanged = () => {
  renderEchartsInPreview()
}

onMounted(() => {
  renderEchartsInPreview()
  window.addEventListener('resize', resizeAllCharts)
})

const resizeAllCharts = () => {
  echartsInstanceSet.forEach(instance => {
    instance.resize()
  })
}

onBeforeUnmount(() => {
  window.removeEventListener('resize', resizeAllCharts)
  disposeAllCharts()
})

const onUploadImg = async (files, callback) => {
  ElMessage.info('上传')
  const res = await Promise.all(
    files.map(file => {
      return new Promise(() => {
        const form = new FormData()
        form.append('file', file)
      })
    })
  )

  callback(res.map(item => item.data.url))
}

const onSave = (v, h) => {
  h.then(() => {
    ElMessage.info('save')
  })
}
</script>

<style lang="scss" scoped>
@use './index.scss' as *;
</style>
