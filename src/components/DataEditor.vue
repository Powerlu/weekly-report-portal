<template>
  <div class="bg-white shadow rounded-lg">
    <div class="px-4 py-5 sm:p-6">
      <div class="flex justify-between items-center mb-6">
        <h3 class="text-lg leading-6 font-medium text-gray-900">
          数据编辑面板
        </h3>
        <div class="flex space-x-3">
          <button
            @click="showAddModal = true"
            class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
          >
            新增任务
          </button>
          <button
            @click="saveToGitHub"
            :disabled="isSaving || !isGitHubConfigured"
            :class="[
              isSaving ? 'opacity-50 cursor-not-allowed' : '',
              'inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-green-600 hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500'
            ]"
          >
            <svg v-if="isSaving" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            {{ isSaving ? '保存中...' : '保存并同步至 GitHub' }}
          </button>
        </div>
      </div>

      <!-- GitHub 配置检查 -->
      <div v-if="!isGitHubConfigured" class="mb-4 p-4 bg-yellow-50 border border-yellow-200 rounded-md">
        <p class="text-sm text-yellow-800">
          请先在上方填写完整的 GitHub 配置信息（Token、Owner、Repo）
        </p>
      </div>

      <!-- 任务列表 -->
      <div v-if="tasks.length === 0" class="text-center py-8">
        <p class="text-gray-500">暂无任务数据，点击"新增任务"开始添加</p>
      </div>

      <div v-else class="space-y-4">
        <div
          v-for="(task, index) in tasks"
          :key="index"
          class="border border-gray-200 rounded-lg p-4 hover:shadow-md transition-shadow"
        >
          <div class="flex justify-between items-start">
            <div class="flex-1">
              <h4 class="text-sm font-medium text-gray-900">
                {{ task.task_name }}
              </h4>
              <p class="text-xs text-gray-500 mt-1">
                {{ task.project_name }} · {{ task.module }} · {{ task.assignee }}
              </p>
            </div>
            <div class="flex space-x-2">
              <button
                @click="editTask(index)"
                class="text-sm text-blue-600 hover:text-blue-800"
              >
                编辑
              </button>
              <button
                @click="deleteTask(index)"
                class="text-sm text-red-600 hover:text-red-800"
              >
                删除
              </button>
            </div>
          </div>
          <div class="mt-2">
            <span
              :class="[
                'inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium',
                getStatusClass(task.status)
              ]"
            >
              {{ task.status }}
            </span>
            <span class="ml-2 text-xs text-gray-500">
              目标日期: {{ task.target_date }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- 新增/编辑任务模态框 -->
  <div v-if="showAddModal || showEditModal" class="fixed inset-0 bg-gray-500 bg-opacity-75 flex items-center justify-center z-50">
    <div class="bg-white rounded-lg px-4 pt-5 pb-4 shadow-xl sm:my-8 sm:align-middle sm:max-w-2xl sm:w-full sm:p-6">
      <div class="sm:flex sm:items-start">
        <div class="mt-3 sm:mt-0 sm:text-left w-full">
          <h3 class="text-lg leading-6 font-medium text-gray-900 mb-4">
            {{ showEditModal ? '编辑任务' : '新增任务' }}
          </h3>

          <div class="grid grid-cols-1 gap-4 sm:grid-cols-2">
            <!-- 项目名称（可输入 + 下拉选择 + 自动记忆） -->
            <div class="sm:col-span-2">
              <label class="block text-sm font-medium text-gray-700">项目名称 <span class="text-gray-400 text-xs">（可直接输入新项目名，回车或失焦后自动记住）</span></label>
              <div class="relative mt-1">
                <input
                  v-model="formData.project_name"
                  type="text"
                  list="project-list"
                  @blur="onProjectBlur"
                  @keydown.enter.prevent="onProjectBlur"
                  class="mt-0 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                  placeholder="输入或选择项目名称"
                />
                <datalist id="project-list">
                  <option v-for="p in projectList" :key="p" :value="p" />
                </datalist>
              </div>
            </div>

            <!-- 任务名称 -->
            <div class="sm:col-span-2">
              <label class="block text-sm font-medium text-gray-700">任务名称</label>
              <input
                v-model="formData.task_name"
                type="text"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                placeholder="请输入任务名称"
              />
            </div>

            <!-- 功能模块 -->
            <div>
              <label class="block text-sm font-medium text-gray-700">功能/模块</label>
              <input
                v-model="formData.module"
                type="text"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                placeholder="如：前端、后端"
              />
            </div>

            <!-- 目标日期 -->
            <div>
              <label class="block text-sm font-medium text-gray-700">目标日期</label>
              <input
                v-model="formData.target_date"
                type="date"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
              />
            </div>

            <!-- 完成人 -->
            <div>
              <label class="block text-sm font-medium text-gray-700">完成人</label>
              <input
                v-model="formData.assignee"
                type="text"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                placeholder="如：路俊博、王荣欣"
              />
            </div>

            <!-- 状态 -->
            <div>
              <label class="block text-sm font-medium text-gray-700">状态</label>
              <select
                v-model="formData.status"
                class="mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm rounded-md"
              >
                <option value="已完成">已完成</option>
                <option value="进行中">进行中</option>
                <option value="延期">延期</option>
                <option value="阻塞">阻塞</option>
              </select>
            </div>

            <!-- 工作成果 -->
            <div class="sm:col-span-2">
              <label class="block text-sm font-medium text-gray-700">本周工作成果</label>
              <textarea
                v-model="formData.achievements"
                rows="3"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                placeholder="请输入本周完成的工作内容"
              ></textarea>
            </div>

            <!-- 堵点 -->
            <div class="sm:col-span-2">
              <label class="block text-sm font-medium text-gray-700">存在问题或堵点</label>
              <textarea
                v-model="formData.blockers"
                rows="2"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                placeholder="如有问题或堵点，请说明"
              ></textarea>
            </div>

            <!-- 需支持事项 -->
            <div class="sm:col-span-2">
              <label class="block text-sm font-medium text-gray-700">需支持或协调事项</label>
              <textarea
                v-model="formData.support_needed"
                rows="2"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                placeholder="如需支持或协调，请说明"
              ></textarea>
            </div>
          </div>
        </div>
      </div>

      <div class="mt-5 sm:mt-4 sm:flex sm:flex-row-reverse">
        <button
          @click="saveTask"
          class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-blue-600 text-base font-medium text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 sm:ml-3 sm:w-auto sm:text-sm"
        >
          保存
        </button>
        <button
          @click="closeModal"
          class="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 sm:mt-0 sm:w-auto sm:text-sm"
        >
          取消
        </button>
      </div>
    </div>
  </div>

  <!-- Toast 提示 -->
  <div
    v-if="toast.show"
    :class="[
      toast.type === 'success' ? 'bg-green-500' : 'bg-red-500',
      'fixed bottom-4 right-4 px-4 py-2 rounded-md text-white shadow-lg z-50'
    ]"
  >
    {{ toast.message }}
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  tasks: {
    type: Array,
    required: true
  },
  githubConfig: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['update:tasks'])

// 状态
const showAddModal = ref(false)
const showEditModal = ref(false)
const editingIndex = ref(-1)
const isSaving = ref(false)
const formData = ref({
  project_name: '',
  task_name: '',
  module: '',
  target_date: '',
  assignee: '',
  status: '进行中',
  achievements: '',
  blockers: '',
  support_needed: ''
})

// Toast 提示
const toast = ref({
  show: false,
  message: '',
  type: 'success'
})

// 项目列表（从现有任务中提取 + localStorage 记忆）
const getInitialProjects = () => {
  const saved = localStorage.getItem('weekly-report-projects')
  if (saved) return JSON.parse(saved)
  // 从当前任务中提取去重
  const fromTasks = [...new Set(props.tasks.map(t => t.project_name).filter(Boolean))]
  return fromTasks.length > 0 ? fromTasks : ['项目管理系统', '面试微信小程序', '智能简历解析']
}
const projectList = ref(getInitialProjects())

// 保存项目列表到 localStorage
const saveProjectList = () => {
  const allProjects = [...new Set([...projectList.value, ...props.tasks.map(t => t.project_name).filter(Boolean)])]
  projectList.value = allProjects
  localStorage.setItem('weekly-report-projects', JSON.stringify(allProjects))
}

// 项目输入框失焦或回车时，自动记忆新项目
const onProjectBlur = () => {
  const name = formData.value.project_name?.trim()
  if (name && !projectList.value.includes(name)) {
    projectList.value.push(name)
    localStorage.setItem('weekly-report-projects', JSON.stringify(projectList.value))
  }
}

// 计算属性：检查 GitHub 配置是否完整
const isGitHubConfigured = computed(() => {
  return props.githubConfig.token &&
         props.githubConfig.owner &&
         props.githubConfig.repo &&
         props.githubConfig.path
})

// 显示 Toast
const showToast = (message, type = 'success') => {
  toast.value = {
    show: true,
    message,
    type
  }
  setTimeout(() => {
    toast.value.show = false
  }, 3000)
}

// 编辑任务
const editTask = (index) => {
  editingIndex.value = index
  formData.value = { ...props.tasks[index] }
  showEditModal.value = true
}

// 删除任务
const deleteTask = (index) => {
  if (confirm('确定要删除这个任务吗？')) {
    const newTasks = [...props.tasks]
    newTasks.splice(index, 1)
    emit('update:tasks', newTasks)
    showToast('任务已删除')
  }
}

// 保存任务（新增或编辑）
const saveTask = () => {
  if (!formData.value.project_name || !formData.value.task_name) {
    showToast('请填写项目名称和任务名称', 'error')
    return
  }

  const newTasks = [...props.tasks]

  if (showEditModal.value) {
    // 编辑模式
    newTasks[editingIndex.value] = { ...formData.value }
    showToast('任务已更新')
  } else {
    // 新增模式
    newTasks.push({ ...formData.value })
    showToast('✅ 任务已添加（别忘了点「保存并同步至 GitHub」持久化）')
  }

  emit('update:tasks', newTasks)
  saveProjectList()
  closeModal()
}

// 关闭模态框
const closeModal = () => {
  showAddModal.value = false
  showEditModal.value = false
  editingIndex.value = -1
  formData.value = {
    project_name: '',
    task_name: '',
    module: '',
    target_date: '',
    assignee: '',
    status: '进行中',
    achievements: '',
    blockers: '',
    support_needed: ''
  }
}

// 保存到 GitHub
const saveToGitHub = async () => {
  if (!isGitHubConfigured.value) {
    showToast('请先填写完整的 GitHub 配置', 'error')
    return
  }

  isSaving.value = true

  try {
    const { token, owner, repo, path } = props.githubConfig

    // 1. GET 请求获取文件的 sha
    const getUrl = `https://api.github.com/repos/${owner}/${repo}/contents/${path}`
    const getResponse = await fetch(getUrl, {
      headers: {
        'Authorization': `token ${token}`,
        'Accept': 'application/vnd.github.v3+json'
      }
    })

    if (!getResponse.ok) {
      throw new Error(`获取文件失败: ${getResponse.statusText}`)
    }

    const fileData = await getResponse.json()
    const sha = fileData.sha

    // 2. 将数据转换为 Base64
    const content = JSON.stringify(props.tasks, null, 2)
    const base64Content = btoa(unescape(encodeURIComponent(content)))

    // 3. PUT 请求提交文件更新
    const putUrl = `https://api.github.com/repos/${owner}/${repo}/contents/${path}`
    const putResponse = await fetch(putUrl, {
      method: 'PUT',
      headers: {
        'Authorization': `token ${token}`,
        'Accept': 'application/vnd.github.v3+json',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        message: 'Update weekly report data via admin',
        content: base64Content,
        sha: sha
      })
    })

    if (!putResponse.ok) {
      const errorData = await putResponse.json()
      throw new Error(`保存失败: ${errorData.message}`)
    }

    showToast('✅ 数据已提交至 GitHub！Vercel 预计将在 1-2 分钟内自动完成全网更新', 'success')
  } catch (error) {
    console.error('GitHub API 错误:', error)
    showToast(`❌ ${error.message}`, 'error')
  } finally {
    isSaving.value = false
  }
}

// 状态样式
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