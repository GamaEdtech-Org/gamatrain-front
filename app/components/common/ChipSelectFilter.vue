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
      <div class="inline-filter-options">
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
        :disabled="disabled"
        @click="onFilterUpdate(null)"
      >
        All
      </v-btn>
        <v-btn
          v-for="item in items"
          :key="item.id"
          variant="outlined"
          class="inline-filter-option"
          :class="{ 'inline-filter-option-selected': selectedItem?.id == item.id }"
          :disabled="disabled"
          @click="onFilterUpdate(item)"
        >
          {{ itemTitle?.(item) || item.title }}
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
  background: rgb(var(--v-theme-surface));
  border-color: rgb(var(--v-theme-grey300)) !important;
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
  color: rgb(var(--v-theme-grey500));
}

.search-filter-empty .search-filter-label {
  font-size: 16px;
  font-weight: 500;
  color: rgb(var(--v-theme-grey700));
}

.search-filter-value {
  max-width: 120px;
  overflow: hidden;
  font-size: 14px;
  font-weight: 600;
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
  background-color: #fff8e8;
  border: 1px solid #ffebb9;
}

.dependent-selected-btn {
  color: #397f7b !important;
  background-color: #f2f9f8 !important;
  border-color: #7dbfba !important;
}

.dependent-selected-btn .search-filter-label,
.dependent-selected-btn .search-filter-value,
.dependent-selected-btn .v-icon {
  color: #397f7b !important;
}

.v-btn .search-filter-clear-icon {
  color: rgb(var(--v-theme-grey500)) !important;
}

.v-btn .search-filter-clear-icon:hover {
  color: rgb(var(--v-theme-error)) !important;
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
  font-size: 20px;
  font-weight: 600;
  color: rgb(var(--v-theme-grey800));
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
  border-top: 1px solid rgb(var(--v-theme-grey300));
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
  color: rgb(var(--v-theme-grey800));
  background: rgb(var(--v-theme-surface));
  border-color: rgb(var(--v-theme-grey300)) !important;
}

.inline-filter-option-selected {
  color: white !important;
  background: #397f7b !important;
  border-color: #397f7b !important;
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
