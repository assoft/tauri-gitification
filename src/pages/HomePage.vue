<script lang="ts">
</script>

<script lang="ts" setup>
import { open } from '@tauri-apps/api/shell'
import { ref } from 'vue'
import { useStore } from '../stores/store'
import NotificationItem from '../components/NotificationItem.vue'
import type { Thread } from '../api/notifications'
import { toGithubWebURL } from '../utils/github'
import { AppStorage } from '../storage'
import NotificationSkeleton from '../components/NotificationSkeleton.vue'
import type { Option } from '../types'
import { useElementNavigation } from '../composables/useElementNavigation'
import EmptyState, { EmptyStateIconSize } from '../components/EmptyState.vue'
import { Icons } from '../components/Icons'
import AppButton from '../components/AppButton.vue'

const store = useStore()

if (store.currentPageState.fetchOnEnter)
  store.fetchNotifications(true)

function handleNotificationClick(notification: Thread) {
  const url = toGithubWebURL({ notification, userId: AppStorage.get('user')!.id })
  open(url)
}

function handleRepoClick(repoFullName: string) {
  open(`https://github.com/${repoFullName}`)
}

const home = ref<Option<HTMLElement>>(null)

useElementNavigation({
  target: home,
  navigateNextHotkey: 'down',
  navigatePreviousHotkey: 'up',
  targetQuery: '.notification-item, .notification-title',
})
</script>

<template>
  <div
    ref="home"
    class="home"
  >
    <NotificationSkeleton v-if="store.skeletonVisible" />

    <EmptyState
      v-else-if="store.failedLoadingNotifications"
      :iconSize="EmptyStateIconSize.Big"
      :icon="Icons.X"
      description="Oopsie! Couldn't load notifications."
    >
      <template #footer>
        <AppButton @click="store.fetchNotifications()">
          Refresh
        </AppButton>
      </template>
    </EmptyState>

    <EmptyState
      v-else-if="store.notifications.length === 0"
      :iconSize="EmptyStateIconSize.Big"
      :icon="Icons.Check"
      description="It's all clear sir!"
    />

    <NotificationItem
      v-for="item of store.notifications"
      :key="item.id"
      :value="item"
      @click:notification="handleNotificationClick"
      @click:repo="handleRepoClick"
    />
  </div>
</template>

<style lang="scss" scoped>
  .home {
    padding: 10px;
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow-y: auto;
    scroll-padding-top: 10px;
    scroll-padding-bottom: 10px;
  }
</style>
