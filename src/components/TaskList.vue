<template>
  <div class="bg-white shadow rounded-lg">
    <div class="px-4 py-5 sm:p-6">
      <!-- 快速筛选按钮 -->
      <div class="flex flex-wrap gap-2 mb-6">
        <button
          @click="$emit('update:statusFilter', 'all')"
          :class="[
            statusFilter === 'all'
              ? 'bg-gray-900 text-white'
              : 'bg-gray-200 text-gray-700 hover:bg-gray-300',
            'px-4 py-2 rounded-md text-sm font-medium'
          ]"
        >
          全部
        </button>
        <button
          @click="$emit('update:statusFilter', 'unfinished')"
          :class="[
            statusFilter === 'unfinished'
              ? 'bg-blue-600 text-white'
              : 'bg-blue-100 text-blue-700 hover:bg-blue-200',
            'px-4 py-2 rounded-md text-sm font-medium'
          ]"
        >
          只看未完成
        </button>
        <button
          @click="$emit('update:statusFilter', 'blocked')"
          :class="[
            statusFilter === 'blocked'
              ? 'bg-red-600 text-white'
              : 'bg-red-100 text-red-700 hover:bg-red-200',
            'px-4 py-2 rounded-md text-sm font-medium'
          ]"
        >
          只看堵点
        </button>
      </div>

      <!-- 任务列表 -->
      <div v-if="tasks.length === 0" class="text-center py-8">
        <p class="text-gray-500">暂无任务数据</p>
      </div>

      <div v-else class="space-y-4">
        <div
          v-for="(task, index) in tasks"
          :key="index"
          :class="[
            'border rounded-lg p-4 transition-shadow hover:shadow-md',
            task.blockers || task.support_needed ? 'border-red-300 bg-red-50' : 'border-gray-200'
          ]"
        >
          <!-- 任务头部 -->
          <div class="flex justify-between items-start mb-3">
            <div class="flex-1 min-w-0">
              <h4 class="text-sm font-medium text-gray-900 break-words">
                {{ task.task_name }}
              </h4>
              <p class="text-xs text-gray-500 mt-1">
                {{ task.project_name }} · {{ task.module }}
              </p>
            </div>
            <span
              :class="[
                'inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium ml-2 flex-shrink-0',
                getStatusClass(task.status)
              ]"
            >
              {{ task.status }}
            </span>
          </div>

          <!-- 任务详情 (移动端优化：垂直堆叠) -->
          <div class="space-y-2 mb-3 text-xs text-gray-600">
            <div class="flex justify-between">
              <span class="font-medium">负责人:</span>
              <span class="ml-2">{{ task.assignee }}</span>
            </div>
            <div class="flex justify-between">
              <span class="font-medium">目标日期:</span>
              <span class="ml-2">{{ task.target_date }}</span>
            </div>
          </div>

          <!-- 工作成果（可折叠，移动端优化：增大触摸区域） -->
          <div v-if="task.achievements" class="mb-3">
            <p class="text-xs font-medium text-gray-700 mb-1">工作成果:</p>
            <div
              :ref="el => setAchievementRef(index, el)"
              class="text-xs text-gray-600 leading-relaxed overflow-hidden break-words"
              :style="{ maxHeight: expandedTasks[index] ? 'none' : '2.5rem', lineHeight: '1.5' }"
            >
              {{ task.achievements }}
            </div>
            <button
              v-if="shouldShowExpandButton(index)"
              @click="toggleExpand(index)"
              class="text-xs text-blue-600 hover:text-blue-800 mt-1 p-1"
            >
              {{ expandedTasks[index] ? '收起' : '展开更多...' }}
            </button>
          </div>

          <!-- 堵点高亮 (移动端优化：增大内边距) -->
          <div v-if="task.blockers" class="mb-2 p-3 bg-red-100 border border-red-300 rounded">
            <p class="text-xs font-medium text-red-800">堵点:</p>
            <p class="text-xs text-red-700 mt-1 break-words">{{ task.blockers }}</p>
          </div>

          <!-- 需支持事项高亮 -->
          <div v-if="task.support_needed" class="p-3 bg-orange-100 border border-orange-300 rounded">
            <p class="text-xs font-medium text-orange-800">需支持:</p>
            <p class="text-xs text-orange-700 mt-1 break-words">{{ task.support_needed }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  tasks: {
    type: Array,
    required: true
  },
  statusFilter: {
    type: String,
    required: true
  }
})

defineEmits(['update:statusFilter'])

const expandedTasks = ref({})
const achievementRefs = ref({})

const setAchievementRef = (index, el) => {
  if (el) {
    achievementRefs.value[index] = el
  }
}

const shouldShowExpandButton = (index) => {
  const el = achievementRefs.value[index]
  if (el) {
    return el.scrollHeight > 40 // 2.5rem = 40px
  }
  return false
}

const toggleExpand = (index) => {
  expandedTasks.value[index] = !expandedTasks.value[index]
}

const getStatusClass = (status) => {
  const statusMap = {
    '已完成': 'bg-green-100 text-green-800',
    '进行中': 'bg-blue-100 text-blue-800',
    '延期': 'bg-yellow-100 text-yellow-800',
    '阻塞': 'bg-red-100 text-red-800'
  }
  return statusMap[status] || 'bg-gray-100 text-gray-800'
}
</script>