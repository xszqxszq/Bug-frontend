<script lang="ts">
import {reactive, ref} from 'vue'
import { useRoute } from 'vue-router'
import { ElMessage } from 'element-plus'
import { EditPen } from '@element-plus/icons-vue'
import BreadCrumbNav from '@/components/BreadCrumbNav.vue'
import project from '@/api/project.ts'
import bug from '@/api/bug.ts'
import utils from '@/api/utils.ts'

const moduleName = 'bug'
const id = ref('')
const grades = reactive([])
const formData = reactive({
  name: '',
  grade: '',
  module: '',
  feature: '',
  description: '',
})
const loading = ref(true)
const formDataRef = ref()
const nowProject = reactive(project.empty)

export default {
  components: {EditPen, BreadCrumbNav},
  setup() {
    return {
      loading,
      formData,
      formDataRef,
      formRules: reactive({
        name: [
          {
            required: true,
            message: '请输入 Bug 标题',
            trigger: 'blur'
          },
          { max: 50, message: 'Bug 标题不能超过50个字', trigger: 'blur' },
        ],
        description: [
          { max: 500, message: 'Bug 详情不能超过500个字', trigger: 'blur' },
        ],
        grade: [
          {
            required: true,
            message: '请选择 Bug 等级',
            trigger: 'blur'
          }
        ],
        module: [
          {
            required: true,
            message: '请选择所属模块',
            trigger: 'blur'
          }
        ],
        feature: [
          {
            required: true,
            message: '请选择所属功能',
            trigger: 'blur'
          }
        ],
      }),
    }
  },
  data() {
    return {
      grades,
      id,
      project: nowProject
    }
  },
  mounted() {
    loading.value = true
    Object.keys(formData).forEach(function(key) {
      formData[key] = ''
    })
    id.value = useRoute().query.id ?.toString()
    bug.getGrades().then((result) => {
      grades.length = 0
      Object.assign(grades, utils.toOptions(result, true))
    })
    project.get(id.value).then((result) => {
      if (result)
        Object.assign(nowProject, result)
      loading.value = false
    })
  },
  computed: {
    features() {
      if (this.formData.module != '') {
        let m = this.project.modules.find(m => { return m.id == this.formData.module })
        return utils.toOptions(m?.features, true)
      }
      return utils.toOptions([], true)
    },
  },
  methods: {
    handleSubmit() {
      try {
        formDataRef.value.validate().then(() => {
          bug.create(formData).then((response) => {
            if (response.success) {
              ElMessage.success('添加成功')
              formDataRef.value.resetFields()
              this.$router.push('/' + moduleName + '/bugs?id='+this.id)
            } else {
              ElMessage.error('添加失败')
            }
          })
        })
      } catch (error) {
        ElMessage.error('请检查输入内容')
      }
    },
    handleReturn() {
      this.$router.go(-1)
    },
    clearFeature() {
      this.formData.feature = ''
    },
  }
}
</script>

<template>
  <BreadCrumbNav :page-paths="['Bug 管理', '项目列表', 'Bug 列表', 'Bug 添加']"></BreadCrumbNav>
  <el-card class="info-card" shadow="never" v-loading="loading">
    <template #header>
      <div class="card-header">
        <el-icon>
          <EditPen/>
        </el-icon>&nbsp;&nbsp;[ {{ project.name }} ] Bug 添加
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
      <el-form-item label="Bug 标题" prop="name">
        <el-input v-model="formData.name"/>
      </el-form-item>
      <el-form-item label="Bug 等级" prop="grade">
        <el-select v-model="formData.grade" placeholder="请选择..." no-data-text="暂无等级">
          <el-option
              v-for="m in grades"
              :key="m.value"
              :label="m.name"
              :value="m.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="所属模块" prop="module">
        <el-select v-model="formData.module" placeholder="请选择..." no-data-text="暂无模块" @change="clearFeature()">
          <el-option
              v-for="m in project.modules"
              :key="m.id"
              :label="m.name"
              :value="m.id"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="所属功能" prop="feature">
        <el-select v-model="formData.feature" placeholder="请选择..." no-data-text="暂无功能">
          <el-option
              v-for="f in features"
              :key="f.value"
              :label="f.name"
              :value="f.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="Bug 详情" prop="description">
        <el-input v-model="formData.description" type="textarea" maxlength="500" show-word-limit/>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-row class="row-bg" justify="end">
        <div class="flex-grow" />
        <el-button type="info" @click="handleReturn" round>返回 Bug 列表</el-button>
        <el-button type="primary" @click="handleSubmit" round>提交</el-button>
      </el-row>
    </template>
  </el-card>
</template>

<style>

</style>
