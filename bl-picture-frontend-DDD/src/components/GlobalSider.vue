<template>
  <div id="globalSider">
    <a-layout-sider
      v-if="loginUserStore.loginUser.id"
      class="sider"
      width="200"
      breakpoint="lg"
      collapsed-width="0"
    >
      <a-menu
        mode="inline"
        v-model:selectedKeys="current"
        :items="menuItems"
        @click="doMenuClick"
      />
    </a-layout-sider>
  </div>
</template>

<script lang="ts" setup>
import { computed, h, ref, watchEffect } from 'vue'
import {
  PictureOutlined,
  UserOutlined,
  TeamOutlined,
  CloudUploadOutlined,
} from '@ant-design/icons-vue'
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore'
import { SPACE_TYPE_ENUM } from '@/constants/space'
import { listMyTeamSpaceUsingPost } from '@/api/spaceUserController'
import { message } from 'ant-design-vue'

const loginUserStore = useLoginUserStore()

// 固定的菜单列表
const fixedMenuItems = [
  {
    key: '/',
    label: '公共图库',
    icon: () => h(PictureOutlined),
  },
  {
    key: '/add_picture',
    label: '上传图片',
    icon: () => h(CloudUploadOutlined),
  },
  {
    key: '/my_space',
    label: '我的空间',
    icon: () => h(UserOutlined),
  },
  {
    key: '/add_space?type=' + SPACE_TYPE_ENUM.TEAM,
    label: '创建团队',
    icon: () => h(TeamOutlined),
  },
]

const router = useRouter()

// 当前选中菜单
const current = ref<string[]>([])
// 监听路由变化，更新当前选中菜单
router.afterEach((to, from, failure) => {
  current.value = [to.path]
})

// 路由跳转事件
const doMenuClick = ({ key }: { key: string }) => {
  router.push(key)
}

// 全局侧边栏
const teamSpaceList = ref<API.SpaceUserVO[]>([])
const menuItems = computed(() => {
  // 没有团队空间，只展示固定菜单
  if (teamSpaceList.value.length < 1) {
    if (!loginUserStore.loginUser.vipSign) {
      // 去除 fixedMenuItems 里面的 key = '/add_space?type=' + SPACE_TYPE_ENUM.TEAM 的这一项
      return fixedMenuItems.filter((item) => item.key !== '/add_space?type=' + SPACE_TYPE_ENUM.TEAM)
    }
    return fixedMenuItems
  }
  // 展示团队空间分组
  const teamSpaceSubMenus = teamSpaceList.value.map((spaceUser) => {
    const space = spaceUser.space
    return {
      key: '/space/' + spaceUser.spaceId,
      label: space?.spaceName,
    }
  })
  const teamSpaceMenuGroup = {
    type: 'group',
    label: '团队空间',
    key: 'teamSpace',
    children: teamSpaceSubMenus,
  }
  if (!loginUserStore.loginUser.vipSign) {
    // 去除 fixedMenuItems 里面的 key = '/add_space?type=' + SPACE_TYPE_ENUM.TEAM 的这一项
    return [
      ...fixedMenuItems.filter((item) => item.key !== '/add_space?type=' + SPACE_TYPE_ENUM.TEAM),
      teamSpaceMenuGroup,
    ]
  }
  return [...fixedMenuItems, teamSpaceMenuGroup]
})

// 加载团队空间列表
const fetchTeamSpaceList = async () => {
  const res = await listMyTeamSpaceUsingPost()
  if (res.data.code === 0 && res.data.data) {
    teamSpaceList.value = res.data.data
  } else {
    message.error('加载我的团队空间失败，' + res.data.message)
  }
}

/**
 * 监听变量，改变时触发数据的重新加载
 */
watchEffect(() => {
  // 登录才加载
  if (loginUserStore.loginUser.id) {
    fetchTeamSpaceList()
  }
})
</script>

<style scoped></style>
