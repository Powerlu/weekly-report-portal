<template>
  <div class="bg-white shadow rounded-lg mb-6">
    <div class="px-4 py-5 sm:p-6">
      <h3 class="text-lg leading-6 font-medium text-gray-900 mb-4">
        GitHub 配置
      </h3>

      <div class="grid grid-cols-1 gap-4 sm:grid-cols-2">
        <!-- GitHub Token -->
        <div class="sm:col-span-2">
          <label for="token" class="block text-sm font-medium text-gray-700">
            GitHub Personal Access Token
          </label>
          <div class="mt-1">
            <input
              id="token"
              :type="showToken ? 'text' : 'password'"
              :value="config.token"
              @input="updateConfig('token', $event.target.value)"
              class="shadow-sm focus:ring-blue-500 focus:border-blue-500 block w-full sm:text-sm border-gray-300 rounded-md"
              placeholder="ghp_xxxxxxxxxxxxxxxxxxxx"
            />
          </div>
          <div class="mt-1 flex items-center">
            <input
              id="show-token"
              type="checkbox"
              v-model="showToken"
              class="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
            />
            <label for="show-token" class="ml-2 block text-sm text-gray-700">
              显示 Token
            </label>
          </div>
        </div>

        <!-- 仓库所有者 -->
        <div>
          <label for="owner" class="block text-sm font-medium text-gray-700">
            仓库所有者 (Owner)
          </label>
          <div class="mt-1">
            <input
              id="owner"
              type="text"
              :value="config.owner"
              @input="updateConfig('owner', $event.target.value)"
              class="shadow-sm focus:ring-blue-500 focus:border-blue-500 block w-full sm:text-sm border-gray-300 rounded-md"
              placeholder="your-github-username"
            />
          </div>
        </div>

        <!-- 仓库名称 -->
        <div>
          <label for="repo" class="block text-sm font-medium text-gray-700">
            仓库名称 (Repo Name)
          </label>
          <div class="mt-1">
            <input
              id="repo"
              type="text"
              :value="config.repo"
              @input="updateConfig('repo', $event.target.value)"
              class="shadow-sm focus:ring-blue-500 focus:border-blue-500 block w-full sm:text-sm border-gray-300 rounded-md"
              placeholder="weekly-report-portal"
            />
          </div>
        </div>

        <!-- 文件路径 -->
        <div class="sm:col-span-2">
          <label for="path" class="block text-sm font-medium text-gray-700">
            文件路径 (File Path)
          </label>
          <div class="mt-1">
            <input
              id="path"
              type="text"
              :value="config.path"
              @input="updateConfig('path', $event.target.value)"
              class="shadow-sm focus:ring-blue-500 focus:border-blue-500 block w-full sm:text-sm border-gray-300 rounded-md"
              placeholder="src/assets/data.json"
            />
          </div>
        </div>
      </div>

      <!-- 保存提示 -->
      <div v-if="saveMessage" class="mt-4">
        <p :class="saveMessageClass" class="text-sm">
          {{ saveMessage }}
        </p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  config: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['update:config'])

const showToken = ref(false)
const saveMessage = ref('')
const saveMessageClass = ref('')

// 更新配置并保存到 localStorage
const updateConfig = (key, value) => {
  const newConfig = { ...props.config, [key]: value }
  emit('update:config', newConfig)

  // 保存到 localStorage
  localStorage.setItem('weekly-report-github-config', JSON.stringify(newConfig))

  // 显示保存提示
  saveMessage.value = '配置已自动保存'
  saveMessageClass.value = 'text-green-600'

  setTimeout(() => {
    saveMessage.value = ''
  }, 2000)
}
</script>