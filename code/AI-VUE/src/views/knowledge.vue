<template>
  <div>
    <PageHead title="知识文章">
    <template #buttons>
      <el-button @click="handleEdit({})" type="primary">新增</el-button>
    </template>
    </PageHead>
    <TableSearch :formItem="formItem" @search="handleSearch" />
    <el-table :data="tableData" style="width:100% ;margin-top:25px;">
      <el-table-column label="文章标题" width="500" fixed="left">
          <template #default="scope">
            <div style="display: flex;align-items: center">
              <el-icon><timer /></el-icon>
              <span>{{scope.row.title}}</span>
            </div>
          </template>
      </el-table-column>
      <el-table-column label="分类" width="200">
          <template #default="scope">
            <div style="display: flex;align-items: center">
              <el-icon><timer /></el-icon>
              <span>{{categoryMap[scope.row.categoryId]}}</span>
            </div>
          </template>
      </el-table-column>
      <el-table-column prop="authorName" width="150" label="作者" />
      <el-table-column prop="readCount" width="150" label="阅读量" />
      <el-table-column prop="updatedAt" width="150" label="发布时间" />
      <el-table-column label="操作" width="240" fixed="right">
          <template #default="scope">
            <el-button @click="handleEdit(scope.row)" text type="primary" >编辑</el-button>
            <el-button v-if="scope.row.status===0 || scope.row.status===2" text type="success">发布</el-button>
            <el-button v-if="scope.row.status===1" text type="warning">下线</el-button>
            <el-button text type="danger">删除</el-button>
          </template>
      </el-table-column>
    </el-table>
    <el-pagination
      style="margin-top:25px"
      :page-size="pagination.size"
      layout="prev,pager,next"
      :total="pagination.total"
      @change="handleChange" />
      <ArticleDialog :article="currentArticle" v-model:modelValue="dialogVisible" :categories="categories" @success="handleSuccess"/>
  </div>
</template>
<script setup> 
import { onMounted,reactive,ref } from 'vue'
import PageHead from '@/components/PageHead.vue'
import TableSearch from '@/components/TableSearch.vue'
import { categoryTree,articlePage,getArticleDetail } from '@/api/admin'
import ArticleDialog from '@/components/ArticleDialog.vue'

const formItem = [
  {comp:'input',prop:'title',label:'文章标题',placeholder:'请输入文章标题'},
  {comp:'select',prop:'categoryId',label:'分类',placeholder:'请选择分类',options:[]},
  {comp:'select',prop:'status',label:'状态',placeholder:'请选择状态',options:[
    {label:'草稿',
      value:0
    },
    {label:'已发布',
      value:1
    },
    {label:'已下线',
      value:2
    },
  ]},
]

//分页参数
const pagination = reactive({
  currentPage: 1,
  size: 10,
  total: 0,
})

const handleSearch = async (formData) => {
  console.log(formData,'formData')
  const params={
    ...formData,
    ...pagination,
  }
  const {records,total} = await articlePage(params)
  tableData.value = records
  pagination.total = total
  
}
const handleChange = (page) => {
  pagination.currentPage = page
  handleSearch({})
}

const categoryMap = reactive({})

const categories = ref([])
  
//列表数据
const tableData = ref([])

const dialogVisible = ref(false)
const currentArticle = ref(null)
const handleSuccess = () =>{}
const handleEdit = (row) =>{
 if(!row.id){
   //新增
   currentArticle.value = null  
   dialogVisible.value = true
 }else{

 //编辑
 getArticleDetail(row.id).then(res=>{
  currentArticle.value = res
  dialogVisible.value = true
 })
 }



}
onMounted(async() => {
  const data = await categoryTree()
  categories.value = data.map(item =>{
    categoryMap[item.id] = item.categoryName
    return{
      label:item.categoryName,
      value:item.id,
    }
  })
  
    formItem[1].options = categories.value

    handleSearch({})
})
</script>