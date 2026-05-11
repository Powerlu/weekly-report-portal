<script setup>
import './style.css'
</script>

<template>
  <div>
    <!-- 顶部导航栏 -->
    <nav class="bg-white shadow-sm border-b border-gray-200">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between items-center h-16">
          <div class="flex items-center space-x-8">
            <router-link to="/" class="text-xl font-bold text-gray-900">
              周报管理系统
            </router-link>
            <div class="hidden sm:flex sm:space-x-4">
              <router-link
                to="/"
                class="px-3 py-2 rounded-md text-sm font-medium text-gray-700 hover:text-gray-900 hover:bg-gray-100"
                active-class="bg-gray-100 text-gray-900"
              >
                审阅视图
              </router-link>
              <router-link
                to="/admin"
                class="px-3 py-2 rounded-md text-sm font-medium text-gray-700 hover:text-gray-900 hover:bg-gray-100"
                active-class="bg-gray-100 text-gray-900"
              >
                管理后台
              </router-link>
            </div>
          </div>
        </div>
      </div>
    </nav>

    <!-- 路由视图 -->
    <router-view />
  </div>
</template>
