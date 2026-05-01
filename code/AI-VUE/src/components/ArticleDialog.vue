<template>
  <el-dialog
    :title="isEdit ? '编辑文章' : '新增文章'"
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
    <el-form-item label="文章摘要" prop="summary">
      <el-input type="textarea" v-model="formData.summary" placeholder="请输入文章摘要" show-word-limit maxlength="1000" :rows="4" />
    </el-form-item>
    <el-form-item label="标签" prop="tags">
      <el-select v-model="formData.tagArray" placeholder="请输入文章标签（多个标签用逗号隔开）" multiple filterable allow-create style="width: 100%" >
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
          <el-button type="danger" size="small" @click="handleRemove">删除</el-button>
        </div>
      </div>
    </el-form-item>
    <el-form-item label="文章内容" prop="content">
      <RichTextEditor v-model="formData.content" placeholder="请输入文章内容" min-height="400px"
      :maxCharCount="5000" @change="handleContentChande" @created="handleEditorCreated" />
    </el-form-item>
  </el-form>
  <div v-if="btnPreview">
    <h3>内容预览</h3>
    <div v-html="formData.content"></div>
  </div>
  <template #footer>
    <el-button type="primary" @click="btnPreview = !btnPreview">{{ btnPreview ? '关闭预览' : '预览效果' }}</el-button>
    <el-button @click="handleClose">取消</el-button>
    <el-button type="primary" :loading="loading" @click="handleSubmit">{{ isEdit ? '更新' : '创建' }}</el-button>
  </template>
  </el-dialog>
</template>

<script setup>
import { ElMessage } from 'element-plus'
import { ref,reactive, computed,nextTick,watch } from 'vue'
import { uploadFile,createArticle } from '@/api/admin'
import { fileBaseUrl } from '@/config/index'
import RichTextEditor from '@/components/RichTextEditor.vue'
import { updateArticle } from '@/api/admin'

const props = defineProps({
modelValue:{
  type:Boolean,
  default:false
},
categories:{
  type:Array,
  default:()=> []
},
article:{
  type:Object,
  default:null
}
})

const emit = defineEmits(['update:modelValue','success'])
const dialogVisible = computed({
  get(){
    return props.modelValue
  },
  set(val){
    emit('update:modelValue',val)
  }
})
const isEdit = computed(() => !!props.article?.id)
const handleClose = () =>{
  formRef.value.resetFields()

  businessId.value = null
  formData.tagArray = []
  handleRemove()
  emit('update:modelValue',false)
}
//监听编辑数据
watch(()=>props.article, (newVal) => {
  if (newVal) {
    nextTick(()=>{
    Object.assign(formData,newVal)
    businessId.value = newVal.id
    imgUrl.value = fileBaseUrl + newVal.coverImage
    })
  }
})


const formData = reactive({
  "title": "",
  "content": "",
  "coverImage": "",
  "categoryId": "",
  "summary": "",
  "tags": "",
  "tagArray": [],
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
const formRef = ref()
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

const businessId = ref(null)
const handleUploadRequest = async({file})=>{
  businessId.value = crypto.randomUUID()
  const fileRes = await uploadFile(file,{
    businessId:businessId.value
  })
  imgUrl.value = fileBaseUrl + fileRes.filePath
  formData.coverImage = fileRes.filePath
}
const handleRemove = () =>{
  imgUrl.value = ''
  formData.coverImage = ''
}
const handleContentChande = (data) =>{
  console.log(data,'富文本内容')
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
const loading = ref(false)
const handleSubmit =() =>{
  // 将tagArray转换为tags字符串
  if(formData.tagArray && formData.tagArray.length > 0){
    formData.tags = formData.tagArray.join(',')
  } else {
    formData.tags = ''
  }

  formRef.value.validate((valid,fields)=>{
    if(valid){
      loading.value = true

      // 创建提交数据
      const submitData = {
        ...formData
      }
      // 删除tagArray属性，因为后端不需要这个字段
      if('tagArray' in submitData){
        delete submitData.tagArray
      }
      if(!isEdit.value){
        submitData.id = businessId.value
        createArticle(submitData).then(res=>{
        loading.value = false
        emit('success')
      })
      }else{
        updateArticle(props.article.id,submitData).then(res=>{
          loading.value = false
          emit('success')
        })
      }

      // 调用创建文章接口
      
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