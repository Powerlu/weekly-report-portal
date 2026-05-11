<template>
  <div class="bg-white shadow rounded-lg border-l-4 border-red-500">
    <div class="px-4 py-5 sm:p-6">
      <h3 class="text-lg leading-6 font-medium text-red-800 mb-4">
        🚨 红区预警
      </h3>

      <div v-if="blockedTasks.length === 0" class="text-center py-4">
        <p class="text-sm text-gray-500">暂无堵点或需协调事项</p>
      </div>

      <div v-else class="space-y-4 max-h-[420px] overflow-y-auto pr-2 custom-scrollbar">
        <div
          v-for="(task, index) in blockedTasks"
          :key="index"
          class="border border-red-200 rounded-md p-3 bg-red-50"
        >
          <div class="flex justify-between items-start">
            <div class="flex-1">
              <h4 class="text-sm font-medium text-red-900">
                {{ task.project_name }} - {{ task.task_name }}
              </h4>
              <p class="text-xs text-red-700 mt-1">
                <span class="font-medium">负责人:</span> {{ task.assignee }}
              </p>
            </div>
            <span
              :class="[
                'inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium',
                getStatusClass(task.status)
              ]"
            >
              {{ task.status }}
            </span>
          </div>

          <div v-if="task.blockers" class="mt-2">
            <p class="text-xs font-medium text-red-800">堵点:</p>
            <p class="text-xs text-red-700 mt-1">{{ task.blockers }}</p>
          </div>

          <div v-if="task.support_needed" class="mt-2">
            <p class="text-xs font-medium text-orange-800">需支持:</p>
            <p class="text-xs text-orange-700 mt-1">{{ task.support_needed }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  blockedTasks: {
    type: Array,
    required: true
  }
})

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

<style scoped>
.custom-scrollbar::-webkit-scrollbar {
  width: 5px;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #fca5a5;
  border-radius: 3px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: #fef2f2;
}
</style>
