<template>
  <div class="min-h-screen bg-gray-50 flex flex-col">
    <!-- 顶部标题栏 -->
    <header class="bg-white shadow-sm border-b border-gray-200 flex-shrink-0">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between items-center h-16">
          <h1 class="text-2xl font-bold text-gray-900">周报管理系统</h1>
          <div class="flex items-center space-x-4">
            <!-- 一键复制周报摘要按钮 -->
            <button
              @click="copySummary"
              class="inline-flex items-center px-3 py-2 border border-gray-300 shadow-sm text-sm leading-4 font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
            >
              <svg class="h-4 w-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3" />
              </svg>
              复制周报摘要
            </button>
            <div class="text-sm text-gray-500">
              领导审阅视图
            </div>
          </div>
        </div>
      </div>
    </header>

    <!-- 主内容区：flex-1 撑满剩余视口高度 -->
    <main class="flex-1 max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6 w-full overflow-hidden">
      <!-- 项目 Tab 导航 -->
      <ProjectTabs
        :projects="projects"
        :activeProject="activeProject"
        @update:activeProject="activeProject = $event"
      />

      <div class="mt-6 grid grid-cols-1 lg:grid-cols-4 gap-6" style="height: calc(100vh - 180px);">
        <!-- 左列：红区预警面板（固定高度 + 内部滚动） -->
        <div class="lg:col-span-1 min-h-0">
          <BlockerAlert
            :blockedTasks="blockedTasks"
            ref="blockerAlert"
          />
        </div>

        <!-- 右列：任务矩阵列表（固定高度 + 内部滚动） -->
        <div class="lg:col-span-3 min-h-0">
          <TaskList
            :tasks="filteredTasks"
            :statusFilter="statusFilter"
            @update:statusFilter="statusFilter = $event"
          />
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import ProjectTabs from './ProjectTabs.vue'
import BlockerAlert from './BlockerAlert.vue'
import TaskList from './TaskList.vue'

// 数据状态
const tasks = ref([])
const activeProject = ref('')
const statusFilter = ref('all')
const blockerAlert = ref(null)

// 计算属性
const projects = computed(() => {
  const fromTasks = new Set(tasks.value.map(task => task.project_name))
  // 合并手动添加的项目（独立于任务数据）
  try {
    const manual = JSON.parse(localStorage.getItem('weekly-report-manual-projects') || '[]')
    manual.forEach(p => fromTasks.add(p))
  } catch {}
  return Array.from(fromTasks).sort()
})

const filteredTasks = computed(() => {
  let result = tasks.value

  // 按项目筛选
  if (activeProject.value) {
    result = result.filter(task => task.project_name === activeProject.value)
  }

  // 按状态筛选
  if (statusFilter.value !== 'all') {
    if (statusFilter.value === 'unfinished') {
      result = result.filter(task => task.status !== '已完成')
    } else if (statusFilter.value === 'blocked') {
      result = result.filter(task => task.blockers || task.support_needed)
    }
  }

  return result
})

const blockedTasks = computed(() => {
  return tasks.value.filter(task => task.blockers || task.support_needed)
})

// 复制到剪贴板
const copySummary = async () => {
  try {
    // 构建摘要文本
    let summary = '【周报摘要】\n\n'

    // 添加红区预警内容
    if (blockedTasks.value.length > 0) {
      summary += '⚠️ 红区预警（堵点/需协调事项）:\n'
      blockedTasks.value.forEach(task => {
        summary += `\n• ${task.project_name} - ${task.task_name} (${task.assignee})\n`
        if (task.blockers) {
          summary += `  堵点: ${task.blockers}\n`
        }
        if (task.support_needed) {
          summary += `  需支持: ${task.support_needed}\n`
        }
      })
    } else {
      summary += '✅ 暂无堵点或需协调事项\n'
    }

    summary += `\n📊 当前共 ${tasks.value.length} 个任务，其中 ${tasks.value.filter(t => t.status === '已完成').length} 个已完成，${tasks.value.filter(t => t.status === '进行中').length} 个进行中\n`

    summary += '\n📎 详细周报链接: ' + window.location.origin

    // 使用 Clipboard API 复制
    await navigator.clipboard.writeText(summary)

    alert('✅ 周报摘要已复制到剪贴板！可直接粘贴到微信群')
  } catch (error) {
    console.error('复制失败:', error)
    alert('❌ 复制失败，请手动复制')
  }
}

// 生命周期
onMounted(async () => {
  await loadData()
})

// 加载数据（优先 localStorage，其次静态文件）
const loadData = async () => {
  try {
    // 1. 优先从 localStorage 读取（Admin 编辑后的数据）
    const savedData = localStorage.getItem('weekly-report-tasks')
    if (savedData) {
      tasks.value = JSON.parse(savedData)
      console.log('从 localStorage 加载数据:', tasks.value.length, '条')
    } else {
      // 2. 首次访问：从静态文件加载
      const response = await fetch('/src/assets/data.json')
      tasks.value = await response.json()
      // 保存到 localStorage 供后续使用
      localStorage.setItem('weekly-report-tasks', JSON.stringify(tasks.value))
      console.log('从静态文件加载数据:', tasks.value.length, '条')
    }

    // 默认选择第一个项目
    if (projects.value.length > 0) {
      activeProject.value = projects.value[0]
    }
  } catch (error) {
    console.error('加载数据失败:', error)
  }
}
</script>
