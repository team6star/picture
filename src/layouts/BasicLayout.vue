<template>
  <div id="basicLayout">
    <a-layout style="min-height: 100vh">
      <!-- 全局顶部栏 -->
      <a-layout-header class="header">
        <GlobalHeader />
      </a-layout-header>
      <a-layout-content class="content">
        <!-- 动态替换内容 -->
        <router-view />
      </a-layout-content>
      <!-- 全局底部栏 -->
      <a-layout-footer class="footer">
        <a href="https://www.codefather.cn" target="_blank"> 云图库 by coderstar </a>
      </a-layout-footer>
    </a-layout>
  </div>
</template>

<script setup lang="ts">
import GlobalHeader from '@/components/GlobalHeader.vue'
</script>

<style scoped>
#basicLayout .footer {
  background: #efefef;
  padding: 16px;
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  text-align: center;
}

#basicLayout .content {
  background: linear-gradient(to right, #fefefe, #fff);
  margin-bottom: 28px;
  padding: 20px;
}
#basicLayout .header {
  padding-inline: 20px;
  margin-bottom: 16px;
  color: unset;
  background: white;
}
</style>
