<template>
  <el-aside width="264px">
    <el-menu
        default-active="2"
        class="menu-style"
      >
        <div class="brand">
          <el-image style="width: 50px; height: 50px;margin-right: 10px;"  :src="iconUrl"  alt="机器人" />
          <div class="info-card">
            <h1 class="brand-title">心理健康AI助手</h1>
            <p class="brand-subtitle">管理后台</p>
            
          </div>
        </div>
        <el-menu-item @click="(key) => seleckMenu(item.path)" v-for="item in router.options.routes[0].children" :key="item.path" :index="item.path">
          <el-icon :size="20">
            <component :is="item.meta.icon" />
          </el-icon>
          <span>{{item.meta.title}}</span>
        </el-menu-item>
       
      </el-menu>
  </el-aside>
</template>

<script setup>
import{useRouter}from 'vue-router'
const router = useRouter()

const iconUrl = new URL("@/assets/images/机器人.png", import.meta.url).href

const seleckMenu = (path) => {
  const currentRoute = router.options.routes[0]
  router.push(`${currentRoute.path}/${path}`)
}
console.log(router,'router')
const handleOpen = () => {

}
const handleClose = () => {
  
}

</script>

<style lang="scss" scoped>
.menu-style{
  height: 100%;
  .brand{
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px 0;

  .info-card{
    .brand-title{
      font-size: 20px;
      font-weight: bold;
      margin-bottom: 5px;
      color: #1f2937;
    }
    .brand-subtitle{
      font-size: 14px;
      color:#6b7280
    }
  }
}
}


</style>