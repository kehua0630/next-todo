<script setup lang="ts">
useHead({
  title: "Nuxt Todo",
  meta: [{ name: "description", content: "Nuxt 4 Todo List Demo" }],
});
</script>

<template>
  <!-- Nuxt UI 主題 / 深色模式容器 -->
  <UApp>
    <!-- 頁面內容 -->
    <NuxtPage />
  </UApp>
</template>
