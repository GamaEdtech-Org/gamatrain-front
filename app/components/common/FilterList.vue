<template>
  <div
    class="w-100 d-flex justify-center flex-wrap"
    :class="{ 'filter-list-sticky-host': stickyContent }"
  >
    <slot
      name="services-navigation"
      :select-service="selectService"
    />
    <v-col
      :cols="hasKeywordSearch ? `4` : `12`"
      md="12"
      class="d-flex d-md-none justify-start"
    >
      <CommonFilterTrigger
        :count="countFilterSelect"
        @click="dialogFilterMobileModel = !dialogFilterMobileModel"
      />
    </v-col>
    <v-col
      v-if="hasKeywordSearch"
      v-show="!headerSearchActive"
      cols="8"
      md="12"
      class="d-flex justify-end justify-md-center"
      :class="{ 'header-search-teleport-source': keywordSearchInHeader }"
    >
      <Teleport
        :to="headerSearchActive ? '#search-header-keyword' : null"
        :disabled="!headerSearchActive"
      >
      <v-text-field
        v-model="textSearch"
        label="Search anything...."
        variant="outlined"
        color="#F2C94C"
        max-width="330"
        density="compact"
        hide-details
        class="custom-search-text-field"
        :class="{ 'header-keyword-search': headerSearchActive }"
        @update:model-value="changeTextSearch"
      >
        <template #append>
          <v-btn
            icon
            varient="text"
            color="#F2C94C"
            width="50"
            class="rounded-ts rounded-te-xl rounded-be-xl rounded-bs h-100 ml-n2"
            flat
          >
            <v-icon
              size="x-large"
              icon="md:search"
              color="grey800"
            />
          </v-btn>
        </template>
      </v-text-field>
      </Teleport>
    </v-col>

    <div
      class="w-100 d-flex justify-center flex-wrap"
      :class="{ 'filter-list-sticky-content': stickyContent }"
    >
      <div
        ref="filterControlsShell"
        class="desktop-filter-controls-shell w-100 d-flex justify-center"
        :style="stickyMode
          ? { height: `${filterControlsHeight}px` }
          : undefined"
      >
        <div
          ref="filterControls"
          class="desktop-filter-controls w-100"
          :class="{
            'desktop-filter-controls-fixed': stickyMode,
            'desktop-filter-controls-collapsed': stickyMode && !stickyFiltersExpanded,
          }"
          :style="stickyMode
            ? { height: stickyFiltersExpanded ? `${filterControlsHeight}px` : '0px' }
            : undefined"
        >
          <div
            ref="filterControlsContent"
            class="desktop-filter-controls-content w-100 d-flex justify-center flex-wrap"
          >
            <component
              :is="filterContainer || 'div'"
              :style="filterContainer ? undefined : { display: 'contents' }"
            >
            <div
              class="w-100 d-none d-md-flex justify-center align-center flex-wrap ga-4"
              :class="{ 'mt-2': !filterContainer }"
              :style="filterContainer ? { marginTop: '16px' } : undefined"
            >
              <div class="d-flex flex-wrap w-100 max-width-container justify-start ga-2">
                <div
                  v-for="(filter, index) in filters"
                  :key="filter.title || index"
                  :style="{ display: $slots['services-navigation'] && filter.queryKey === 'type' ? 'none' : 'contents' }"
                >
                  <CommonChipSelectFilter
                    v-if="!filter.inlineOptions"
                    :ref="(el) => (filters[index].refElement = el)"
                    :title="filter.title"
                    :api="filter.api"
                    :selected-item="filter.selectedItem"
                    :extra-api-params="filter.extraApiParams"
                    :static-list="filter.staticList"
                    :item-filter="filter.itemFilter"
                    :item-transform="filter.itemTransform"
                    :item-sort="filter.itemSort"
                    :list-transform="filter.listTransform"
                    :show-item-icon="filter.showItemIcon"
                    :icon-src="filter.iconSrc"
                    :fallback-icon="filter.fallbackIcon"
                    :fallback-icon-src="filter.fallbackIconSrc"
                    :empty-fallback-icon-src="filter.emptyFallbackIconSrc"
                    :fallback-icon-padding="filter.fallbackIconPadding"
                    :boxed="filter.boxed"
                    :show-clear="Boolean(filterContainer && filter.closable && !filter.defaultValue)"
                    :selected-variant="filter.selectedVariant"
                    :control-icon="filter.controlIcon"
                    :control-icon-src="filter.controlIconSrc"
                    :control-icon-svg="filter.controlIconSvg"
                    :unselected-icon-color="filter.unselectedIconColor"
                    :control-icon-padding="filter.controlIconPadding"
                    :inline-options="filter.inlineOptions"
                    :inline-allow-clear="filter.inlineAllowClear"
                    :item-title="filter.itemTitle"
                    :disabled="filter.disabled"
                    :has-search="filter.hasSearch"
                    @update-selected-item="updateSelectedItem($event, index)"
                    @clear="clearFilter(index)"
                  />
                </div>
              </div>
              <div
                v-if="!filterContainer"
                class="justify-start d-flex w-100 max-width-container"
              >
                <div class="d-flex flex-wrap ga-2 px-2">
                  <template v-for="(filter, index) in filters">
                    <v-chip
                      v-if="filter.selectedItem && !filter.defaultValue && !filter.inlineOptions"
                      :key="filter.title"
                      variant="flat"
                      class="text-h5 pl-5 pr-5"
                      color="grey100"
                    >
                      <span class="text-grey500">{{ filter.selectedItem?.title }}</span>
                      <template #close>
                        <v-icon
                          v-if="filter.closable"
                          class="filter-clear-icon"
                          color="grey500"
                          @click="clearFilter(index)"
                        >
                          md:cancel
                        </v-icon>
                      </template>
                    </v-chip>
                  </template>
                </div>
              </div>
            </div>

            <div
              v-if="hasInlineFilters"
              class="inline-filter-group-wrapper"
            >
              <div class="inline-filter-group">
                <CommonChipSelectFilter
                  v-for="(entry, inlineIndex) in inlineFilterEntries"
                  :key="`inline-${entry.filter.title || entry.index}`"
                  :ref="(el) => (filters[entry.index].refElement = el)"
                  :title="entry.filter.title"
                  :api="entry.filter.api"
                  :selected-item="entry.filter.selectedItem"
                  :extra-api-params="entry.filter.extraApiParams"
                  :static-list="entry.filter.staticList"
                  :item-filter="entry.filter.itemFilter"
                  :inline-options="true"
                  :inline-allow-clear="entry.filter.inlineAllowClear"
                  :inline-grouped="true"
                  :inline-divider-after="inlineIndex === 0 && inlineFilterEntries.length > 1"
                  :inline-leading-option-slots="entry.filter.inlineLeadingOptionSlots"
                  :item-title="entry.filter.itemTitle"
                  :disabled="entry.filter.disabled"
                  @update-selected-item="updateSelectedItem($event, entry.index)"
                />
              </div>
            </div>
            </component>
          </div>
        </div>
      </div>

      <div
        ref="persistentContentShell"
        class="persistent-search-content-shell w-100 d-flex justify-center"
        :style="stickyMode ? { height: `${persistentContentHeight}px` } : undefined"
      >
        <div
          ref="persistentContent"
          class="persistent-search-content w-100 d-flex justify-center flex-wrap"
          :class="{ 'persistent-search-content-fixed': stickyMode }"
          :style="stickyMode
            ? { top: stickyFiltersExpanded ? `${filterControlsHeight}px` : '0px' }
            : undefined"
        >
          <div
            v-if="stickyMode"
            class="sticky-filter-trigger-row d-none d-md-flex w-100 max-width-container"
            :class="{ 'sticky-filter-trigger-row-visible': !stickyFiltersExpanded }"
          >
            <CommonFilterTrigger
              :count="countFilterSelect"
              @click="expandStickyFilters"
            />
          </div>

          <slot name="after-inline-filters" />

          <v-col
            cols="12"
            class="d-flex align-end justify-end ga-2 mt-1 py-0 px-2 max-width-container"
          >
            <slot
              name="results-heading"
              :count="countDataFound"
              :loading="loading"
            >
              <span class="text-h5 text-grey400">Result</span>
              <v-skeleton-loader
                v-if="loading"
                width="100"
                height="20"
                class="rounded-lg"
              />
              <span
                v-else
                class="text-h4 text-grey700 font-weight-bold"
              >{{
                $numberFormat(countDataFound)
              }}</span>
            </slot>
          </v-col>
        </div>
      </div>

      <v-dialog
      v-model="dialogFilterMobileModel"
      transition="dialog-bottom-transition"
      fullscreen
      scrim="#ffffff"
    >
      <div
        class="w-100 h-100 d-flex flex-column justify-space-between overflow-y-auto bg-white position-relative"
      >
        <v-container class="flex-column mb-10">
          <v-col
            cols="12"
            class="d-flex justify-space-between align-center"
          >
            <span class="text-h3 text-grey600">Filter</span>

            <v-icon
              size="x-large"
              color="grey300"
              @click="dialogFilterMobileModel = false"
            >
              md:cancel
            </v-icon>
          </v-col>
          <v-col
            cols="12"
            class="d-flex flex-wrap align-center ga-4"
          >
            <template v-for="(filter, index) in filters">
              <v-chip
                v-if="filter.selectedItem && !filter.defaultValue && !filter.inlineOptions"
                :key="filter.title"
                variant="flat"
                class="text-h5 pl-5 pr-5"
                color="grey100"
              >
                <span class="text-grey500">{{ filter.selectedItem?.title }}</span>
                <template #close>
                  <v-icon
                    v-if="filter.closable"
                    class="filter-clear-icon"
                    color="grey500"
                    @click="clearFilter(index)"
                  >
                    md:cancel
                  </v-icon>
                </template>
              </v-chip>
            </template>
          </v-col>
          <v-col
            cols="12"
            class="d-flex flex-column justify-start align-center mt-4"
          >
            <template
              v-for="(filter, index) in filters"
              :key="filter.title || index"
            >
              <div
                v-if="!filter.inlineOptions && !($slots['services-navigation'] && filter.queryKey === 'type')"
                :class="`w-100 d-flex justify-space-between align-center flex-wrap pt-2 pb-2 ${
                  filter.disabled ? `opacity-20 cursor-not-allowed` : ``
                }`"
                @click="openFilterSelectModal(filter)"
              >
              <v-badge
                :color="filter.selectedItem ? `lightError` : `#ffffff`"
                dot
                floating
              >
                <span class="text-h4 text-grey500 font-weight-bold">{{ filter.title }} </span>
              </v-badge>
              <div class="d-flex align-center ga-1">
                <v-chip
                  v-if="filter.selectedItem"
                  variant="flat"
                  class="text-h5 font-weight-bold pl-5 pr-5"
                  color="grey100"
                >
                  <span class="text-grey500">{{ filter.selectedItem?.title }}</span>
                </v-chip>

                <v-icon
                  color="grey500"
                  size="34"
                >
                  md:keyboard_arrow_down
                </v-icon>
              </div>

                <v-divider
                  :thickness="2"
                  class="border-opacity-100 mt-4 mb-4"
                  color="grey100"
                />
              </div>
            </template>
          </v-col>
        </v-container>
        <div
          class="w-100 d-flex align-center justify-center ga-3 pb-2 box-button position-fixed bottom-0 bg-white"
        >
          <v-btn
            variant="text"
            class="text-h5 text-grey700"
            @click="clearAllFilter"
          >
            Clear All
          </v-btn>
          <v-btn
            color="#F2C94C"
            rounded="xl"
            height="40"
            width="200"
            class="text-h5 text-grey800"
            :loading="loading"
            @click="dialogFilterMobileModel = false"
          >
            Show {{ $numberFormat(countDataFound) }} Results
          </v-btn>
        </div>
      </div>
      </v-dialog>
      <slot />
    </div>
  </div>
</template>

<script setup>
import { useDisplay } from 'vuetify'

const router = useRouter()
const route = useRoute()
const { mdAndUp } = useDisplay()
const searchHeaderReady = ref(false)

const props = defineProps({
  filterList: {
    type: Array,
    default: () => [],
  },
  filterContainer: {
    type: [Object, Function],
    default: null,
  },
  countDataFound: {
    type: [Number, String],
    default: () => 0,
  },
  hasKeywordSearch: {
    type: Boolean,
    default: false,
  },
  keywordSearchInHeader: {
    type: Boolean,
    default: false,
  },
  loading: {
    type: Boolean,
    default: false,
  },
  stickyContent: {
    type: Boolean,
    default: false,
  },
  desktopStickyFilters: {
    type: Boolean,
    default: false,
  },
})

const headerSearchActive = computed(() => props.keywordSearchInHeader && searchHeaderReady.value && mdAndUp.value)
const emits = defineEmits(['changeFilter'])

const filters = ref(
  props.filterList.map(filter => ({
    ...filter,
    initialDisabled: filter.disabled,
  })),
)
const dialogFilterMobileModel = ref(false)
const countFilterSelect = ref(Object.keys(route.query).length)
const textSearch = ref(route.query.title ? route.query.title : '')
const timer = ref(null)
const hasExclusiveDisabledState = ref(false)
let pendingServiceChange = false
const filterControlsShell = ref(null)
const filterControls = ref(null)
const filterControlsContent = ref(null)
const persistentContentShell = ref(null)
const persistentContent = ref(null)
const stickyMode = ref(false)
const stickyFiltersExpanded = ref(false)
const filterControlsHeight = ref(0)
const persistentContentHeight = ref(0)
let filterBoundaryObserver = null
let filterControlsResizeObserver = null
let persistentContentResizeObserver = null
let desktopMediaQuery = null
let scrollDirectionAnchor = 0

const DOWNWARD_COLLAPSE_THRESHOLD = 80
const UPWARD_BASELINE_RESET_THRESHOLD = 12

onMounted(async () => {
  searchHeaderReady.value = true
  await fetchDataRequireFilter()
  await fetchFilterAvailableInQuery()
  await nextTick()
  setupDesktopStickyBehavior()
})

onBeforeUnmount(() => {
  filterBoundaryObserver?.disconnect()
  filterControlsResizeObserver?.disconnect()
  persistentContentResizeObserver?.disconnect()
  desktopMediaQuery?.removeEventListener('change', handleDesktopBreakpointChange)
  window.removeEventListener('scroll', handleStickyScroll)
})

const updateSelectedItem = async (itemSelected, index) => {
  filters.value[index].selectedItem = itemSelected

  const isExclusiveSelected = isExclusiveFilterSelected(index)
  if (hasExclusiveDisabledState.value && !isExclusiveSelected) {
    restoreDisabledState()
    hasExclusiveDisabledState.value = false
  }

  resetDescendants(index)

  if (isExclusiveSelected) {
    disableOtherFilters(index)
    updateQueryFromFilters()
  }
  else {
    await enableReadyChildren(index)

    updateQueryFromFilters()
  }
}

const selectService = (serviceId) => {
  const index = filters.value.findIndex(filter => filter.queryKey === 'type')
  const filter = filters.value[index]
  const service = filter?.staticList?.find(item => item.id === serviceId)
  if (!service || filter.selectedItem?.id === serviceId) return

  pendingServiceChange = true
  return updateSelectedItem(service, index)
    .finally(() => {
      pendingServiceChange = false
    })
}

const isExclusiveFilterSelected = (index) => {
  const filter = filters.value[index]

  return filter?.disableOtherFiltersOnSelectedIds?.includes(
    filter.selectedItem?.id,
  )
}

const restoreDisabledState = () => {
  filters.value.forEach((filter) => {
    filter.disabled = filter.initialDisabled
  })
}

const disableOtherFilters = (sourceIndex) => {
  hasExclusiveDisabledState.value = true

  filters.value.forEach((filter, index) => {
    if (index === sourceIndex) return

    filter.selectedItem = null
    filter.disabled = true
  })
}

const resetDescendants = (indexFilter) => {
  const filterParent = filters.value[indexFilter]

  if (filterParent.childrenForGetStaticData) {
    for (const childIndex of filterParent.childrenForGetStaticData) {
      const child = filters.value[childIndex]
      const readyForGetStatic
        = child.dependenciesForGetStaticData?.includes(indexFilter)

      if (readyForGetStatic) {
        if (child.getStaticList) {
          const staticList = child.getStaticList('reset')
          child.refElement.setStaticItem(staticList)
          child.selectedItem = null
        }
      }
    }
  }

  if (!filterParent.children || filterParent.children.length == 0) return

  for (const childIndex of filterParent.children) {
    const child = filters.value[childIndex]

    child.selectedItem = null
    child.disabled = true
    resetDescendants(childIndex)
  }
}

const enableReadyChildren = async (indexFilter) => {
  const filterParent = filters.value[indexFilter]

  if (filterParent.childrenForGetStaticData) {
    for (const childIndex of filterParent.childrenForGetStaticData) {
      const child = filters.value[childIndex]
      const readyForGetStatic
        = child.dependenciesForGetStaticData?.includes(indexFilter)

      if (readyForGetStatic) {
        if (
          child.getStaticList
          && filterParent.selectedItem
          && filterParent.selectedItem.id
        ) {
          const staticList = child.getStaticList(filterParent.selectedItem.id)
          child.refElement.setStaticItem(staticList)
        }
      }
    }
  }

  if (!filterParent.children || filterParent.children.length == 0) return

  for (const childIndex of filterParent.children) {
    const child = filters.value[childIndex]

    const ready = child.dependencies.every(
      dep => !!filters.value[dep.parent].selectedItem,
    )

    if (ready) {
      const disableValue = child.dependencies.some(dep =>
        dep.disableIds?.includes(filters.value[dep.parent].selectedItem.id),
      )

      if (disableValue) {
        child.disabled = true
        continue
      }

      if (
        child.queryMap
        && child.parentIndexChangeQueryKey
        && filters.value[child.parentIndexChangeQueryKey].selectedItem
      ) {
        const id
          = filters.value[child.parentIndexChangeQueryKey].selectedItem.id
        child.queryKey = child.queryMap[id] ?? child.queryKey
      }

      child.disabled = false
      if (child.api && !child.staticList?.length) {
        if (!child.idInParams) {
          child.dependencies.forEach((dep) => {
            const parentNode = filters.value[dep.parent]
            child.extraApiParams[dep.targetKey]
              = parentNode.selectedItem?.[dep.sourceKey] ?? null
          })
        }
        await child.refElement?.getItems(
          child.idInParams ? filterParent.selectedItem.id : '',
        )
      }

      enableReadyChildren(childIndex)
    }
  }
}

const clearFilter = (index) => {
  filters.value[index].selectedItem = null

  if (hasExclusiveDisabledState.value) {
    restoreDisabledState()
    hasExclusiveDisabledState.value = false
  }

  resetDescendants(index)

  updateQueryFromFilters()
}

const updateQueryFromFilters = async () => {
  const query = {}
  const titles = {}

  filters.value.forEach((f) => {
    // Due to the update to version 2 of the backend for the board, this f.title != 'Board' has been placed.
    // if (f.queryKey && f.selectedItem?.code && f.title != 'Board') {
    if (f.queryKey && f.selectedItem?.code) {
      query[f.queryKey] = f.selectedItem.code
      titles[f.queryKey] = f.selectedItem.title
    }
    else if (f.queryKey && f.selectedItem?.id) {
      query[f.queryKey] = f.selectedItem.id
      titles[f.queryKey] = f.selectedItem.title
    }
  })

  countFilterSelect.value = Object.keys(query).length
  router.replace({ query })
  emits('changeFilter', query, titles, { serviceChange: pendingServiceChange })
}

const fetchDataRequireFilter = async () => {
  for (let i = 0; i < filters.value.length; i++) {
    const filter = filters.value[i]
    if (!filter.dependencies?.length) {
      if (filter.api && !filter.staticList?.length) {
        await filter.refElement.getItems()
      }
    }
    if (filter.getStaticList) {
      const staticList = filter.getStaticList()
      filter.refElement.setStaticItem(staticList)
    }
  }
}

const fetchFilterAvailableInQuery = async () => {
  for (let index = 0; index < filters.value.length; index++) {
    const filter = filters.value[index]
    const qVal = route.query[filter.queryKey]
    // Due to the update to version 2 of the backend for the board, change filter key to id.
    // old code
    const filterKey = filter.queryKey == 'section' ? 'code' : 'id'
    // const filterKey = 'id'

    if (filter.defaultValue && !qVal) {
      filters.value[index].selectedItem = filter.defaultValue
      await enableReadyChildren(index)

      const query = { ...route.query }
      query[filter.queryKey] = filter.defaultValue.id
      router.replace({ query })
    }
    if (!qVal) continue

    const ready = filter.dependencies?.every(
      dep => filters.value[dep.parent].selectedItem,
    )

    if (!ready && filter.dependencies?.length) continue

    if (filter.staticList?.length) {
      const selected = filter.staticList.find(
        x => String(x[filterKey]) === String(qVal),
      )
      filters.value[index].selectedItem = selected
      if (isExclusiveFilterSelected(index)) {
        disableOtherFilters(index)
        continue
      }
      await enableReadyChildren(index)
    }
    else {
      const selected = await filter.refElement?.getItemById(qVal, filterKey)
      if (selected) {
        filters.value[index].selectedItem = selected
        if (isExclusiveFilterSelected(index)) {
          disableOtherFilters(index)
          continue
        }
        await enableReadyChildren(index)
      }
    }
  }
}

const openFilterSelectModal = (filter) => {
  if (filter.disabled) return

  filter.refElement.openSelectModal()
}

const measureStickyContent = () => {
  if (filterControlsContent.value) {
    filterControlsHeight.value = filterControlsContent.value.offsetHeight
  }
  if (persistentContent.value && !stickyMode.value) {
    persistentContentHeight.value = persistentContent.value.getBoundingClientRect().height
  }
}

const resetStickyMode = () => {
  stickyMode.value = false
  stickyFiltersExpanded.value = false
  scrollDirectionAnchor = window.scrollY
}

const handleDesktopBreakpointChange = (event) => {
  filterBoundaryObserver?.disconnect()
  filterBoundaryObserver = null

  if (!event.matches) {
    resetStickyMode()
    return
  }

  nextTick(() => {
    measureStickyContent()
    observeFilterBoundary()
  })
}

const setupDesktopStickyBehavior = () => {
  if (!props.desktopStickyFilters || !import.meta.client) return

  desktopMediaQuery = window.matchMedia('(min-width: 960px)')
  desktopMediaQuery.addEventListener('change', handleDesktopBreakpointChange)
  window.addEventListener('scroll', handleStickyScroll, { passive: true })

  filterControlsResizeObserver = new ResizeObserver(() => {
    if (filterControlsContent.value) {
      filterControlsHeight.value = filterControlsContent.value.offsetHeight
    }
  })
  persistentContentResizeObserver = new ResizeObserver(() => {
    if (persistentContent.value && !stickyMode.value) {
      persistentContentHeight.value = persistentContent.value.getBoundingClientRect().height
    }
  })

  if (filterControlsContent.value) filterControlsResizeObserver.observe(filterControlsContent.value)
  if (persistentContent.value) persistentContentResizeObserver.observe(persistentContent.value)

  measureStickyContent()
  scrollDirectionAnchor = window.scrollY
  if (desktopMediaQuery.matches) observeFilterBoundary()
}

const observeFilterBoundary = () => {
  const target = filterControlsShell.value
  if (!target) return

  filterBoundaryObserver = new IntersectionObserver(([entry]) => {
    const hasPassedAboveViewport
      = !entry.isIntersecting && entry.boundingClientRect.bottom <= 0

    if (hasPassedAboveViewport && !stickyMode.value) {
      stickyFiltersExpanded.value = true
      stickyMode.value = true
      scrollDirectionAnchor = window.scrollY

      requestAnimationFrame(() => {
        requestAnimationFrame(() => {
          if (stickyMode.value) stickyFiltersExpanded.value = false
        })
      })
    }
    else if (!hasPassedAboveViewport) {
      resetStickyMode()
    }
  }, { threshold: 0 })

  filterBoundaryObserver.observe(target)
}

const expandStickyFilters = async () => {
  measureStickyContent()
  stickyFiltersExpanded.value = true
  scrollDirectionAnchor = window.scrollY
  await nextTick()
  measureStickyContent()
}

const handleStickyScroll = () => {
  if (!stickyMode.value || !stickyFiltersExpanded.value) {
    scrollDirectionAnchor = window.scrollY
    return
  }

  const scrollDelta = window.scrollY - scrollDirectionAnchor
  if (scrollDelta >= DOWNWARD_COLLAPSE_THRESHOLD) {
    stickyFiltersExpanded.value = false
    scrollDirectionAnchor = window.scrollY
  }
  else if (scrollDelta <= -UPWARD_BASELINE_RESET_THRESHOLD) {
    scrollDirectionAnchor = window.scrollY
  }
}

watch(
  () => route.query,
  async (query) => {
    for (const filter of filters.value) {
      if (!filter.inlineOptions || !filter.queryKey) continue

      const queryValue = query[filter.queryKey]
      if (!queryValue) {
        filter.selectedItem = null
        continue
      }

      if (String(filter.selectedItem?.id) === String(queryValue)) continue

      const selected = await filter.refElement?.getItemById(queryValue, 'id')
      if (selected) filter.selectedItem = selected
    }
  },
  { deep: true },
)
const inlineFilterEntries = computed(() =>
  filters.value
    .map((filter, index) => ({ filter, index }))
    .filter(entry => entry.filter.inlineOptions),
)
const hasInlineFilters = computed(() => inlineFilterEntries.value.length > 0)

const changeTextSearch = () => {
  if (props.hasKeywordSearch) {
    const query = { ...route.query }
    if (textSearch.value.length == 0) {
      delete query.title
    }
    else {
      query.title = textSearch.value
    }
    router.replace({ query })
    debouncedSearchText()
  }
}

const debouncedSearchText = () => {
  if (timer.value) {
    clearTimeout(timer.value)
    timer.value = null
  }
  timer.value = setTimeout(() => {
    emits('changeFilter', route.query)
  }, 800)
}

const clearAllFilter = async () => {
  for (let i = 0; i < filters.value.length; i++) {
    const filter = filters.value[i]
    if (
      !filter.dependencies?.length
      && filter.selectedItem
      && !filter.defaultValue
    ) {
      filter.selectedItem = null
      resetDescendants(i)
    }
  }
  updateQueryFromFilters()
}
</script>

<style scoped>
.header-keyword-search {
  width: 100%;
  min-width: 0;
}

.header-keyword-search :deep(.v-field) {
  color: #202238;
  background: #ffffff;
  border-radius: 12px;
}

.header-keyword-search :deep(.v-field__outline) {
  color: #8c959f;
}

.header-keyword-search :deep(.v-field--focused .v-field__outline) {
  color: #f2c94c;
}

@media (min-width: 960px) {
  .header-search-teleport-source {
    display: none !important;
  }
}

.filter-clear-icon:hover {
  color: #c93c37 !important;
}

:deep(.height-badge .v-badge__wrapper .v-badge__badge) {
  height: 20px !important;
}

:deep(.custom-search-text-field .v-field__outline__start) {
  border-radius: 24px 0 0 24px !important;
  flex: 0 0 30px !important;
}
:deep(.custom-search-text-field .v-field__outline__end) {
  border-radius: 0 4px 4px 0 !important;
}
.max-width-container {
  max-width: 1200px;
}
.filter-list-sticky-content {
  position: sticky;
  top: 0;
  z-index: 2;
  background: #f7f7f4;
}
.filter-list-sticky-host {
  display: contents !important;
}
.desktop-filter-controls,
.persistent-search-content {
  background: #f7f7f4;
}
.desktop-filter-controls-content {
  transform-origin: top center;
}
.sticky-filter-trigger-row {
  box-sizing: border-box;
  height: 0;
  overflow: hidden;
  padding: 0;
  opacity: 0;
  transform: scale(0.96);
  transform-origin: top left;
}
.inline-filter-group-wrapper {
  display: flex;
  width: 100%;
  max-width: 1200px;
  justify-content: flex-start;
}
.inline-filter-group {
  display: flex;
  width: fit-content;
  max-width: 1200px;
  flex-direction: column;
  align-items: flex-start;
  padding: 16px 24px 0 0;
  margin-right: auto;
  margin-top: 16px;
  background: #ffffff;
  border: 1px solid #dcdde5;
  border-radius: 12px;
  box-shadow: 0 1px 2px rgb(36 41 47 / 8%);
}

@media (min-width: 960px) {
  .desktop-filter-controls-fixed,
  .persistent-search-content-fixed {
    position: fixed;
    z-index: 11;
    top: 0;
    left: 0;
  }

  .desktop-filter-controls-fixed {
    z-index: 12;
    overflow: hidden;
    transition: height 250ms cubic-bezier(0.16, 1, 0.3, 1);
  }

  .desktop-filter-controls-fixed .desktop-filter-controls-content {
    transition:
      opacity 250ms cubic-bezier(0.16, 1, 0.3, 1),
      transform 250ms cubic-bezier(0.16, 1, 0.3, 1);
  }

  .desktop-filter-controls-collapsed .desktop-filter-controls-content {
    pointer-events: none;
    opacity: 0;
    transform: translateY(-8px) scale(0.98);
  }

  .persistent-search-content-fixed {
    transition: top 250ms cubic-bezier(0.16, 1, 0.3, 1);
  }

  .sticky-filter-trigger-row {
    transition:
      height 250ms cubic-bezier(0.16, 1, 0.3, 1),
      padding 250ms cubic-bezier(0.16, 1, 0.3, 1),
      opacity 200ms ease-out,
      transform 250ms cubic-bezier(0.16, 1, 0.3, 1);
  }

  .sticky-filter-trigger-row-visible {
    height: 56px;
    padding: 8px 0;
    opacity: 1;
    transform: scale(1);
  }
}
</style>
