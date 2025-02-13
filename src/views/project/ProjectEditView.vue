<script lang="ts">
import { reactive, ref } from 'vue'
import { EditPen } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'
import BreadCrumbNav from '@/components/BreadCrumbNav.vue'
import user from '@/api/user.ts'
import project from '@/api/project.ts'

const loading = ref(true)
const moduleName = 'project'
const formData = reactive({
  id: '',
  name: '',
  keyword: '',
  description: '',
  owner: ''
})
const formDataRef = ref()
const users = reactive([])

export default {
  components: {EditPen, BreadCrumbNav},
  setup() {
    return {
      loading,
      users,
      formData,
      formDataRef,
      formRules: reactive({
        name: [
          {
            required: true,
            message: '请输入项目名称',
            trigger: 'blur'
          },
          { max: 50, message: '项目名称不能超过50个字', trigger: 'blur' },
        ],
        description: [
          { max: 200, message: '项目描述不能超过200个字', trigger: 'blur' },
        ],
        keyword: [
          {
            required: true,
            message: '请输入项目关键字',
            trigger: 'blur'
          },
          { max: 20, message: '项目关键字不能超过20个字', trigger: 'blur' },
        ],
        owner: [
          {
            required: true,
            message: '请选择项目负责人',
            trigger: 'blur'
          }
        ]
      }),
    }
  },
  mounted() {
    loading.value = true
    formData.id = this.$route.query.id ? this.$route.query.id.toString() : ''
    user.all().then(data => {
      users.length = 0
      Object.assign(users, data)
    })
    project.get(formData.id).then(projectInfo => {
      if (projectInfo) {
        formData.name = projectInfo.name
        formData.keyword = projectInfo.keyword
        formData.description = projectInfo.description
        formData.owner = projectInfo.owner.id
      }
      loading.value = false
    })
  },
  methods: {
    handleSubmit() {
      try {
        formDataRef.value.validate().then(() => {
          project.modify(formData).then(response => {
            if (response.success) {
              ElMessage.success('修改成功')
              formDataRef.value.resetFields()
              this.$router.push('/' + moduleName + '/list')
            } else {
              ElMessage.error('修改失败')
            }
          })
        })
      } catch (error) {
        ElMessage.error('请检查输入内容')
      }
    },
    handleReturn() {
      this.$router.go(-1)
    }
  }
}
</script>

<template>
  <BreadCrumbNav :page-paths="['项目管理', '项目列表', '项目修改']"></BreadCrumbNav>
  <el-card class="info-card" shadow="never" v-loading="loading">
    <template #header>
      <div class="card-header">
        <el-icon>
          <EditPen/>
        </el-icon>&nbsp;&nbsp;项目修改
      </div>
    </template>
    <el-form
        ref="formDataRef"
        :model="formData"
        :rules="formRules"
        label-width="30%"
        style="width: 60%"
        @keyup.enter="handleSubmit"
    >
      <el-form-item label="项目名称" prop="name">
        <el-input v-model="formData.name"/>
      </el-form-item>
      <el-form-item label="项目关键字" prop="keyword">
        <el-input v-model="formData.keyword"/>
      </el-form-item>
      <el-form-item label="项目描述信息" prop="description">
        <el-input v-model="formData.description" type="textarea" maxlength="200" show-word-limit/>
      </el-form-item>
      <el-form-item label="项目负责人" prop="owner">
        <el-select v-model="formData.owner" placeholder="请选择..." no-data-text="暂无用户">
          <el-option
              v-for="user in users"
              :key="user.id"
              :label="user.realName"
              :value="user.id"
          />
        </el-select>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-row class="row-bg" justify="end">
        <div class="flex-grow" />
        <el-button type="info" @click="handleReturn" round>返回项目列表</el-button>
        <el-button type="primary" @click="handleSubmit" round>提交</el-button>
      </el-row>
    </template>
  </el-card>
</template>

<style>

</style>
