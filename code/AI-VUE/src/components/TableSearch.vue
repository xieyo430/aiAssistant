<template>
  <el-form :model="formData">
    <template v-for="item in formItem" :key="item.prop">
    <el-form-item :label="item.label" :prop="item.prop">
      <component :is="isComp(item.comp)" :placeholder="item.placeholder" v-model="formData[item.prop]">
        <template v-if="item.comp === 'select'">
          <el-option label="全部" value="" />
          <el-option v-for="opt in item.options" :key="opt.value" :label="opt.Label" :value="opt.value" />
        </template>
      </component>
    </el-form-item>
    </template>
    <el-button type="primary" @click="handleSearch">查询</el-button>
  </el-form>
</template>
<script setup>
import {ref,reactive} from 'vue'

const formData = reactive({})

const props = defineProps({
  formItem: {
    type: Array,
    default: () => []
  }
})
const isComp = (comp)=>{
  return{
    input:'el-input',
    select:'el-select',
    textarea:'el-textarea',
  }[comp]
}
      
</script>