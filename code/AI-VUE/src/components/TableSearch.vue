<template>
  <el-form ref="ruleFormRef" :model="formData">
<el-row :gutter="24">
 <template v-for="item in formItemAttr" :key="item.prop">
  <el-col v-bind="item.col">
<el-form-item :label="item.label" :prop="item.prop">
      <component :is="isComp(item.comp)" :placeholder="item.placeholder" v-model="formData[item.prop]">
        <template v-if="item.comp === 'select'">
          <el-option label="全部" value="" />
          <el-option v-for="opt in item.options" :key="opt.value" :label="opt.label" :value="opt.value" />
        </template>
      </component>
    </el-form-item>
  
  </el-col>
    </template>
    
</el-row>
   <el-row>
    <el-form-item>
      <el-button type="primary" @click="handleSearch">查询</el-button>
      <el-button  @click="handleReset(ruleFormRef)">重置</el-button>
    </el-form-item>
   </el-row>
  </el-form>
</template>
<script setup>
import {ref,reactive,computed} from 'vue'

const props = defineProps({
  formItem: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['search','reset'])
const formItemAttr = computed(()=>{
  const {formItem} = props
  formItem.forEach(item => {
    item.col = {xs:24 , sm:12,md:8,lg:6,xl:6,}
  })
  return formItem
})
const ruleFormRef = ref()
const formData = reactive({})
const isComp = (comp)=>{
  return{
    input:'el-input',
    select:'el-select',
    textarea:'el-textarea',
  }[comp]
}
      

      const handleSearch = () => {
        emit('search', formData)
      }
      const handleReset = (formEl) => {
        if(!formEl) return
          formEl.resetFields()
          emit('search', formData)
      }
      
</script>