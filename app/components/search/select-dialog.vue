<template>
  <v-dialog
    v-model="dialogModel"
    max-width="400"
    :fullscreen="!mdAndUp"
    @click="clickOnOverlay"
  >
    <div
      class="select-dialog-panel w-100 d-flex flex-wrap flex-column bg-white pa-6 mobile-style"
      @click="clickOnModal"
    >
      <v-row>
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
            size="x-large"
            @click="closeModal"
          >
            md:cancel
          </v-icon>
        </v-col>
      </v-row>
      <v-row
        v-if="hasSearch"
        class="pl-2 pr-2 mt-6"
      >
        <v-text-field
          v-model="searchText"
          :label="`Search ${titleModal}`"
          prepend-inner-icon="md:search"
          glow
          variant="outlined"
          color="#F2C94C"
          icon-color="#171633"
          density="compact"
          rounded="xl"
        >
          <template #append-inner>
            <v-btn
              class="mr-n3"
              color="#F2C94C"
              width="70"
              height="30"
              rounded="xl"
            >
              Search
            </v-btn>
          </template>
        </v-text-field>
      </v-row>
      <v-list
        v-if="!isLoading"
        max-height="320"
      >
        <v-list-item
          v-for="item in filteredItems"
          :key="item.title"
          :value="item.title"
          :active="item.id == selectedItem?.id"
          color="#F2C94C"
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
                color="grey500"
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
        class="text-center pt-8"
      >
        <v-progress-circular
          indeterminate
          :width="3"
          color="#F2C94C"
        />
      </div>

      <v-alert
        v-if="searchText && filteredItems.length === 0 && !isLoading"
        type="info"
        color="#F2C94C"
        density="compact"
        class="mb-4"
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
        color="#F2C94C"
        density="compact"
        class="mb-4"
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
                ? 'background-color: #FFB600; color: white;'
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
  color: #202238;
  border: 1px solid #dcdde5;
  border-radius: 16px;
  box-shadow: 0 12px 32px rgb(36 41 47 / 18%);
}

.select-dialog-title {
  color: #202238;
  font-size: 20px;
  font-weight: 700;
  line-height: 28px;
}

.select-dialog-close {
  color: #65697a;
  cursor: pointer;
}

.select-dialog-close:hover {
  color: #c93c37;
}

.select-dialog-result-count {
  display: inline-flex;
  align-items: baseline;
  gap: 4px;
  white-space: nowrap;
}

.select-dialog-result-count__number {
  color: #202238;
  font-size: 18px;
  font-weight: 700;
  line-height: 24px;
}

.select-dialog-result-count__label {
  color: #65697a;
  font-size: 13px;
  font-weight: 500;
  line-height: 20px;
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
@media only screen and (max-width: 960px) {
  .mobile-style {
    position: absolute;
    bottom: 0;
    border-radius: 24px 24px 0 0 !important;
  }
}
</style>
