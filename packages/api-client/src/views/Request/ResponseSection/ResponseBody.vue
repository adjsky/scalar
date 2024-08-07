<script lang="ts" setup>
import DataTable from '@/components/DataTable/DataTable.vue'
import DataTableHeader from '@/components/DataTable/DataTableHeader.vue'
import DataTableRow from '@/components/DataTable/DataTableRow.vue'
import ViewLayoutCollapse from '@/components/ViewLayout/ViewLayoutCollapse.vue'
import { useDarkModeState } from '@/hooks'
import { useWorkspace } from '@/store/workspace'
import { ScalarCodeBlock, ScalarIcon } from '@scalar/components'
import { getThemeStyles } from '@scalar/themes'
import { computed, nextTick, ref, watch } from 'vue'

const props = withDefaults(
  defineProps<{
    title: string
    active: boolean
    data: any
    headers: { name: string; value: string; required: boolean }[]
  }>(),
  {
    active: false,
    data: null,
  },
)

const { activeWorkspace } = useWorkspace()

const codeLanguage = computed(() => {
  const contentTypeHeader =
    props.headers.find((header) => header.name.toLowerCase() === 'content-type')
      ?.value ?? ''

  if (contentTypeHeader.includes('json')) return 'json'
  if (contentTypeHeader.includes('html')) return 'html'
  return 'plaintext'
})

const iframe = ref<HTMLIFrameElement | null>(null)

const activePreview = ref('raw')
const previewOptions = ['raw', 'preview']

const { isDark } = useDarkModeState()
watch(
  () => activePreview.value,
  async (newValue) => {
    if (newValue === 'preview') {
      await nextTick()
      if (iframe.value) {
        const doc =
          iframe.value.contentDocument || iframe.value.contentWindow?.document
        if (!doc) return
        doc.open()
        doc.write(props.data)
        doc.close()
      }
    }
  },
)

watch(
  () => isDark.value,
  async (newValue) => {
    await nextTick()
    if (iframe.value) {
      const doc =
        iframe.value.contentDocument || iframe.value.contentWindow?.document
      if (!doc) return
      doc.body.classList.toggle('dark-mode', isDark.value)
      doc.body.classList.toggle('light-mode', !isDark.value)
    }
  },
)

watch(
  () => activePreview.value,
  async (newValue) => {
    if (newValue === 'preview') {
      await nextTick()
      if (iframe.value) {
        const doc =
          iframe.value.contentDocument || iframe.value.contentWindow?.document
        if (!doc) return
        doc.open()
        doc.write(props.data)
        doc.body.classList.toggle('dark-mode', isDark.value)
        doc.body.classList.toggle('light-mode', !isDark.value)
        doc.write(
          '<style>body,html {body:not(:has(* + style + style)) {font-family: var(--scalar-font-code); font-size: 12px; font-weight: 400;background:var(--scalar-background-1); color: var(--scalar-color-2)}</style>',
        )
        if (activeWorkspace.value?.themeId) {
          doc.write(
            `<style>${getThemeStyles(activeWorkspace.value.themeId)}</style>`,
          )
        }
        doc.close()
      }
    }
  },
)
</script>
<template>
  <ViewLayoutCollapse>
    <template #title>{{ title }}</template>
    <template v-if="active">
      <DataTable :columns="['']">
        <DataTableRow>
          <DataTableHeader
            class="relative col-span-full flex h-8 cursor-pointer items-center px-[2.25px] py-[2.25px]">
            <div
              class="text-c-2 group-hover:text-c-1 flex h-full w-full items-center justify-start rounded px-1.5">
              <span class="capitalize">{{ activePreview }}</span>
              <ScalarIcon
                class="text-c-3 ml-1 mt-px"
                icon="ChevronDown"
                size="xs" />
            </div>
            <select
              v-model="activePreview"
              class="absolute inset-0 w-auto opacity-0"
              @click.prevent>
              <option
                v-for="value in previewOptions"
                :key="value"
                :value="value">
                {{ value }}
              </option>
            </select>
          </DataTableHeader>
        </DataTableRow>
        <DataTableRow>
          <template v-if="activePreview === 'raw'">
            <ScalarCodeBlock
              class="force-text-sm rounded-b border-t-0"
              :content="data"
              :lang="codeLanguage" />
          </template>
          <template v-else>
            <iframe
              ref="iframe"
              allowfullscreen
              allowtransparency="true"
              class="w-full aspect-video"
              frameborder="0"></iframe>
          </template>
        </DataTableRow>
      </DataTable>
    </template>
  </ViewLayoutCollapse>
</template>
<style scoped>
.force-text-sm {
  --scalar-small: 13px;
}
iframe {
  background-color: transparent;
}
</style>
