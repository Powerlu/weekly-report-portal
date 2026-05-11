<template>
  <div class="min-h-screen bg-gray-50">
    <!-- 密码验证弹窗 -->
    <div v-if="!isAuthenticated" class="fixed inset-0 bg-gray-500 bg-opacity-75 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg px-4 pt-5 pb-4 shadow-xl sm:my-8 sm:align-middle sm:max-w-lg sm:w-full sm:p-6">
        <div class="sm:flex sm:items-start">
          <div class="mt-3 text-center sm:mt-0 sm:text-left w-full">
            <h3 class="text-lg leading-6 font-medium text-gray-900 mb-4">
              访问验证
            </h3>
            <div>
              <label for="password" class="block text-sm font-medium text-gray-700">
                请输入访问密码
              </label>
              <input
                id="password"
                v-model="password"
                type="password"
                @keyup.enter="checkPassword"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                placeholder="请输入密码"
              />
              <p v-if="passwordError" class="mt-2 text-sm text-red-600">
                密码错误，请重试
              </p>
            </div>
          </div>
        </div>
        <div class="mt-5 sm:mt-4 sm:flex sm:flex-row-reverse">
          <button
            @click="checkPassword"
            class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-blue-600 text-base font-medium text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 sm:ml-3 sm:w-auto sm:text-sm"
          >
            确认
          </button>
        </div>
      </div>
    </div>

    <!-- 主内容区 (验证通过后显示) -->
    <div v-if="isAuthenticated">
      <!-- 顶部标题栏 -->
      <header class="bg-white shadow-sm border-b border-gray-200">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div class="flex justify-between items-center h-16">
            <h1 class="text-2xl font-bold text-gray-900">管理后台</h1>
            <button
              @click="logout"
              class="px-4 py-2 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
            >
              退出登录
            </button>
          </div>
        </div>
      </header>

      <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6">
        <!-- GitHub 配置面板 -->
        <GitHubConfig
          :config="githubConfig"
          @update:config="githubConfig = $event"
        />

        <!-- 数据编辑面板 -->
        <DataEditor
          :tasks="tasks"
          @update:tasks="tasks = $event"
          :githubConfig="githubConfig"
        />
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import GitHubConfig from './GitHubConfig.vue'
import DataEditor from './DataEditor.vue'

// 密码验证相关
const isAuthenticated = ref(false)
const password = ref('')
const passwordError = ref(false)
const CORRECT_PASSWORD = 'admin123'

// GitHub 配置
const githubConfig = ref({
  token: '',
  owner: '',
  repo: '',
  path: 'src/assets/data.json'
})

// 任务数据
const tasks = ref([])

// 监听任务数据变化，自动同步到 localStorage
watch(tasks, (newTasks) => {
  if (newTasks && newTasks.length > 0) {
    localStorage.setItem('weekly-report-tasks', JSON.stringify(newTasks))
    console.log('任务数据已同步到 localStorage:', newTasks.length, '条')
  }
}, { deep: true })

// 检查是否已认证
onMounted(() => {
  const savedAuth = localStorage.getItem('weekly-report-auth')
  if (savedAuth === 'true') {
    isAuthenticated.value = true
  }

  // 加载 GitHub 配置
  const savedConfig = localStorage.getItem('weekly-report-github-config')
  if (savedConfig) {
    githubConfig.value = JSON.parse(savedConfig)
  }

  // 加载任务数据
  loadTasks()
})

// 加载任务数据
const loadTasks = async () => {
  try {
    // 优先从 localStorage 读取
    const savedTasks = localStorage.getItem('weekly-report-tasks')
    if (savedTasks) {
      tasks.value = JSON.parse(savedTasks)
    } else {
      // 首次访问：从静态文件加载
      const response = await fetch('/src/assets/data.json')
      tasks.value = await response.json()
      localStorage.setItem('weekly-report-tasks', JSON.stringify(tasks.value))
    }
  } catch (error) {
    console.error('加载数据失败:', error)
  }
}

// 验证密码
const checkPassword = () => {
  if (password.value === CORRECT_PASSWORD) {
    isAuthenticated.value = true
    localStorage.setItem('weekly-report-auth', 'true')
    passwordError.value = false
    password.value = ''
  } else {
    passwordError.value = true
  }
}

// 退出登录
const logout = () => {
  isAuthenticated.value = false
  localStorage.removeItem('weekly-report-auth')
}
</script>