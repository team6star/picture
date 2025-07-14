<template>
  <div id="globalHeader">
    <!-- {{ JSON.stringify(loginUserStore.loginUser) }} -->
    <a-row :wrap="false">
      <a-col flex="200px">
        <router-link to="/">
          <div class="title-bar">
            <img class="logo" src="../assets/logo.png" alt="logo" />
            <div class="title">云图库</div>
          </div>
        </router-link>
      </a-col>
      <a-col flex="auto">
        <a-menu
          v-model:selectedKeys="current"
          mode="horizontal"
          @click="onMenuClick"
          :items="items"
        />
      </a-col>
      <a-col flex="120px">
        <div class="user-login-status">
          <div v-if="loginUserStore.loginUser.id">
            <a-dropdown>
              <a-space>
                <a-avatar :src="loginUserStore.loginUser.userAvatar" />
                {{ loginUserStore.loginUser.userName ?? '无名' }}
              </a-space>
              <template #overlay>
                <a-menu>
                  <a-menu-item @click="showProfileModal"><UserOutlined />个人中心</a-menu-item>
                  <a-menu-item @click="doLogout"><LogoutOutlined />退出登录</a-menu-item>
                </a-menu>
              </template>
            </a-dropdown>
          </div>
          <div v-else>
            <a-button type="primary" href="/user/login">登录</a-button>
          </div>
        </div>
      </a-col>
    </a-row>
  </div>
  <a-modal v-model:open="profileVisible" title="个人中心" @ok="handleProfileUpdate">
    <a-form :model="profileForm" layout="vertical">
      <a-form-item label="用户名">
        <a-input v-model:value="profileForm.userName" />
      </a-form-item>
      <a-form-item label="用户角色">
        <a-select v-model:value="profileForm.userRole">
          <a-select-option value="admin">管理员</a-select-option>
          <a-select-option value="user">普通用户</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label="用户简介">
        <a-textarea v-model:value="profileForm.userProfile" />
      </a-form-item>
    </a-form>
  </a-modal>
</template>
<script lang="ts" setup>
import { h, ref, computed, reactive } from 'vue'
import { HomeOutlined, LogoutOutlined, UserOutlined } from '@ant-design/icons-vue'
import { message, type MenuProps } from 'ant-design-vue'
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore'
import { userLogoutUsingPost, updateUserUsingPost } from '@/api/userController'

const loginUserStore = useLoginUserStore()

// 个人中心相关状态
const profileVisible = ref(false)
const profileForm = reactive({
  id: undefined,
  userName: '',
  userRole: '',
  userAvatar: '',
  userProfile: '',
})

// 显示个人中心模态框
const showProfileModal = () => {
  Object.assign(profileForm, loginUserStore.loginUser)
  profileVisible.value = true
}

// 提交个人信息更新
const handleProfileUpdate = async () => {
  try {
    const res = await updateUserUsingPost(profileForm)
    if (res.data.code === 0) {
      message.success('更新成功')
      loginUserStore.setLoginUser({ ...loginUserStore.loginUser, ...profileForm })
      profileVisible.value = false
    }
  } catch (error) {
    message.error('更新失败')
  }
}

// ref<MenuProps['items']>
// 菜单列表
const originItems = [
  {
    key: '/',
    icon: () => h(HomeOutlined),
    label: '主页',
    title: '主页',
  },
  {
    key: '/admin/userManage',
    label: '用户管理',
    title: '用户管理',
  },
  {
    key: 'others',
    label: h('a', { href: 'https://www.codefather.cn', target: '_blank' }, '编程导航'),
    title: '编程导航',
  },
]

const router = useRouter()
const current = ref<string[]>([''])

const onMenuClick = ({ key }) => {
  router.push({
    path: key,
  })
}
// 根据权限过滤菜单项
const filterMenus = (menus = [] as MenuProps['items']) => {
  return menus?.filter((menu) => {
    // 管理员才能看到 /admin 开头的菜单
    if (menu?.key?.startsWith('/admin')) {
      const loginUser = loginUserStore.loginUser
      if (!loginUser || loginUser.userRole !== 'admin') {
        return false
      }
    }
    return true
  })
}
// 展示在菜单的路由数组
const items = computed<MenuProps['items']>(() => filterMenus(originItems))
// 用户注销
const doLogout = async () => {
  const res = await userLogoutUsingPost()
  console.log(res)
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({
      userName: '未登录',
    })
    message.success('退出登录成功')
    await router.push('/user/login')
  } else {
    message.error('退出登录失败,' + res.data.message)
  }
}

router.afterEach((to) => {
  current.value = [to.path]
})
</script>

<style scoped>
.title-bar {
  display: flex;
  align-items: center;
}

.title {
  color: black;
  font-size: 18px;
  margin-left: 16px;
}

.logo {
  height: 48px;
}
</style>
