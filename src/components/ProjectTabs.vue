<template>
  <div class="border-b border-gray-200">
    <nav class="-mb-px flex space-x-4 overflow-x-auto pb-2 scrollbar-hide">
      <button
        v-for="project in projects"
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
        <!-- 显示进行中/阻塞任务数量角标 -->
        <span
          v-if="getBlockedCount(project) > 0"
          class="ml-2 inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-red-100 text-red-800"
        >
          {{ getBlockedCount(project) }}
        </span>
      </button>
    </nav>
  </div>
</template>

<script setup>
import { computed } from 'vue'

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

defineEmits(['update:activeProject'])

// 计算各项目的进行中/阻塞任务数量
const getBlockedCount = (projectName) => {
  // 这里需要从父组件获取数据，暂时返回 0
  // 实际实现中应该通过 provide/inject 或事件总线来共享数据
  return 0
}
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