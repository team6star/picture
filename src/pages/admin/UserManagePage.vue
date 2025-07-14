<template>
  <div id="userManagePage">
    <!-- 搜索表单 -->
    <a-form layout="inline" :model="searchParams" @finish="doSearch">
      <a-form-item label="账号">
        <a-input v-model:value="searchParams.userAccount" placeholder="输入账号" allow-clear />
      </a-form-item>
      <a-form-item label="用户名">
        <a-input v-model:value="searchParams.userName" placeholder="输入用户名" allow-clear />
      </a-form-item>
      <a-form-item>
        <a-button type="primary" html-type="submit">搜索</a-button>
      </a-form-item>
    </a-form>
    <div style="margin-bottom: 16px" />
    <!-- 表格 -->
    <a-table
      :columns="columns"
      :data-source="dataList"
      :pagination="pagination"
      @change="doTableChange"
    >
      <template #bodyCell="{ column, record }">
        <template v-if="column.dataIndex === 'userAvatar'">
          <a-image :src="record.userAvatar" :width="120" />
        </template>
        <template v-else-if="column.dataIndex === 'userRole'">
          <div v-if="record.userRole === 'admin'">
            <a-tag color="green">管理员</a-tag>
          </div>
          <div v-else>
            <a-tag color="blue">普通用户</a-tag>
          </div>
        </template>
        <template v-if="column.dataIndex === 'createTime'">
          {{ dayjs(record.createTime).format('YYYY-MM-DD HH:mm:ss') }}
        </template>
        <template v-else-if="column.dataIndex === 'updateTime'">
          {{ dayjs(record.updateTime).format('YYYY-MM-DD HH:mm:ss') }}
        </template>
        <template v-else-if="column.key === 'action'">
          <a-space>
            <a-button type="primary" @click="handleEdit(record)">编辑</a-button>
            <a-button danger @click="doDelete(record.id)">删除</a-button>
          </a-space>
        </template>
      </template>
    </a-table>
  </div>
  <!-- // ... 在模板最后添加编辑模态框 ... -->
  <a-modal v-model:open="editVisible" title="编辑用户" @ok="handleEditSubmit">
    <a-form :model="editForm" layout="vertical">
      <a-form-item label="用户名">
        <a-input v-model:value="editForm.userName" />
      </a-form-item>
      <a-form-item label="用户角色">
        <a-select v-model:value="editForm.userRole">
          <a-select-option value="admin">管理员</a-select-option>
          <a-select-option value="user">普通用户</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label="用户头像">
        <a-upload
          name="file"
          :action="uploadUrl"
          @change="handleAvatarChange"
          :show-upload-list="false"
        >
          <a-button type="primary">上传新头像</a-button>
          <template v-if="editForm.userAvatar">
            <img :src="editForm.userAvatar" alt="avatar" style="width: 120px; margin-top: 10px" />
          </template>
        </a-upload>
      </a-form-item>
      <a-form-item label="用户简介">
        <a-textarea v-model:value="editForm.userProfile" />
      </a-form-item>
    </a-form>
  </a-modal>
</template>

<script lang="ts" setup>
import { computed, onMounted, reactive, ref } from 'vue'
import { message } from 'ant-design-vue'
import {
  deleteUserUsingPost,
  listUserVoByPageUsingPost,
  updateUserUsingPost,
} from '@/api/userController.ts'

import dayjs from 'dayjs'
import { testUploadFileUsingPost } from '@/api/fileController'

// 添加编辑相关状态
const editVisible = ref(false)
// https://yupicture-1318019277.cos.ap-shanghai.myqcloud.com/test/Lion.jpg
// const baseURL = 'http://localhost:8123'
// 修改COS访问域名
const COS_DOMAIN = 'https://yupicture-1318019277.cos.ap-shanghai.myqcloud.com'
// 上传接口
const uploadUrl = ref('http://localhost:8123/api/file/test/upload')
const editForm = reactive({
  id: undefined,
  userName: '',
  userRole: '',
  userAvatar: '',
  userProfile: '',
})

// 处理头像上传
const handleAvatarChange = async (info: any) => {
  if (info.file.status === 'done') {
    const res = await testUploadFileUsingPost({ file: info.file.originFileObj })
    if (res.data.code === 0) {
      // 拼接完整COS访问路径
      editForm.userAvatar = `${COS_DOMAIN}/${res.data.data}`
      message.success('头像上传成功')
    }
  }
}
// 编辑按钮点击处理
const handleEdit = (record) => {
  Object.assign(editForm, record)
  editVisible.value = true
}

// 提交编辑
const handleEditSubmit = async () => {
  try {
    const res = await updateUserUsingPost(editForm)
    if (res.data.code === 0) {
      message.success('更新成功')
      editVisible.value = false
      await fetchData()
    }
  } catch (error) {
    message.error('更新失败')
  }
}
const columns = [
  {
    title: 'id',
    dataIndex: 'id',
  },
  {
    title: '账号',
    dataIndex: 'userAccount',
  },
  {
    title: '用户名',
    dataIndex: 'userName',
  },
  {
    title: '头像',
    dataIndex: 'userAvatar',
  },
  {
    title: '简介',
    dataIndex: 'userProfile',
  },
  {
    title: '用户角色',
    dataIndex: 'userRole',
  },
  {
    title: '创建时间',
    dataIndex: 'createTime',
  },
  {
    title: '更新时间',
    dataIndex: 'updateTime',
  },
  {
    title: '操作',
    key: 'action',
  },
]

const doDelete = async (id: number) => {
  const res = await deleteUserUsingPost({ id })
  console.log('res', res)
  if (res.data.code === 0) {
    message.success('删除成功')
    // 刷新数据
    await fetchData()
  } else {
    message.error('删除失败')
  }
}

// 数据
const dataList = ref<API.UserVO[]>([])
const total = ref(0)

// 搜索条件
const searchParams = reactive<API.UserQueryRequest>({
  current: 1,
  pageSize: 10,
  sortField: 'createTime',
  sortOrder: 'ascend',
})

// 获取数据
const fetchData = async () => {
  const res = await listUserVoByPageUsingPost({
    ...searchParams,
  })
  if (res.data.data) {
    // console.log('用户数据响应:', res.data.data) // 添加调试日志
    dataList.value = res.data.data.records ?? []
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
}

// 表格变化之后，重新获取数据
const doTableChange = (page: any) => {
  searchParams.current = page.current
  searchParams.pageSize = page.pageSize
  fetchData()
}

// 搜索数据
const doSearch = () => {
  // 重置页码
  searchParams.current = 1
  fetchData()
}

// 分页参数
const pagination = computed(() => {
  return {
    current: searchParams.current,
    pageSize: searchParams.pageSize,
    total: total.value,
    showSizeChanger: true,
    showTotal: (total) => `共 ${total} 条`,
  }
})
// 页面加载时请求一次
onMounted(() => {
  fetchData()
})
</script>

<style scoped></style>
