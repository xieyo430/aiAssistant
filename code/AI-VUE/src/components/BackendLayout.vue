<template>
  <div class="backend-layout">
    <el-container>
      <Sidebar />
      <el-container>
        <el-header>
          
        </el-header>
        <el-main>Main</el-main>
      </el-container>
    </el-container>
  </div>
</template>
<script setup>
import Sidebar from './Sidebar.vue'
</script>
<script>
export default {
  components: {
    Sidebar
  }
}
</script>
<style lang='scss' scoped>
.backend-layout {
  height: 100vh;
}
</style>