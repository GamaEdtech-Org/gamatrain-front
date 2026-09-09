<template>
  <v-dialog
    v-model="dialogModel"
    max-width="440"
    :fullscreen="!mdAndUp"
    @click="clickOnOverlay"
  >
    <div
      class="select-dialog-panel w-100 d-flex flex-column bg-white pa-0 mobile-style"
      @click="clickOnModal"
    >
      <v-row class="select-dialog-header">
        <v-col cols="6">
          <span class="select-dialog-title">{{ titleModal }}</span>
        </v-col>
        <v-col
          cols="6"
          class="d-flex align-center justify-end ga-2"
        >
          <div
            v-if="hasSearch && compactResultCount"
            class="select-dialog-result-count"
          >
            <span
              v-if="filteredItems.length > 0"
              class="select-dialog-result-count__number"
            >{{ filteredItems.length }}</span>
            <span class="select-dialog-result-count__label">{{ filteredResultLabel }}</span>
          </div>
          <template v-else-if="hasSearch">
            <span class="text-h5 text-grey400">result</span>
            <span class="text-h4 text-green font-weight-bold">{{ filteredItems.length }}</span>
          </template>
          <v-icon
            class="select-dialog-close ml-4"
            size="20"
            role="button"
            tabindex="0"
            aria-label="Close dialog"
            @click="closeModal"
            @keydown.enter.stop.prevent="closeModal"
            @keydown.space.stop.prevent="closeModal"
          >
            md:cancel
          </v-icon>
        </v-col>
      </v-row>
      <v-row
        v-if="hasSearch"
        class="select-dialog-search-row"
      >
        <v-text-field
          v-model="searchText"
          :label="`Search ${titleModal}`"
          prepend-inner-icon="md:search"
          glow
          variant="outlined"
          color="#F4B400"
          icon-color="#1E2A44"
          density="compact"
          rounded="lg"
          hide-details
          class="select-dialog-search"
        >
          <template #append-inner>
            <v-btn
              class="select-dialog-search-button mr-n2"
              color="#F4B400"
              width="64"
              height="32"
              rounded="lg"
            >
              Search
            </v-btn>
          </template>
        </v-text-field>
      </v-row>
      <v-list
        v-if="!isLoading"
        class="select-dialog-list"
        max-height="320"
      >
        <v-list-item
          v-for="item in filteredItems"
          :key="item.title"
          :value="item.title"
          :active="item.id == selectedItem?.id"
          color="#1E2A44"
          class="select-dialog-item"
          :class="{
            'select-dialog-item--main-topic': titleModal === 'Topic' && item.season,
            'select-dialog-item--subtopic': titleModal === 'Topic' && !item.season,
          }"
          @click="changeSelectedItem(item)"
        >
          <template #prepend>
            <span
              v-if="showItemIcon"
              class="select-item-icon mr-2"
              :style="item.iconPadding ? { padding: `${item.iconPadding}px` } : undefined"
            >
              <v-img
                v-if="item.icon && !failedIconIds.has(String(item.id))"
                :src="getIconSrc(item)"
                :alt="item.title"
                contain
                @error="markIconFailed(item)"
              />
              <span
                v-else-if="item.contentIcon"
                :class="`${item.contentIcon} select-item-content-icon`"
                :style="{ color: item.color }"
              />
              <v-img
                v-else-if="fallbackIconSrc"
                :src="fallbackIconSrc"
                alt=""
                contain
              />
              <v-icon
                v-else
                size="34"
                color="#1E2A44"
              >{{ fallbackIcon }}</v-icon>
            </span>
            <v-avatar
              v-else-if="item.icon"
              size="34"
            >
              <v-img :src="`/images/boards/${item.icon}.svg`" />
            </v-avatar>
            <span
              v-else-if="item.contentIcon"
              :class="`${item.contentIcon} size-icon`"
              :style="{ color: item.color }"
            />
          </template>
          <v-list-item-title class="text-h5">
            <HighlightedText
              :text="item.title"
              :search-text="searchText"
            />
          </v-list-item-title>
        </v-list-item>
      </v-list>

      <div
        v-if="isLoading"
        class="select-dialog-loading text-center"
      >
        <v-progress-circular
          indeterminate
          :width="3"
          color="#F4B400"
        />
      </div>

      <v-alert
        v-if="searchText && filteredItems.length === 0 && !isLoading"
        type="info"
        color="#1E2A44"
        density="compact"
        class="select-dialog-empty"
        variant="tonal"
      >
        No {{ titleModal }} found matching "{{ searchText }}". Try a different
        search term.
      </v-alert>
      <v-alert
        v-if="
          searchText.length == 0 && filteredItems.length === 0 && !isLoading
        "
        type="info"
        color="#1E2A44"
        density="compact"
        class="select-dialog-empty"
        variant="tonal"
      >
        No {{ titleModal }} found.
      </v-alert>
    </div>
  </v-dialog>
</template>

<script setup>
import { useDisplay } from 'vuetify'
import { ref, computed, defineComponent, h } from 'vue'

// HighlightedText component for safe text highlighting
const HighlightedText = defineComponent({
  props: {
    text: {
      type: String,
      required: true,
    },
    searchText: {
      type: String,
      required: true,
    },
  },
  setup(props) {
    const parts = computed(() => {
      if (!props.searchText) return [{ text: props.text, highlight: false }]

      const regex = new RegExp(`(${props.searchText})`, 'gi')
      const segments = props.text.split(regex)

      return segments.map(segment => ({
        text: segment,
        highlight: segment.toLowerCase() === props.searchText.toLowerCase(),
      }))
    })

    return () =>
      h(
        'span',
        parts.value.map(part =>
          h(
            'span',
            {
              style: part.highlight
                ? 'background-color: #FFF8E1; color: #1E2A44;'
                : '',
            },
            part.text,
          ),
        ),
      )
  },
})

const { mdAndUp } = useDisplay()

const props = defineProps({
  titleModal: {
    type: String,
    required: true,
  },
  items: {
    type: Array,
    default: () => [],
  },
  showDialog: {
    type: Boolean,
    default: false,
  },
  selectedItem: {
    type: Object,
  },
  isLoading: {
    type: Boolean,
    default: false,
  },
  hasSearch: {
    type: Boolean,
    default: true,
  },
  compactResultCount: {
    type: Boolean,
    default: false,
  },
  showItemIcon: {
    type: Boolean,
    default: false,
  },
  iconSrc: {
    type: Function,
    default: null,
  },
  fallbackIcon: {
    type: String,
    default: 'md:school',
  },
  fallbackIconSrc: {
    type: String,
    default: '',
  },
})

const emit = defineEmits(['update:showDialog', 'changeSelectedItem'])

// Start Section Search Item In List
const searchText = ref('')
const failedIconIds = ref(new Set())

watch(
  () => props.showDialog,
  (isOpen) => {
    if (!isOpen) searchText.value = ''
  },
)

const getIconSrc = item => props.iconSrc?.(item) || item.icon
const markIconFailed = (item) => {
  failedIconIds.value = new Set([
    ...failedIconIds.value,
    String(item.id),
  ])
}
const filteredItems = computed(() => {
  if (!searchText.value) return props.items
  return props.items.filter(item =>
    item.title.toLowerCase().includes(searchText.value.toLowerCase()),
  )
})
const filteredResultLabel = computed(() => {
  if (filteredItems.value.length === 0) return 'No Result'
  return filteredItems.value.length === 1 ? 'Result' : 'Results'
})
const _highlightSearchText = (text) => {
  if (!searchText.value) return text
  const regex = new RegExp(`(${searchText.value})`, 'gi')
  return text.replace(regex, '<mark>$1</mark>')
}
// End Section Search Item In List

// Start Section Handle Status Modal
const dialogModel = computed({
  get: () => props.showDialog,
  set: value => emit('update:showDialog', value),
})

const closeModal = () => {
  emit('update:showDialog', false)
}
// End Section Handle Status Modal

const changeSelectedItem = (item) => {
  emit('changeSelectedItem', item)
}

const clickOnOverlay = () => {
  if (!mdAndUp.value) {
    emit('update:showDialog', false)
  }
}

const clickOnModal = (event) => {
  event.stopPropagation()
}
</script>

<style scoped>
.select-dialog-panel {
  max-height: min(82vh, 680px);
  overflow: hidden;
  color: #1e2a44;
  background: #fcfcfd !important;
  border: 1px solid #d8dee8;
  border-radius: 16px;
  box-shadow: 0 20px 48px rgb(30 42 68 / 16%);
}

.select-dialog-header {
  width: 100%;
  min-height: 64px;
  align-items: center;
  padding: 16px 20px;
  margin: 0 !important;
  border-bottom: 1px solid #d8dee8;
}

.select-dialog-header > .v-col {
  padding: 0;
}

.select-dialog-title {
  color: #1e2a44;
  font-size: 18px;
  font-weight: 700;
  line-height: 24px;
}

.select-dialog-close {
  box-sizing: border-box;
  display: inline-flex;
  width: 32px;
  height: 32px;
  flex: 0 0 32px;
  align-items: center;
  justify-content: center;
  margin-left: 8px !important;
  color: #667085;
  background: transparent;
  border: 1px solid transparent;
  border-radius: 8px;
  cursor: pointer;
  transition:
    color 160ms ease,
    background-color 160ms ease,
    border-color 160ms ease,
    transform 160ms ease;
}

.select-dialog-close:hover {
  color: #c93c37;
  background: #f7f8fa;
  border-color: #d8dee8;
}

.select-dialog-close:active {
  transform: scale(0.96);
}

.select-dialog-close:focus-visible {
  outline: 3px solid rgb(244 180 0 / 28%);
  outline-offset: 2px;
}

.select-dialog-result-count {
  display: inline-flex;
  align-items: baseline;
  gap: 4px;
  white-space: nowrap;
}

.select-dialog-result-count__number {
  color: #1e2a44;
  font-size: 16px;
  font-weight: 700;
  line-height: 20px;
}

.select-dialog-result-count__label {
  color: #667085;
  font-size: 12px;
  font-weight: 500;
  line-height: 18px;
}

.select-dialog-search-row {
  width: 100%;
  padding: 16px 24px 8px;
  margin: 0 !important;
}

.select-dialog-search {
  width: 100%;
}

.select-dialog-search :deep(.v-field) {
  min-height: 44px;
  color: #1e2a44;
  background: #ffffff;
  border-radius: 12px !important;
  box-shadow: none;
}

.select-dialog-search :deep(.v-field__outline) {
  color: #d8dee8;
  --v-field-border-opacity: 1;
}

.select-dialog-search :deep(.v-field.v-field--focused .v-field__outline) {
  color: #f4b400;
}

.select-dialog-search :deep(.v-field__prepend-inner),
.select-dialog-search :deep(.v-field__input),
.select-dialog-search :deep(.v-label) {
  color: #1e2a44;
}

.select-dialog-search-button {
  color: #1e2a44 !important;
  background: #f4b400 !important;
  font-size: 12px;
  font-weight: 650;
  letter-spacing: 0;
  text-transform: none;
  box-shadow: none;
}

.select-dialog-search-button:hover {
  background: #e8a800 !important;
}

.select-dialog-list {
  width: 100%;
  min-height: 0;
  padding: 8px 24px 16px;
  overflow-y: auto;
  background: transparent;
}

.select-dialog-list :deep(.select-dialog-item) {
  min-height: 48px;
  padding: 6px 12px;
  margin: 0;
  color: #1e2a44;
  background: transparent;
  border: 1px solid transparent;
  border-radius: 12px;
  transition:
    color 160ms ease,
    background-color 160ms ease,
    border-color 160ms ease,
    box-shadow 160ms ease;
}

.select-dialog-list :deep(.select-dialog-item:hover) {
  background: #f7f8fa;
  border-color: #d8dee8;
}

.select-dialog-list :deep(.select-dialog-item.v-list-item--active) {
  color: #1e2a44 !important;
  background: #eef1f5 !important;
  border-color: #d8dee8;
  box-shadow: 0 1px 2px rgb(30 42 68 / 8%);
}

.select-dialog-list :deep(.select-dialog-item:not(:last-child)) {
  border-bottom: 1px solid #eef1f5;
}

.select-dialog-list :deep(.select-dialog-item .v-list-item__overlay) {
  opacity: 0 !important;
}

.select-dialog-list :deep(.select-dialog-item .v-list-item-title) {
  overflow: hidden;
  font-size: 15px !important;
  font-weight: 500;
  line-height: 22px;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.select-dialog-list :deep(.select-dialog-item.v-list-item--active .v-list-item-title) {
  font-weight: 500;
}

.select-dialog-list :deep(.select-dialog-item--main-topic .v-list-item-title) {
  font-weight: 600;
}

.select-dialog-list :deep(.select-dialog-item--subtopic) {
  padding-left: 27px;
}

.select-dialog-list :deep(.v-list-item__prepend > .v-list-item__spacer) {
  width: 12px;
}

.select-dialog-loading {
  min-height: 128px;
  padding: 40px 20px;
}

.select-dialog-empty {
  margin: 8px 20px 20px;
  color: #1e2a44;
  background: #f7f8fa !important;
  border: 1px solid #d8dee8;
  border-radius: 12px;
}

.size-icon {
  font-size: 24px;
  margin-right: 12px;
}
.select-item-content-icon {
  display: inline-flex;
  width: 34px;
  height: 34px;
  align-items: center;
  justify-content: center;
  font-size: 34px;
  line-height: 1;
}
.select-item-icon {
  box-sizing: border-box;
  display: inline-flex;
  width: 34px;
  height: 34px;
  flex: 0 0 34px;
  align-items: center;
  justify-content: center;
}
.select-item-icon .v-img {
  width: 100%;
  height: 100%;
}

@media (prefers-reduced-motion: reduce) {
  .select-dialog-close,
  .select-dialog-list :deep(.select-dialog-item) {
    transition: none;
  }
}

@media only screen and (max-width: 960px) {
  .mobile-style {
    position: absolute;
    bottom: 0;
    max-height: 92dvh;
    border-radius: 20px 20px 0 0 !important;
  }

  .select-dialog-header {
    padding: 16px;
  }

  .select-dialog-search-row {
    padding: 16px 24px 8px;
  }

  .select-dialog-list {
    max-height: calc(92dvh - 144px) !important;
    padding-inline: 24px;
  }

  .select-dialog-empty {
    margin-inline: 16px;
  }
}
</style>
