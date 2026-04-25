<template>
  <div>
    <PageHead title="知识文章">
    <template #buttons>
      <el-button type="primary">新增</el-button>
    </template>
    </PageHead>
    <TableSearch :formItem="formItem" @search="handleSearch" />
  </div>
</template>
<script setup> 
import PageHead from '@/components/PageHead.vue'
import TableSearch from '@/components/TableSearch.vue'

const formItem = [
  {comp:'input',prop:'title',label:'文章标题',placeholder:'请输入文章标题'},
  {comp:'select',prop:'categoryId',label:'分类',placeholder:'请选择分类',options:[
    {
    Label:'心理健康基础',
    value:1,
  },
  {
    Label:'情绪管理',
    value:2,
  }] },
  {comp:'input',prop:'content',label:'文章内容',placeholder:'请输入文章内容'},
]

const handleSearch = (formData) => {
  console.log(formData,'formData')
}
</script>