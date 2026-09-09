<template>
  <div
    v-if="inlineOptions"
    class="inline-filter-selector"
    :class="{
      'inline-filter-disabled': disabled,
      'inline-filter-grouped-row': inlineGrouped,
    }"
  >
    <div class="inline-filter-row-content">
      <span class="inline-filter-label">{{ title }}</span>
      <div
        class="inline-filter-options"
        :style="inlineGrouped
          ? { gridTemplateColumns: `max-content repeat(${inlineItemsPerRow}, max-content)` }
          : undefined"
      >
        <span
          v-for="slot in inlineLeadingOptionSlots"
          :key="`inline-option-spacer-${slot}`"
          class="inline-filter-option-spacer"
          aria-hidden="true"
        />
      <v-btn
        v-if="inlineAllowClear"
        variant="outlined"
        class="inline-filter-option"
        :class="{ 'inline-filter-option-selected': !selectedItem }"
        :style="inlineGrouped ? { gridColumn: 1, gridRow: 1 } : undefined"
        :disabled="disabled"
        @click="onFilterUpdate(null)"
      >
        All
      </v-btn>
        <v-btn
          v-for="(item, itemIndex) in items"
          :key="item.id"
          variant="outlined"
          class="inline-filter-option"
          :class="{
            'inline-filter-option-selected': selectedItem?.id == item.id,
            'inline-filter-option-multi-digit': isMultiDigitPaperOption(item),
          }"
          :style="inlineGrouped
            ? {
              gridColumn: (itemIndex % inlineItemsPerRow) + (inlineAllowClear ? 2 : 1),
              gridRow: Math.floor(itemIndex / inlineItemsPerRow) + 1,
            }
            : undefined"
          :disabled="disabled"
          @click="onFilterUpdate(item)"
        >
          {{ getInlineItemTitle(item) }}
        </v-btn>
      </div>
    </div>
    <div
      v-if="inlineDividerAfter"
      class="inline-filter-divider"
    />
  </div>
  <v-btn
    v-else
    :class="[
      'text-h5',
      { 'open-style-btn': isShowSelectModal || (selectedItem && selectedVariant !== 'dependent-green') },
      { 'dependent-selected-btn': selectedItem && selectedVariant === 'dependent-green' },
      { 'search-filter-empty': !selectedItem },
      { 'search-filter-control': boxed },
      { 'search-filter-has-icon': showItemIcon || controlIcon || controlIconSrc || controlIconSvg },
    ]"
    variant="outlined"
    :rounded="boxed ? 'lg' : 'xl'"
    value="board"
    color="grey200"
    :disabled="disabled"
    :loading="loading"
    @click="isShowSelectModal = !isShowSelectModal"
  >
    <span
      v-if="showItemIcon || controlIcon || controlIconSrc || controlIconSvg"
      class="search-filter-icon mr-2"
      :class="{ 'search-filter-icon-padded': controlIconPadding }"
      :style="[
        controlIconPadding
          ? { padding: `${controlIconPadding}px` }
          : selectedItem?.iconPadding
            ? { padding: `${selectedItem.iconPadding}px` }
            : undefined,
        !selectedItem && unselectedIconColor ? { color: unselectedIconColor } : undefined,
      ]"
    >
      <v-img
        v-if="showItemIcon && selectedItem?.icon && !selectedIconFailed"
        :src="getIconSrc(selectedItem)"
        :alt="selectedItem.title"
        contain
        @error="selectedIconFailed = true"
      />
      <span
        v-else-if="showItemIcon && selectedItem?.contentIcon"
        :class="`${selectedItem.contentIcon} search-filter-content-icon`"
        :style="{ color: selectedItem.color }"
      />
      <span
        v-else-if="controlIconSvg"
        class="search-filter-inline-svg-icon"
      >
        <svg
          width="28"
          height="28"
          :viewBox="controlIconSvg.viewBox"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            v-for="path in controlIconSvg.paths"
            :key="path"
            :d="path"
            fill="currentColor"
            stroke="currentColor"
            :stroke-width="controlIconSvg.strokeWidth"
            stroke-linejoin="round"
            paint-order="stroke fill"
          />
        </svg>
      </span>
      <span
        v-else-if="controlIconSrc"
        class="search-filter-svg-icon"
        :style="{
          maskImage: `url(${controlIconSrc})`,
          WebkitMaskImage: `url(${controlIconSrc})`,
        }"
      />
      <img
        v-else-if="emptyFallbackIconSrc || fallbackIconSrc"
        :src="!selectedItem && emptyFallbackIconSrc ? emptyFallbackIconSrc : fallbackIconSrc"
        alt=""
        class="search-filter-fallback-image"
        :style="fallbackIconPadding
          ? {
            width: `${28 - (fallbackIconPadding * 2)}px`,
            height: `${28 - (fallbackIconPadding * 2)}px`,
          }
          : undefined"
      />
      <v-icon
        v-else
        size="28"
      >{{ controlIcon || fallbackIcon }}</v-icon>
    </span>
    <span
      v-if="boxed"
      class="search-filter-copy text-grey700"
    >
      <span class="search-filter-label">{{ title }}</span>
      <span
        v-if="selectedItem"
        class="search-filter-value"
      >{{ selectedItem.title }}</span>
    </span>
    <span
      v-else
      class="text-grey700"
    >{{ selectedItem ? selectedItem.title : title }}</span>
    <template #append>
      <v-icon
        v-if="showClear && selectedItem"
        class="search-filter-clear-icon mr-1"
        color="grey500"
        size="18"
        role="button"
        tabindex="0"
        :aria-label="`Clear ${title}`"
        @click.stop="emit('clear')"
        @keydown.enter.stop.prevent="emit('clear')"
        @keydown.space.stop.prevent="emit('clear')"
      >
        md:cancel
      </v-icon>
      <v-icon
        class="mt-1"
        color="grey500"
      >
        md:keyboard_arrow_down
      </v-icon>
    </template>
  </v-btn>

  <search-select-dialog
    v-model:show-dialog="isShowSelectModal"
    :title-modal="title"
    :items="items"
    :selected-item="selectedItem"
    :has-search="hasSearch"
    :compact-result-count="boxed"
    :show-item-icon="showItemIcon"
    :icon-src="iconSrc"
    :fallback-icon="fallbackIcon"
    :fallback-icon-src="fallbackIconSrc"
    @change-selected-item="onFilterUpdate"
  />
</template>

<script setup>
const props = defineProps({
  title: {
    type: String,
    default: '',
  },
  hasSearch: {
    type: Boolean,
    default: true,
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  api: {
    type: [String, null],
    required: true,
  },
  pageFilterSkip: {
    type: Number,
    default: 0,
  },
  pageFilterSize: {
    type: Number,
    default: 1000,
  },
  returnTotalRecordsCount: {
    type: Boolean,
    default: true,
  },
  extraApiParams: {
    type: Object,
    default: () => {},
  },
  selectedItem: {
    type: Object,
    default: () => {},
  },
  staticList: {
    type: Array,
    default: () => [],
  },
  itemFilter: {
    type: Function,
    default: null,
  },
  itemTransform: {
    type: Function,
    default: null,
  },
  itemSort: {
    type: Function,
    default: null,
  },
  listTransform: {
    type: Function,
    default: null,
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
  emptyFallbackIconSrc: {
    type: String,
    default: '',
  },
  fallbackIconPadding: {
    type: Number,
    default: 0,
  },
  boxed: {
    type: Boolean,
    default: false,
  },
  showClear: {
    type: Boolean,
    default: false,
  },
  selectedVariant: {
    type: String,
    default: '',
  },
  controlIcon: {
    type: String,
    default: '',
  },
  controlIconSrc: {
    type: String,
    default: '',
  },
  controlIconSvg: {
    type: Object,
    default: null,
  },
  unselectedIconColor: {
    type: String,
    default: '',
  },
  controlIconPadding: {
    type: Number,
    default: 0,
  },
  inlineOptions: {
    type: Boolean,
    default: false,
  },
  inlineAllowClear: {
    type: Boolean,
    default: false,
  },
  inlineGrouped: {
    type: Boolean,
    default: false,
  },
  inlineItemsPerRow: {
    type: Number,
    default: 3,
  },
  inlineDividerAfter: {
    type: Boolean,
    default: false,
  },
  inlineLeadingOptionSlots: {
    type: Number,
    default: 0,
  },
  itemTitle: {
    type: Function,
    default: null,
  },
})

const emit = defineEmits(['UpdateSelectedItem', 'clear'])

const items = ref([...props.staticList])
const isShowSelectModal = ref(false)
const loading = ref(false)
const selectedIconFailed = ref(false)

watch(
  () => props.selectedItem?.icon,
  () => {
    selectedIconFailed.value = false
  },
)

const getIconSrc = item => props.iconSrc?.(item) || item.icon
const getInlineItemTitle = item => props.itemTitle?.(item) || item.title
const isMultiDigitPaperOption = item =>
  props.title === 'Paper'
  && /^\d{2,}$/.test(String(getInlineItemTitle(item)).trim())

const onFilterUpdate = (itemSelected) => {
  isShowSelectModal.value = false
  emit('UpdateSelectedItem', itemSelected)
}

const getItems = async (extraIdParam = '') => {
  try {
    loading.value = true
    if (props.api) {
      items.value = []
      const url
        = extraIdParam.toString().length > 0
          ? props.api + '/' + extraIdParam
          : props.api
      const params = {
        ...props.extraApiParams,
      }
      if (props.title == 'School' || props.title == 'Country' || props.title == 'State' || props.title == 'City') {
        params['PagingDto.PageFilter.Skip'] = props.pageFilterSkip
        params['PagingDto.PageFilter.Size'] = props.pageFilterSize
        params['PagingDto.PageFilter.ReturnTotalRecordsCount'] = props.returnTotalRecordsCount
      }

      const response = await useApiService.get(url, params, { public: true })

      if (response.succeeded || response.status == 1) {
        const responseList = response.data.list || response.data
        let transformedList = props.itemTransform
          ? responseList.map(props.itemTransform)
          : responseList
        if (props.listTransform) {
          transformedList = await props.listTransform(transformedList)
        }
        const filteredList = props.itemFilter
          ? transformedList.filter(props.itemFilter)
          : transformedList
        const list = props.itemSort
          ? [...filteredList].sort(props.itemSort)
          : filteredList
        if (props.title == 'School') {
          if (list && list.length > 0) {
            items.value = list.map(s => ({
              title: s.name,
              id: s.id,
            }))
          }
        }
        else {
          items.value = list
        }
      }
    }
  }
  catch (error) {
    console.log('error', error)
  }
  finally {
    loading.value = false
  }
}

const getItemById = (id, filterKey) => {
  if (!id) return null

  const searchField = filterKey === 'code' ? 'code' : 'id'

  return items.value.find(item =>
    String(item[searchField]) === String(id),
  ) || null
}

const openSelectModal = () => {
  isShowSelectModal.value = true
}

const setStaticItem = (staticItem) => {
  items.value = staticItem
}

defineExpose({
  getItems,
  getItemById,
  openSelectModal,
  setStaticItem,
})
</script>

<style>
.search-filter-control {
  min-width: 152px;
  height: 52px !important;
  justify-content: space-between;
  padding-inline: 16px;
  color: #1e2a44;
  background: #fcfcfd;
  border-color: #d8dee8 !important;
  border-radius: 12px !important;
  transition: background-color 160ms ease, border-color 160ms ease, box-shadow 160ms ease;
}

.search-filter-control:hover {
  background: #f7f8fa;
  border-color: #e8a800 !important;
}

.search-filter-control:focus-visible {
  box-shadow: 0 0 0 3px rgb(244 180 0 / 28%);
}

.search-filter-icon {
  box-sizing: border-box;
  display: inline-flex;
  width: 28px;
  height: 28px;
  flex: 0 0 auto;
  align-items: center;
  justify-content: center;
}

.search-filter-icon .v-img {
  width: 100%;
  height: 100%;
}

.search-filter-fallback-image {
  display: block;
  object-fit: contain;
}

.search-filter-content-icon {
  display: inline-flex;
  width: 28px;
  height: 28px;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  line-height: 1;
}

.search-filter-svg-icon {
  width: 100%;
  height: 100%;
  background-color: currentColor;
  mask-position: center;
  mask-repeat: no-repeat;
  mask-size: contain;
}

.search-filter-inline-svg-icon {
  display: block;
  width: 100%;
  height: 100%;
  background: transparent;
}

.search-filter-inline-svg-icon svg {
  display: block;
  width: 100%;
  height: 100%;
}

.search-filter-icon-padded .v-icon {
  width: 100%;
  height: 100%;
  min-width: 0;
  font-size: 20px !important;
  -webkit-text-stroke: 0.6px currentColor;
  paint-order: stroke fill;
}

.search-filter-copy {
  display: flex;
  min-width: 0;
  flex-direction: column;
  align-items: flex-start;
  line-height: 1.2;
}

.search-filter-label {
  font-size: 12px;
  font-weight: 500;
  color: rgb(30 42 68 / 68%);
}

.search-filter-empty .search-filter-label {
  font-size: 16px;
  font-weight: 600;
  color: #1e2a44;
}

.search-filter-value {
  max-width: 120px;
  overflow: hidden;
  font-size: 14px;
  font-weight: 650;
  color: #1e2a44;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.search-filter-control .v-btn__content {
  flex: 1;
  justify-content: flex-start;
  overflow: hidden;
}

.search-filter-control .v-btn__content > span {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.open-style-btn {
  background-color: #fcfcfd;
  border: 1px solid #1e2a44;
}

.open-style-btn:not(.search-filter-empty) {
  color: #1e2a44 !important;
  background-color: #d8dee8 !important;
  border-color: transparent !important;
}

.open-style-btn:not(.search-filter-empty) .search-filter-label {
  color: #1e2a44 !important;
}

.open-style-btn:not(.search-filter-empty) .search-filter-value,
.open-style-btn:not(.search-filter-empty) .v-icon,
.open-style-btn:not(.search-filter-empty) .search-filter-content-icon {
  color: #1e2a44 !important;
}

.dependent-selected-btn {
  color: #1e2a44 !important;
  background-color: #d8dee8 !important;
  border-color: transparent !important;
}

.dependent-selected-btn .search-filter-label {
  color: #1e2a44 !important;
}

.dependent-selected-btn .search-filter-value,
.dependent-selected-btn .v-icon,
.dependent-selected-btn .search-filter-content-icon {
  color: #1e2a44 !important;
}

.v-btn .search-filter-clear-icon {
  color: #667085 !important;
}

.open-style-btn:not(.search-filter-empty) .search-filter-clear-icon,
.dependent-selected-btn .search-filter-clear-icon {
  color: #667085 !important;
}

.v-btn .search-filter-clear-icon:hover {
  color: #c93c37 !important;
}

.v-btn:has(.search-filter-clear-icon:hover) {
  --v-hover-opacity: 0;
}

.inline-filter-selector {
  --inline-filter-option-width: 72px;
  --inline-filter-option-height: 39px;

  display: flex;
  width: 100%;
  max-width: 1200px;
  min-height: 84px;
  flex: 1 0 100%;
  flex-wrap: wrap;
  align-items: center;
  gap: 0;
  padding: 16px 24px;
  margin-top: 16px;
  background: rgb(var(--v-theme-surface));
  border: 1px solid rgb(var(--v-theme-grey300));
  border-radius: 16px;
}

.inline-filter-disabled {
  opacity: var(--v-disabled-opacity);
}

.inline-filter-label {
  margin-right: 72px;
  font-size: 16px;
  font-weight: 650;
  color: #1e2a44;
}

.inline-filter-options {
  display: flex;
  flex: 1 1 0;
  align-items: center;
  min-width: 0;
  flex-wrap: wrap;
  gap: 12px;
}

.inline-filter-row-content {
  display: contents;
}

.inline-filter-grouped-row {
  display: block;
  width: max-content;
  max-width: 100%;
  min-height: 0;
  flex: none;
  padding: 0;
  margin: 0;
  border: 0;
  border-radius: 0;
}

.inline-filter-grouped-row .inline-filter-row-content {
  display: grid;
  grid-template-columns: 75px max-content;
  align-items: center;
  column-gap: 72px;
}

.inline-filter-grouped-row .inline-filter-label {
  margin-right: 0;
}

.inline-filter-grouped-row .inline-filter-options {
  flex: none;
  gap: 8px;
}

.inline-filter-divider {
  width: 100%;
  margin: 12px 0;
  border-top: 1px solid #d8dee8;
}

.inline-filter-option {
  width: auto;
  min-width: 0;
  max-width: none;
  height: auto !important;
  padding: 8px 12px !important;
  border-radius: 12px !important;
  font-size: 14px;
  line-height: 20px;
  color: #1e2a44;
  background: #fcfcfd;
  border-color: #d8dee8 !important;
  transition: background-color 160ms ease, border-color 160ms ease, box-shadow 160ms ease;
}

.inline-filter-option-multi-digit {
  padding-inline: 8px !important;
}

.inline-filter-option-selected {
  color: #ffffff !important;
  background: #1e2a44 !important;
  border-color: #1e2a44 !important;
  box-shadow: 0 1px 2px rgb(30 42 68 / 16%);
}

@media only screen and (max-width: 600px) {
  .inline-filter-selector {
    align-items: flex-start;
    gap: 12px;
    padding: 12px 16px;
  }

  .inline-filter-label {
    flex-basis: 100%;
    margin-right: 0;
  }

  .inline-filter-options {
    gap: 12px;
  }

}

.inline-filter-option-spacer {
  width: var(--inline-filter-option-width);
  min-width: var(--inline-filter-option-width);
  height: var(--inline-filter-option-height);
  flex: 0 0 var(--inline-filter-option-width);
}
</style>
