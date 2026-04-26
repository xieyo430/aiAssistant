<template>
  <el-dialog
    title="文章详情"
    v-model="dialogVisible"
    width="50%"
    @close="handleClose"
  >
  <el-form :model="formData" :rules="rules" ref="formRef" label-width="120px">
    <el-form-item label="文章标题" prop="title">
      <el-input v-model="formData.title" placeholder="请输入文章标题" show-word-limit clearable  maxlength="200" />
    </el-form-item>
    <el-form-item label="所属分类" prop="categoryId">
      <el-select v-model="formData.categoryId" placeholder="请选择分类">
        <el-option v-for="item in props.categories" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </el-form-item>
    <el-form-item label="文章内容" prop="summary">
      <el-input type="textarea" v-model="formData.summary" placeholder="请输入文章内容" show-word-limit maxlength="1000" :rows="4" />
    </el-form-item>
    <el-form-item label="文章标签" prop="tags">
      <el-select v-model="formData.tagArray" placeholder="请输入文章标签" multiple filterable allow-create style="width: 100%" >
        <el-option v-for="tag in commonTags" :key="tag" :label="tag" :value="tag" />
      </el-select>
    </el-form-item>
    <el-form-item label="封面图片">
      <div class="cover-upload">
        <el-upload
          class="avatar-uploader"
          action="#"
          :before-upload="beforeUpload"
          :show-file-list="false"
          :http-request="handleUploadRequest"
          accept="image/*"
        >
          <div v-if="!imgUrl" class="cover-placehoder">
            <p>点击上传封面</p>
          </div>      
            <img  v-else class="cover-image" :src="imgUrl" alt="封面图片" />
        </el-upload>
        <div v-if="imgUrl" class="cover-remove">
          <el-button type="danger" size="mini" @click="handleRemove">删除</el-button>
        </div>
      </div>
    </el-form-item>
    <el-form-item label="文章内容" prop="content">
      <RichTextEditor v-model="formData.content" placeholder="请输入文章内容" min-height="400px"
      :maxCharCount="5000" @change="handleContentChande" @created="handleEditorCreated" />
    </el-form-item>
  </el-form>
  <div v-if="btnPreview">
    <h3>预览效果</h3>
    <div v-html="formData.content"></div>
  </div>
  <template #footer>
    <el-button type="primary" @click="btnPreview = !btnPreview">{{ btnPreview ? '关闭预览' : '预览效果' }}</el-button>
    <el-button @click="handleClose">取消</el-button>
    <el-button :loading="loading" @click="handleSubmit(formRef)">创建</el-button>
  </template>
  </el-dialog>
</template>

<script setup>
import { ElMessage } from 'element-plus'
import { ref,reactive, computed,nextTick } from 'vue'
import { uploadFile } from '@/api/admin'
import { fileBaseUrl } from '@/config/index'
import RichTextEditor from '@/components/RichTextEditor.vue'
import { createArticle } from '@/api/admin'

const props = defineProps({
modelValue:{
  type:Boolean,
  default:false
},
categories:{
  type:Array,
  default:()=> []
}
})

const emit = defineEmits(['update:modelValue'])
const dialogVisible = computed({
  get(){
    return props.modelValue
  },
  set(val){
    emit('update:modelValue',val)
  }
})
const handleClose = () =>{

}
const formData = reactive({
  "title": "",
    "content": "",
    "coverImage": "",
    "categoryId": "",
    "summary": "",
    "tags": "",
    "id": ""
})

const rules = reactive({
  title: [
    { required: true, message: '请输入文章标题', trigger: 'blur' },
    { max: 200, message: '文章标题最多200个字符', trigger: 'blur' }
  ],
  categoryId: [
    { required: true, message: '请选择分类', trigger: 'change' }
  ],
  summary: [
    { required: true, message: '请输入文章内容', trigger: 'blur' },
    { max: 1000, message: '文章内容最多1000个字符', trigger: 'blur' }
  ],
  tags: [
    { required: true, message: '请输入标签', trigger: 'blur' }
  ],
  id: [
    { required: true, message: '请输入文章ID', trigger: 'blur' }
  ],
  content: [
    { required: true, message: '请输入文章内容', trigger: 'blur' },
    { max: 5000, message: '文章内容最多5000个字符', trigger: 'blur' }
  ]
})
const commonTags = [
  '情绪管理', '焦虑', '抑郁', '压力', '睡眠', 
  '冥想', '正念', '放松', '心理健康', '自我成长',
  '人际关系', '工作压力', '学习方法', '生活技巧'
]
const FormRef = ref()
//上传
const imgUrl = ref('')
const beforeUpload = (file)=>{
  const isImage = file.type.startsWith('image/')
  const isLt5M = file.size / 1024 / 1024 < 5
  if(!isImage){
    ElMessage.error('请上传图片')
    return false
  }
  return isImage
}

const handleUploadRequest = async({file})=>{
  const businessId = crypto.randomUUID()
  const fileRes = await uploadFile(file,{
    businessId:businessId
  })
  imgUrl.value = fileBaseUrl + fileRes.filePath
  formData.coverImage = fileRes.filePath
}
const handleRemove = () =>{
  imgUrl.value = ''
  formData.coverImage = ''
}
const handleContentChande = (data) =>{
  formData.content = data.html
}
const editorInstance = ref(null)
const handleEditorCreated = (editor) =>{
  editorInstance.value = editor
  if(formData.content && editor){
    nextTick(()=>{
      editor.setHtml(formData.content)
    })
  }
}
const btnPreview = ref(false)
 const formRef = ref()
const loading = ref(false)
const handleSubmit =(formRef) =>{
  formRef.value.validate((valid,fields)=>{
    if(valid){
      loading.value = true
    }
  })
}
</script>

<style lang="scss" scoped>
.cover-placehoder{
  display: flex;
  width: 200px;
  height: 120px;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  color: #8b949e;
  background: #f6f8fa

}
.cover-image{
  width: 200px;
  height: 120px;
  display: block;
}
</style>