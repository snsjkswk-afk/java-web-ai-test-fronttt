<template>
  <div class="emp-container">
    <!-- 搜索区域 -->
    <el-card shadow="never" class="search-card">
      <el-form :model="queryParams" inline>
        <el-form-item label="姓名">
          <el-input
            v-model="queryParams.name"
            placeholder="请输入员工姓名"
            clearable
            @clear="handleSearch"
          />
        </el-form-item>
        <el-form-item label="性别">
          <el-select v-model="queryParams.gender" placeholder="请选择" clearable style="width: 120px">
            <el-option label="男" :value="1" />
            <el-option label="女" :value="2" />
          </el-select>
        </el-form-item>
        <el-form-item label="职位">
          <el-select v-model="queryParams.job" placeholder="请选择职位" clearable style="width: 150px">
            <el-option label="班主任" :value="1" />
            <el-option label="讲师" :value="2" />
            <el-option label="学工主管" :value="3" />
            <el-option label="教研主管" :value="4" />
            <el-option label="咨询师" :value="5" />
          </el-select>
        </el-form-item>
        <el-form-item label="入职时间">
          <el-date-picker
            v-model="dateRange"
            type="daterange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            value-format="YYYY-MM-DD"
            @change="handleDateChange"
          />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" :icon="Search" @click="handleSearch">查询</el-button>
          <el-button :icon="Refresh" @click="handleReset">重置</el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <!-- 表格区域 -->
    <el-card shadow="never" class="table-card">
      <el-button type="primary" :icon="Plus" @click="handleAdd" style="margin-bottom: 16px">
        新增员工
      </el-button>

      <el-table :data="empList" border stripe v-loading="loading" style="width: 100%">
        <el-table-column prop="name" label="姓名" align="center" width="100" />
        <el-table-column label="性别" align="center" width="70">
          <template #default="{ row }">
            <el-tag :type="row.gender === 1 ? '' : 'danger'" size="small">
              {{ row.gender === 1 ? '男' : '女' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="phone" label="手机号" align="center" width="130" />
        <el-table-column label="职位" align="center" width="100">
          <template #default="{ row }">
            {{ jobMap[row.job] || '未知' }}
          </template>
        </el-table-column>
        <el-table-column prop="salary" label="薪资" align="center" width="80" />
        <el-table-column prop="deptName" label="部门" align="center" width="100" />
        <el-table-column prop="entryDate" label="入职日期" align="center" width="120" />
        <el-table-column label="头像" align="center" width="80">
          <template #default="{ row }">
            <el-avatar v-if="row.image" :src="row.image" :size="40" />
            <span v-else>-</span>
          </template>
        </el-table-column>
        <el-table-column label="操作" align="center" width="150" fixed="right">
          <template #default="{ row }">
            <el-button type="primary" :icon="Edit" link @click="handleEdit(row)">
              编辑
            </el-button>
            <el-button type="danger" :icon="Delete" link @click="handleDelete(row)">
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="pagination">
        <el-pagination
          v-model:current-page="queryParams.page"
          v-model:page-size="queryParams.pageSize"
          :page-sizes="[5, 10, 20, 50]"
          :total="total"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="loadEmpList"
          @current-change="loadEmpList"
        />
      </div>
    </el-card>

    <!-- 新增/编辑对话框 -->
    <el-dialog
      v-model="dialogVisible"
      :title="isEdit ? '编辑员工' : '新增员工'"
      width="700px"
      @close="resetForm"
    >
      <el-form
        ref="formRef"
        :model="empForm"
        :rules="rules"
        label-width="100px"
      >
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="用户名" prop="username">
              <el-input v-model="empForm.username" placeholder="请输入用户名" :disabled="isEdit" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="姓名" prop="name">
              <el-input v-model="empForm.name" placeholder="请输入姓名" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="密码" prop="password" v-if="!isEdit">
              <el-input v-model="empForm.password" type="password" placeholder="请输入密码" show-password />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="性别" prop="gender">
              <el-radio-group v-model="empForm.gender">
                <el-radio :value="1">男</el-radio>
                <el-radio :value="2">女</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="手机号" prop="phone">
              <el-input v-model="empForm.phone" placeholder="请输入手机号" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="职位" prop="job">
              <el-select v-model="empForm.job" placeholder="请选择职位" style="width: 100%">
                <el-option label="班主任" :value="1" />
                <el-option label="讲师" :value="2" />
                <el-option label="学工主管" :value="3" />
                <el-option label="教研主管" :value="4" />
                <el-option label="咨询师" :value="5" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="薪资" prop="salary">
              <el-input-number v-model="empForm.salary" :min="0" style="width: 100%" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="部门" prop="deptId">
              <el-select v-model="empForm.deptId" placeholder="请选择部门" style="width: 100%">
                <el-option
                  v-for="dept in deptOptions"
                  :key="dept.id"
                  :label="dept.name"
                  :value="dept.id"
                />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="入职日期" prop="entryDate">
              <el-date-picker
                v-model="empForm.entryDate"
                type="date"
                placeholder="请选择日期"
                value-format="YYYY-MM-DD"
                style="width: 100%"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="头像">
              <el-upload
                class="avatar-uploader"
                action="/api/upload"
                :headers="uploadHeaders"
                :show-file-list="false"
                :on-success="handleUploadSuccess"
                :before-upload="beforeUpload"
              >
                <el-avatar v-if="empForm.image" :src="empForm.image" :size="80" shape="square" />
                <el-icon v-else class="avatar-uploader-icon" :size="30"><Plus /></el-icon>
              </el-upload>
            </el-form-item>
          </el-col>
        </el-row>

        <!-- 工作经历 -->
        <el-divider content-position="left">工作经历</el-divider>
        <div v-for="(expr, index) in empForm.exprList" :key="index" class="expr-item">
          <el-row :gutter="10">
            <el-col :span="6">
              <el-form-item :label="'开始时间'">
                <el-date-picker
                  v-model="expr.begin"
                  type="date"
                  placeholder="选择日期"
                  value-format="YYYY-MM-DD"
                  style="width: 200px"
                />
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="结束时间">
                <el-date-picker
                  v-model="expr.end"
                  type="date"
                  placeholder="选择日期"
                  value-format="YYYY-MM-DD"
                  style="width: 200px"
                />
              </el-form-item>
            </el-col>
            <el-col :span="5">
              <el-form-item label="公司">
                <el-input v-model="expr.company" placeholder="公司名称" style="width: 160px" />
              </el-form-item>
            </el-col>
            <el-col :span="5">
              <el-form-item label="职位">
                <el-input v-model="expr.job" placeholder="职位" style="width: 160px" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-button type="danger" :icon="Delete" circle @click="removeExpr(index)" style="margin-top: 2px" />
            </el-col>
          </el-row>
        </div>
        <el-button type="primary" link @click="addExpr" style="margin-left: 100px; width: 180px">
          <el-icon><Plus /></el-icon> 添加工作经历
        </el-button>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" :loading="submitLoading" @click="handleSubmit">
          确定
        </el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { Plus, Edit, Delete, Search, Refresh } from '@element-plus/icons-vue'
import request from '../utils/request'

const jobMap = { 1: '班主任', 2: '讲师', 3: '学工主管', 4: '教研主管', 5: '咨询师' }

const loading = ref(false)
const empList = ref([])
const total = ref(0)
const deptOptions = ref([])
const dialogVisible = ref(false)
const isEdit = ref(false)
const submitLoading = ref(false)
const formRef = ref(null)
const dateRange = ref([])
const queryParams = reactive({
  page: 1,
  pageSize: 10,
  name: '',
  gender: null,
  job: null,
  begin: '',
  end: ''
})

const empForm = reactive({
  id: null,
  username: '',
  password: '',
  name: '',
  gender: 1,
  phone: '',
  job: null,
  salary: 0,
  image: '',
  entryDate: '',
  deptId: null,
  exprList: []
})

const uploadHeaders = computed(() => ({
  token: localStorage.getItem('token')
}))

const rules = {
  username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
  name: [{ required: true, message: '请输入姓名', trigger: 'blur' }],
  password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
  gender: [{ required: true, message: '请选择性别', trigger: 'change' }],
  phone: [{ required: true, message: '请输入手机号', trigger: 'blur' }],
  job: [{ required: true, message: '请选择职位', trigger: 'change' }],
  entryDate: [{ required: true, message: '请选择入职日期', trigger: 'change' }],
  deptId: [{ required: true, message: '请选择部门', trigger: 'change' }]
}

// 加载员工列表
const loadEmpList = async () => {
  loading.value = true
  try {
    const res = await request.get('/emps', { params: queryParams })
    empList.value = res.data.data
    total.value = res.data.count
  } catch (error) {
    console.error('加载员工列表失败:', error)
  } finally {
    loading.value = false
  }
}

// 加载部门列表
const loadDeptOptions = async () => {
  try {
    const res = await request.get('/depts')
    deptOptions.value = res.data
  } catch (error) {
    console.error('加载部门列表失败:', error)
  }
}

// 日期范围变化
const handleDateChange = (val) => {
  if (val) {
    queryParams.begin = val[0]
    queryParams.end = val[1]
  } else {
    queryParams.begin = ''
    queryParams.end = ''
  }
}

// 查询
const handleSearch = () => {
  queryParams.page = 1
  loadEmpList()
}

// 重置
const handleReset = () => {
  queryParams.name = ''
  queryParams.gender = null
  queryParams.job = null
  queryParams.begin = ''
  queryParams.end = ''
  dateRange.value = []
  handleSearch()
}

// 新增
const handleAdd = () => {
  isEdit.value = false
  Object.assign(empForm, {
    id: null, username: '', password: '', name: '', gender: 1,
    phone: '', job: null, salary: 0, image: '', entryDate: '',
    deptId: null, exprList: []
  })
  dialogVisible.value = true
}

// 编辑
const handleEdit = async (row) => {
  isEdit.value = true
  try {
    const res = await request.get(`/emps/${row.id}`)
    const emp = res.data
    Object.assign(empForm, {
      ...emp,
      exprList: emp.exprList || []
    })
    dialogVisible.value = true
  } catch (error) {
    console.error('加载员工详情失败:', error)
  }
}

const handleDelete = async (row) => {
  try {
    await ElMessageBox.confirm(`确定要删除员工 "${row.name}" 吗?`, '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    })
    await request.delete(`/emps/${row.id}`)
    ElMessage.success('删除成功')
    loadEmpList()
  } catch (error) {
    if (error !== 'cancel') {
      console.error('删除员工失败:', error)
    }
  }
}

// 上传成功
const handleUploadSuccess = (res) => {
  if (res.code === 1) {
    empForm.image = res.data
    ElMessage.success('上传成功')
  } else {
    ElMessage.error(res.msg || '上传失败')
  }
}

// 上传前校验
const beforeUpload = (file) => {
  const isImage = file.type.startsWith('image/')
  const isLt2M = file.size / 1024 / 1024 < 2
  if (!isImage) {
    ElMessage.error('只能上传图片文件')
    return false
  }
  if (!isLt2M) {
    ElMessage.error('图片大小不能超过 2MB')
    return false
  }
  return true
}

// 添加工作经历
const addExpr = () => {
  empForm.exprList.push({ begin: '', end: '', company: '', job: '' })
}

// 删除工作经历
const removeExpr = (index) => {
  empForm.exprList.splice(index, 1)
}

// 提交表单
const handleSubmit = async () => {
  const valid = await formRef.value.validate().catch(() => false)
  if (!valid) return

  submitLoading.value = true
  try {
    if (isEdit.value) {
      await request.put('/emps', empForm)
      ElMessage.success('更新成功')
    } else {
      await request.post('/emps', empForm)
      ElMessage.success('新增成功')
    }
    dialogVisible.value = false
    loadEmpList()
  } catch (error) {
    // 错误已在拦截器中处理
  } finally {
    submitLoading.value = false
  }
}

// 重置表单
const resetForm = () => {
  formRef.value?.resetFields()
}

onMounted(() => {
  loadEmpList()
  loadDeptOptions()
})
</script>

<style scoped>
.emp-container {
  min-height: 100%;
}

.search-card {
  margin-bottom: 16px;
}

.table-card {
  margin-bottom: 16px;
}

.pagination {
  display: flex;
  justify-content: flex-end;
  margin-top: 16px;
}

.expr-item {
  background: #f5f7fa;
  padding: 10px;
  border-radius: 6px;
  margin-bottom: 10px;
}

.avatar-uploader {
  cursor: pointer;
}

.avatar-uploader-icon {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999;
}

.avatar-uploader-icon:hover {
  border-color: #409EFF;
  color: #409EFF;
}
</style>