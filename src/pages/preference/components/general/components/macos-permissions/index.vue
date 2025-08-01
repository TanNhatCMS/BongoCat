<script setup lang="ts">
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow'
import { message } from '@tauri-apps/plugin-dialog'
import { Space } from 'ant-design-vue'
import { checkInputMonitoringPermission, requestInputMonitoringPermission } from 'tauri-plugin-macos-permissions-api'
import { onMounted, ref } from 'vue'

import ProList from '@/components/pro-list/index.vue'
import ProListItem from '@/components/pro-list-item/index.vue'
import { isMac } from '@/utils/platform'

const authorized = ref(false)

onMounted(async () => {
  authorized.value = await checkInputMonitoringPermission()

  if (authorized.value) return

  const appWindow = getCurrentWebviewWindow()

  await appWindow.setAlwaysOnTop(true)
  // 如果权限已开启，先选中后点击“-”按钮将其删除，再重新手动添加，并重启应用以确保权限生效。If the permission is enabled, first select it and then click the "-" button to delete it, then add it manually again, and restart the app to ensure that the permissions take effect.
  // 输入监控权限 Enter monitoring permissions
  // 前往开启 Go to Turn on
  await message('Nếu quyền được bật, trước tiên hãy chọn nó và sau đó nhấp vào nút "-" để xóa nó, sau đó thêm nó theo cách thủ công và khởi động lại ứng dụng để đảm bảo rằng các quyền có hiệu lực.', {
    title: 'Nhập quyền giám sát',
    okLabel: 'Đi bật',
    kind: 'warning',
  })

  await appWindow.setAlwaysOnTop(false)

  await requestInputMonitoringPermission()
})
</script>

<template>
  <!-- 权限设置  Permission settings -->
  <ProList
    v-if="isMac"
    title="Cài đặt quyền"
  >
    <!-- 开启输入监控权限，以便接收系统的键盘和鼠标事件来响应你的操作。  Turn on input monitoring permissions to receive system keyboard and mouse events to respond to your operations.  -->
    <!-- 输入监控权限 Enter monitoring permissions -->
    <ProListItem
      description="Bật quyền giám sát đầu vào để nhận các sự kiện bàn phím và chuột của hệ thống nhằm phản hồi các thao tác của bạn."
      title="Nhập quyền giám sát"
    >
      <Space
        v-if="authorized"
        class="text-success font-bold"
        :size="4"
      >
        <div class="i-solar:verified-check-bold text-4.5" />
        <!-- 已授权 Authorized -->
        <span>Được ủy quyền</span>
      </Space>

      <Space
        v-else
        class="cursor-pointer text-danger font-bold"
        :size="4"
        @click="requestInputMonitoringPermission"
      >
        <div class="i-solar:round-arrow-right-bold text-4.5" />
        <!-- 去授权 Go to authorization -->
        <span>Đi đến ủy quyền</span>
      </Space>
    </ProListItem>
  </ProList>
</template>
