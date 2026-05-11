<template>
  <div class="border-b border-gray-200">
    <nav class="-mb-px flex items-center space-x-4 overflow-x-auto pb-2 scrollbar-hide">
      <!-- 全部 Tab -->
      <button
        @click="$emit('update:activeProject', '')"
        :class="[
          activeProject === ''
            ? 'border-blue-500 text-blue-600'
            : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300',
          'whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm'
        ]"
      >
        全部
      </button>

      <!-- 各项目 Tab -->
      <button
        v-for="project in allProjects"
        :key="project"
        @click="$emit('update:activeProject', project)"
        :class="[
          activeProject === project
            ? 'border-blue-500 text-blue-600'
            : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300',
          'whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm'
        ]"
      >
        {{ project }}
        <span
          v-if="getBlockedCount(project) > 0"
          class="ml-2 inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-red-100 text-red-800"
        >
          {{ getBlockedCount(project) }}
        </span>
      </button>

      <!-- + 新增项目按钮 -->
      <button
        v-if="!showAddInput"
        @click="showAddInput = true"
        class="whitespace-nowrap py-4 px-1 border-b-2 border-transparent text-gray-400 hover:text-blue-500 hover:border-blue-300 font-medium text-sm transition-colors"
        title="新增项目分类"
      >
        + 新增项目
      </button>

      <!-- 新增项目输入框 -->
      <div v-if="showAddInput" class="flex items-center space-x-2 ml-2">
        <input
          ref="addInputRef"
          v-model="newProjectName"
          type="text"
          placeholder="输入项目名称"
          class="border border-gray-300 rounded-md shadow-sm py-1.5 px-3 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 w-48"
          @keydown.enter="confirmAddProject"
          @keydown.escape="cancelAddProject"
          @blur="confirmAddProject"
        />
        <button
          @click="confirmAddProject"
          class="px-3 py-1.5 bg-blue-500 text-white text-sm rounded-md hover:bg-blue-600"
        >
          确认
        </button>
        <button
          @click="cancelAddProject"
          class="px-3 py-1.5 bg-gray-200 text-gray-600 text-sm rounded-md hover:bg-gray-300"
        >
          取消
        </button>
      </div>
    </nav>
  </div>
</template>

<script setup>
import { ref, computed, nextTick, onMounted, watch } from 'vue'

const props = defineProps({
  projects: {
    type: Array,
    required: true
  },
  activeProject: {
    type: String,
    required: true
  }
})

const emit = defineEmits(['update:activeProject', 'add-project'])

// 新增项目相关状态
const showAddInput = ref(false)
const newProjectName = ref('')
const addInputRef = ref(null)

// 从 localStorage 读取手动添加的项目列表（独立于任务数据）
const getStoredProjects = () => {
  try {
    const stored = localStorage.getItem('weekly-report-manual-projects')
    return stored ? JSON.parse(stored) : []
  } catch {
    return []
  }
}
const manualProjects = ref(getStoredProjects())

// 合并项目列表：任务中出现的 + 手动添加的（去重）
const allProjects = computed(() => {
  const merged = new Set([...props.projects, ...manualProjects.value])
  return Array.from(merged).sort()
})

// 计算各项目的进行中/阻塞任务数量
const getBlockedCount = (projectName) => {
  // 从 localStorage 读取完整任务数据来计算
  try {
    const tasks = JSON.parse(localStorage.getItem('weekly-report-tasks') || '[]')
    return tasks.filter(t =>
      t.project_name === projectName &&
      (t.status === '阻塞' || t.status === '延期' || t.blockers)
    ).length
  } catch {
    return 0
  }
}

// 确认新增项目
const confirmAddProject = () => {
  const name = newProjectName.value.trim()
  if (!name) {
    showAddInput.value = false
    return
  }
  if (allProjects.value.includes(name)) {
    // 项目已存在，直接切换过去
    emit('update:activeProject', name)
    showAddInput.value = false
    newProjectName.value = ''
    return
  }
  // 添加到手动项目列表
  manualProjects.value.push(name)
  localStorage.setItem('weekly-report-manual-projects', JSON.stringify(manualProjects.value))
  // 通知父组件
  emit('add-project', name)
  // 自动切换到新项目
  emit('update:activeProject', name)
  showAddInput.value = false
  newProjectName.value = ''
}

// 取消新增
const cancelAddProject = () => {
  showAddInput.value = false
  newProjectName.value = ''
}

// 显示输入框时自动聚焦
watch(showAddInput, async (val) => {
  if (val) {
    await nextTick()
    addInputRef.value?.focus()
  }
})
</script>

<style scoped>
/* 隐藏滚动条但保持滚动功能 */
.scrollbar-hide::-webkit-scrollbar {
  display: none;
}

.scrollbar-hide {
  -ms-overflow-style: none;  /* IE and Edge */
  scrollbar-width: none;  /* Firefox */
}
</style>
