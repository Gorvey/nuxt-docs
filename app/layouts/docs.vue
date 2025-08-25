<script setup lang="ts">
import type { ContentNavigationItem } from '@nuxt/content'

const navigation = inject<Ref<ContentNavigationItem[]>>('navigation')!
const route = useRoute()

// 获取第一层级数据作为选择器选项
const firstLevelOptions = computed(() => {
  if (!navigation.value) return []
  return navigation.value.map(item => ({
    label: item.title,
    value: item.path,
    item
  }))
})

// 当前选中的第一层级
const selectedFirstLevel = ref('')
const selectedFirstLevelIcon = computed(() => {
  const selectedItem = navigation.value?.find(item => item.path === selectedFirstLevel.value)
  return selectedItem?.icon || ''
})
// 根据当前路由自动选择对应的第一层级
watchEffect(() => {
  if (route.path && firstLevelOptions.value.length > 0) {
    const matchedOption = firstLevelOptions.value.find(option =>
      route.path.startsWith(option.value)
    )
    if (matchedOption) {
      selectedFirstLevel.value = matchedOption.value
    }
  }
})

// 获取当前选中层级的children数据
const currentNavigation = computed(() => {
  if (!selectedFirstLevel.value || !navigation.value) return []
  const selectedItem = navigation.value.find(item => item.path === selectedFirstLevel.value)
  return selectedItem?.children || []
})

// 处理选择器变化
const handleFirstLevelChange = (value: unknown) => {
  if (!value || typeof value !== 'string') return
  selectedFirstLevel.value = value
  // 如果选中的层级有默认页面，则导航到该页面
  const selectedItem = navigation.value?.find(item => item.path === value)
  if (selectedItem && !selectedItem.page && selectedItem.children && selectedItem.children.length > 0) {
    const firstChild = selectedItem.children[0]
    if (firstChild?.path) {
      navigateTo(firstChild.path)
    }
  }
}
</script>

<template>
  <UContainer>
    <UPage>
      <template #left>
        <UPageAside>
          <div class="mb-4">
            <USelect
              v-model="selectedFirstLevel"
              :items="firstLevelOptions"
              :icon="selectedFirstLevelIcon as string"
              placeholder="选择文档分类"
              class="w-full"
              @update:model-value="handleFirstLevelChange"
            />
          </div>
          <UContentNavigation
            v-if="currentNavigation.length > 0"
            highlight
            :navigation="currentNavigation"
          />
        </UPageAside>
      </template>

      <slot />
    </UPage>
  </UContainer>
</template>
