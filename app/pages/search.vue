<template>
  <v-container class="flex-column margin-top-handle">
    <v-row class="justify-center">
      <div class="w-100 d-flex justify-center flex-wrap top-info-div">
        <CommonFilterList
          :key="activeCategory"
          :filter-list="filters"
          :filter-container="CategoryFilterContainer"
          :style="{ '--search-category-color': activeCategoryColor }"
          :count-data-found="totalDataFind"
          :loading="isInitialDataLoading"
          has-keyword-search
          keyword-search-in-header
          desktop-sticky-filters
          @change-filter="changeFilter"
        >
          <template #category-navigation="{ selectCategory }">
            <SearchCategoryTabs
              :active-category="activeCategory"
              @change="selectCategory"
            />
          </template>
          <template #after-inline-filters>
            <div class="w-100 d-flex align-start justify-start max-width-container">
              <CommonDetailSubjectDirectoryNav :content-data="data[0]" />
            </div>
          </template>
          <template #results-heading="{ count, loading: headingLoading }">
            <div class="search-results-heading w-100 d-flex flex-wrap align-center justify-space-between ga-4">
              <h1 class="search-results-title">
              {{ metadata.title }}
              </h1>
              <div class="search-results-count d-flex align-center ga-2 flex-shrink-0">
                <v-skeleton-loader
                  v-if="headingLoading"
                  width="100"
                  height="24"
                  class="rounded-lg"
                />
                <template v-else>
                  <span class="search-results-count-number">{{ $numberFormat(count) }}</span>
                  <span class="search-results-count-label">Results</span>
                </template>
              </div>
            </div>
          </template>
        </CommonFilterList>
      </div>

      <search-list
        v-if="data && data.length > 0"
        :data-list="data"
        :is-initial-loading="isInitialDataLoading"
        :is-pagination-loading="isPaginationDataLoading"
        :is-all-data-loaded="isAllDataLoaded"
        :is-previous-loading="isPreviousLoading"
        :first-loaded-page-number="firstLoadedPageNumber"
        :is-profile-mode="route.query.type == 'teacher'"
        @load-next-page="loadNextPageData"
        @load-previous-page="loadPreviousPageData"
      />

      <div
        v-else
        class="w-100 d-flex flex-column align-center justify-center ga-4 mt-16"
      >
        <span class="text-h4 font-weight-bold">Be the first to add content to this category.</span>
        <v-btn
          class="text-h5 font-weight-bold"
          width="250"
          color="info"
          rounded="pill"
          flat
          variant="tonal"
          @click="createLinkAddConent()"
        >
          <v-icon color="info">
            md:add
          </v-icon>
          Add
        </v-btn>
      </div>
    </v-row>
  </v-container>
</template>

<script setup>
import CategoryFilterContainer from '~/components/search/CategoryFilterContainer.vue'
import dayjs from 'dayjs'
import { useRoute } from 'vue-router'

const route = useRoute()
const router = useRouter()

const getEquivalentNewType = (type) => {
  switch (type) {
    case 'test':
      return 'paper'
    case 'learnfiles':
      return 'multimedia'
    case 'azmoon':
      return 'quizhub'
    case 'question':
      return 'forum'
    case 'dars':
      return 'tutorial'
    case 'paper':
      return 'paper'
    case 'study-materials':
      return 'study-materials'
    case 'multimedia':
      return 'multimedia'
    case 'quizhub':
      return 'quizhub'
    case 'forum':
      return 'forum'
    case 'tutorial':
      return 'tutorial'
    case 'teacher':
      return 'teacher'
    default:
      return 'paper'
  }
}
const getEquivalentOldType = (type) => {
  switch (type) {
    case 'paper':
      return 'test'
    case 'study-materials':
      return 'test'
    case 'multimedia':
      return 'learnfiles'
    case 'quizhub':
      return 'azmoon'
    case 'forum':
      return 'question'
    case 'tutorial':
      return 'dars'
    case 'test':
      return 'test'
    case 'learnfiles':
      return 'learnfiles'
    case 'azmoon':
      return 'azmoon'
    case 'question':
      return 'question'
    case 'dars':
      return 'dars'
    case 'teacher':
      return 'teacher'
    default:
      return 'test'
  }
}

const activeCategory = computed(() => getEquivalentNewType(route.query.type))

const buildSearchParams = (query, page, perpage) => {
  const frontendType = getEquivalentNewType(query.type)
  const params = {
    page,
    perpage,
    title: query.title,
    section: query.section,
    base: query.base,
    lesson: query.lesson,
    type: getEquivalentOldType(frontendType),
  }

  if (frontendType === 'paper') {
    params.is_paper = true
    params.test_type = query.test_type
    params.variant = query.variant
    params.edu_year = query.edu_year
    params.edu_month = query.edu_month
  }
  else if (frontendType === 'study-materials') {
    params.is_paper = false
    params.test_type = query.test_type
    params.topic = query.topic
  }
  else if (frontendType === 'quizhub') {
    params.exam_type = query.exam_type
    params.topic = query.topic
    params.edu_year = query.edu_year
    params.edu_month = query.edu_month
  }
  else if (frontendType === 'tutorial') {
    params.topic = query.topic
  }
  else if (frontendType === 'multimedia') {
    params.content_type = query.content_type
  }
  else {
    params.topic = query.topic
    params.test_type = query.test_type
    params.content_type = query.content_type
    params.edu_year = query.edu_year
    params.edu_month = query.edu_month
  }

  return params
}

const querySearch = ref({
  ...route.query,
  type: getEquivalentNewType(route.query.type),
  page: Number(route.query.page) || 1,
})
const isInitialDataLoading = ref(false)
const isPaginationDataLoading = ref(false)
const isPreviousLoading = ref(false)
const data = ref([])
const isAllDataLoaded = ref(false)
const totalDataFind = ref(0)
const perPage = 10
const perPageServerSide = 5
const firstLoadedPageNumber = ref(Number(route.query.page) || 1)
const latestLoadedPageNumber = ref(Number(route.query.page) || 1)

const loadNextPageData = async () => {
  latestLoadedPageNumber.value += 1
  querySearch.value.page = latestLoadedPageNumber.value
  const query = { ...route.query }

  query.page = querySearch.value.page
  router.replace({ query })
  isPaginationDataLoading.value = true
  const responseList = await getDataList()
  data.value = [...data.value, ...responseList]
}

const loadPreviousPageData = async () => {
  firstLoadedPageNumber.value -= 1

  querySearch.value.page = firstLoadedPageNumber.value
  const query = { ...route.query }

  query.page = querySearch.value.page
  router.replace({ query })
  isPreviousLoading.value = true
  const responseList = await getDataList()
  data.value = [...responseList, ...data.value]
}

const { data: initialData, pending: _loadingDataServer } = await useAsyncData(
  'dataSearchSSR',
  () => {
    const pageNumber = Number(route.query.page) || 1
    if (getEquivalentOldType(route.query.type) == 'teacher') {
      const query = {
        'PagingDto.PageFilter.Size': perPageServerSide,
        'PagingDto.PageFilter.Skip': (pageNumber - 1) * perPageServerSide,
        'PagingDto.PageFilter.ReturnTotalRecordsCount': true,
      }
      return useApiService.get('/api/v2/identities/profiles/list', query)
    }
    else {
      const params = buildSearchParams(route.query, pageNumber, perPageServerSide)

      return useApiService.get('/api/v1/search', params, { public: true })
    }
  },
)

watchEffect(() => {
  if (initialData.value) {
    data.value = initialData.value.data.list
  }
})

if (initialData.value) {
  data.value = initialData.value.data.list
  if (getEquivalentOldType(route.query.type) == 'teacher') {
    totalDataFind.value = initialData.value.data.totalRecordsCount || 0
  }
  else {
    totalDataFind.value = initialData.value.data.num || 0
  }
  isInitialDataLoading.value = false
  isPaginationDataLoading.value = false
}

const getDataList = async () => {
  if (isAllDataLoaded.value) return
  try {
    const typeRoute = getEquivalentOldType(querySearch.value.type)
    let response = {}

    if (typeRoute == 'teacher') {
      const query = {
        'PagingDto.PageFilter.Size': perPage,
        'PagingDto.PageFilter.Skip': (querySearch.value.page - 1) * perPage,
        'PagingDto.PageFilter.ReturnTotalRecordsCount': true,
      }
      response = await useApiService.get('/api/v2/identities/profiles/list', query)
      totalDataFind.value = response.data.totalRecordsCount || 0
    }
    else {
      const params = buildSearchParams(querySearch.value, querySearch.value.page, perPage)
      response = await useApiService.get('/api/v1/search', params)
      totalDataFind.value = response.data.num || 0
    }

    if (response.data.list && response.data.list.length < perPage) {
      isAllDataLoaded.value = true
    }

    return response.data.list
  }
  catch (err) {
    console.error(err)
    return []
  }
  finally {
    isPaginationDataLoading.value = false
    isInitialDataLoading.value = false
    isPreviousLoading.value = false
  }
}

const allMonths = [
  { id: 1, title: 'January' },
  { id: 2, title: 'February' },
  { id: 3, title: 'March' },
  { id: 4, title: 'April' },
  { id: 5, title: 'May' },
  { id: 6, title: 'June' },
  { id: 7, title: 'July' },
  { id: 8, title: 'August' },
  { id: 9, title: 'September' },
  { id: 10, title: 'October' },
  { id: 11, title: 'November' },
  { id: 12, title: 'December' },
]

const specialMonths = {
  4161: [
    { id: 3, title: 'March' },
    { id: 6, title: 'May/Jun' },
    { id: 11, title: 'Oct/Nov' },
  ],
  22: [
    { id: 3, title: 'March' },
    { id: 6, title: 'May/Jun' },
    { id: 11, title: 'Oct/Nov' },
  ],
  6374: [
    { id: 6, title: 'May/Jun' },
    { id: 11, title: 'Oct/Nov' },
  ],
  23: [
    { id: 6, title: 'May/Jun' },
    { id: 11, title: 'Oct/Nov' },
  ],
  6533: [
    { id: 6, title: 'May/Jun' },
    { id: 11, title: 'Oct/Nov' },
  ],
  6635: [
    { id: 3, title: 'March' },
    { id: 4, title: 'April' },
    { id: 5, title: 'May' },
    { id: 10, title: 'October' },
  ],
  6639: [
    { id: 3, title: 'March' },
    { id: 4, title: 'April' },
    { id: 5, title: 'May' },
    { id: 10, title: 'October' },
  ],
  // AQA-GCSE
  6672: [
    { id: 6, title: 'June' },
    { id: 11, title: 'November' },
  ],
  // AQA-AS LEVEL
  6673: [{ id: 6, title: 'June' }],
  // AQA-A LEVEL
  6674: [{ id: 6, title: 'June' }],
  // OCR GCSE
  6676: [
    { id: 6, title: 'June' },
    { id: 11, title: 'November' },
  ],
  // OCR AS LEVEL
  6677: [
    { id: 6, title: 'June' },
    { id: 11, title: 'Oct/Nov' },
  ],
  // OCR A LEVEL
  6678: [
    { id: 6, title: 'June' },
    { id: 11, title: 'Oct/Nov' },
  ],
  // Edexcel A LEVEL
  6671: [
    { id: 6, title: 'May/Jun' },
    { id: 11, title: 'Oct/Nov' },
  ],
  // Edexcel AS LEVEL
  6675: [
    { id: 6, title: 'May/Jun' },
    { id: 11, title: 'Oct/Nov' },
  ],
  // Edexcel GCSE
  6669: [
    { id: 6, title: 'May/Jun' },
    { id: 11, title: 'Oct/Nov' },
  ],
  // Edexcel International A/AS LEVEL
  6670: [
    { id: 1, title: 'January' },
    { id: 4, title: 'April' },
    { id: 6, title: 'June' },
    { id: 10, title: 'October' },
    { id: 11, title: 'November' },
  ],
  // Edexcel International GCSE
  6668: [
    { id: 1, title: 'January' },
    { id: 4, title: 'April' },
    { id: 6, title: 'June' },
    { id: 10, title: 'October' },
    { id: 11, title: 'November' },
  ],
}
const categoryOptions = [
  { title: 'Past Papers', id: 'paper', contentIcon: 'stat-icon icon-paper', color: '#2e90fa' },
  { title: 'Study Materials', id: 'study-materials', icon: '/images/study-materials.svg', iconPadding: 3, color: 'rgb(18, 183, 106)' },
  { title: 'Exam Hub', id: 'quizhub', contentIcon: 'stat-icon icon-exam', color: '#7c4dff' },
  { title: 'Tutorial', id: 'tutorial', contentIcon: 'stat-icon icon-tutorial', color: '#2e90fa' },
]

const defaultCategory = categoryOptions[0]
const activeCategoryColor = computed(() =>
  (categoryOptions.find(category => category.id === activeCategory.value) || defaultCategory).color,
)

const makeFilter = overrides => ({
  selectedItem: null,
  disabled: false,
  hasSearch: true,
  refElement: null,
  api: null,
  idInParams: false,
  extraApiParams: {},
  dependencies: [],
  children: [],
  closable: true,
  boxed: true,
  ...overrides,
})

const BOARD_ICON_BY_TITLE = {
  Cambridge: 'CIE',
  Edexcel: 'Edexcel',
  AQA: 'AQA',
  OCR: 'OCR',
  GAMA: 'GAMA',
  CXC: 'CXC',
  Nigerian: 'Nijeria',
}

const enrichBoardsWithIcons = async (boards) => {
  try {
    const response = await useApiService.get('/api/v2/boards', undefined, { public: true })
    const boardByCode = new Map(
      (response.data || []).map(board => [String(board.code), board]),
    )

    return boards.map((board) => {
      const v2Board = boardByCode.get(String(board.code))
      return {
        ...board,
        apiIcon: v2Board?.icon || null,
        icon: BOARD_ICON_BY_TITLE[v2Board?.title || board.title] || null,
      }
    })
  }
  catch {
    return boards.map(board => ({
      ...board,
      icon: BOARD_ICON_BY_TITLE[board.title] || null,
    }))
  }
}

const filters = computed(() => {
  const category = activeCategory.value
  const conditionalFilters = {
    paper: ['year', 'session', 'paper', 'variant'],
    'study-materials': ['material', 'topic'],
    quizhub: ['topic', 'year', 'session', 'exam-type'],
    tutorial: ['topic'],
  }[category] || []

  const index = {
    board: 0,
    level: 1,
    subject: 2,
    category: 3,
  }
  conditionalFilters.forEach((name, offset) => {
    index[name] = offset + 4
  })

  const boardChildren = [index.level]
  if (index.paper !== undefined) boardChildren.push(index.paper)
  if (index.material !== undefined) boardChildren.push(index.material)
  const subjectChildren = index.topic === undefined ? [] : [index.topic]
  const categoryChildren = conditionalFilters.map(name => index[name])

  const result = [
    makeFilter({
      title: 'Board',
      api: '/api/v1/types/list',
      extraApiParams: { type: 'section' },
      itemTransform: item => ({ ...item, code: item.id }),
      listTransform: enrichBoardsWithIcons,
      itemSort: (a, b) => Number(a.list_order) - Number(b.list_order),
      showItemIcon: true,
      iconSrc: item => `/images/boards/${item.icon}.svg`,
      fallbackIcon: 'md:school',
      fallbackIconSrc: '/images/board-fallback.svg',
      emptyFallbackIconSrc: '/images/board-control-fallback.svg',
      fallbackIconPadding: 4,
      unselectedIconColor: '#000000',
      queryKey: 'section',
      children: boardChildren,
    }),
    makeFilter({
      title: 'Level',
      disabled: true,
      api: '/api/v1/types/list',
      extraApiParams: { type: 'base' },
      dependencies: [{ parent: index.board, targetKey: 'section_id', sourceKey: 'code' }],
      queryKey: 'base',
      children: [index.subject],
      childrenForGetStaticData: index.session === undefined ? [] : [index.session],
    }),
    makeFilter({
      title: 'Subject',
      disabled: true,
      api: '/api/v1/types/list',
      extraApiParams: { type: 'lesson' },
      dependencies: [{ parent: index.level, targetKey: 'base_id', sourceKey: 'id' }],
      queryKey: 'lesson',
      children: subjectChildren,
    }),
    makeFilter({
      title: 'Category',
      hasSearch: false,
      staticList: categoryOptions,
      queryKey: 'type',
      children: categoryChildren,
      closable: false,
      defaultValue: defaultCategory,
      showItemIcon: true,
      iconSrc: item => item.icon,
      fallbackIcon: 'md:category',
    }),
  ]

  const factories = {
    topic: () => makeFilter({
      title: 'Topic',
      disabled: true,
      api: '/api/v1/types/list',
      extraApiParams: { type: 'topic' },
      dependencies: [{ parent: index.subject, targetKey: 'lesson_id', sourceKey: 'id' }],
      queryKey: 'topic',
      selectedVariant: 'dependent-green',
      controlIcon: 'md:sell_outlined',
      unselectedIconColor: '#000000',
      controlIconPadding: 4,
    }),
    year: () => makeFilter({
      title: 'Year',
      dependencies: [{ parent: index.category, targetKey: 'type', sourceKey: 'id' }],
      staticList: Array.from({ length: 14 }, (_, i) => 2013 + i)
        .reverse()
        .map(year => ({ title: `${year}`, id: year })),
      queryKey: 'edu_year',
      selectedVariant: 'dependent-green',
      controlIconSvg: {
        viewBox: '0 0 28 28',
        strokeWidth: 0.6,
        paths: [
          'M25.375 28H2.625C1.1725 28 0 26.8275 0 25.375V4.375C0 2.9225 1.1725 1.75 2.625 1.75H25.375C26.8275 1.75 28 2.9225 28 4.375V25.375C28 26.8275 26.8275 28 25.375 28ZM2.625 3.5C2.135 3.5 1.75 3.885 1.75 4.375V25.375C1.75 25.865 2.135 26.25 2.625 26.25H25.375C25.865 26.25 26.25 25.865 26.25 25.375V4.375C26.25 3.885 25.865 3.5 25.375 3.5H2.625Z',
          'M7.875 7C7.385 7 7 6.615 7 6.125V0.875C7 0.385 7.385 0 7.875 0C8.365 0 8.75 0.385 8.75 0.875V6.125C8.75 6.615 8.365 7 7.875 7ZM20.125 7C19.635 7 19.25 6.615 19.25 6.125V0.875C19.25 0.385 19.635 0 20.125 0C20.615 0 21 0.385 21 0.875V6.125C21 6.615 20.615 7 20.125 7ZM27.125 10.5H0.875C0.385 10.5 0 10.115 0 9.625C0 9.135 0.385 8.75 0.875 8.75H27.125C27.615 8.75 28 9.135 28 9.625C28 10.115 27.615 10.5 27.125 10.5Z',
        ],
      },
      unselectedIconColor: '#000000',
      controlIconPadding: 4,
    }),
    session: () => makeFilter({
      title: 'Session',
      dependencies: [{ parent: index.category, targetKey: 'type', sourceKey: 'id' }],
      staticList: [],
      dependenciesForGetStaticData: [index.level],
      getStaticList: (id = route.query.base) => {
        const levelId = id === 'reset' ? route.query.base : (id || route.query.base)
        return levelId && specialMonths[levelId] ? specialMonths[levelId] : allMonths
      },
      queryKey: 'edu_month',
      selectedVariant: 'dependent-green',
    }),
    paper: () => makeFilter({
      title: 'Paper',
      disabled: true,
      api: '/api/v1/types/list',
      extraApiParams: { type: 'test_type' },
      dependencies: [{ parent: index.board, targetKey: 'section_id', sourceKey: 'code' }],
      itemFilter: item => item.is_paper === true,
      queryKey: 'test_type',
      inlineOptions: true,
      inlineAllowClear: true,
      itemTitle: (item) => {
        const match = item.title?.match(/^\s*paper\s+(\d+)\s*$/i)
        return match ? match[1] : item.title
      },
    }),
    variant: () => makeFilter({
      title: 'Variants',
      staticList: [
        { id: '7814', title: '1' },
        { id: '7815', title: '2' },
        { id: '7816', title: '3' },
      ],
      queryKey: 'variant',
      inlineOptions: true,
      inlineAllowClear: true,
    }),
    material: () => makeFilter({
      title: 'Material Type',
      disabled: true,
      api: '/api/v1/types/list',
      extraApiParams: { type: 'test_type' },
      dependencies: [{ parent: index.board, targetKey: 'section_id', sourceKey: 'code' }],
      itemFilter: item => item.is_paper === false,
      queryKey: 'test_type',
      selectedVariant: 'dependent-green',
    }),
    'exam-type': () => makeFilter({
      title: 'Exam Type',
      api: '/api/v1/types/list',
      extraApiParams: { type: 'exam_type' },
      queryKey: 'exam_type',
      selectedVariant: 'dependent-green',
    }),
  }

  conditionalFilters.forEach(name => result.push(factories[name]()))
  return result
})

const lastRequestedCategory = ref(activeCategory.value)

const scrollToPageTop = async () => {
  if (!import.meta.client) return

  await nextTick()
  window.scrollTo({
    top: 0,
    left: 0,
    behavior: 'smooth',
  })
  await new Promise(resolve => requestAnimationFrame(resolve))
}

const changeFilter = async (query) => {
  lastRequestedCategory.value = getEquivalentNewType(query.type)
  isAllDataLoaded.value = false
  isInitialDataLoading.value = true
  firstLoadedPageNumber.value = 1
  latestLoadedPageNumber.value = 1
  querySearch.value = { ...query, page: 1 }
  await scrollToPageTop()
  const responseList = await getDataList()
  data.value = responseList
}

watch(activeCategory, async (category) => {
  if (category === lastRequestedCategory.value) return

  lastRequestedCategory.value = category
  isAllDataLoaded.value = false
  isInitialDataLoading.value = true
  firstLoadedPageNumber.value = 1
  latestLoadedPageNumber.value = 1
  querySearch.value = { ...route.query, type: category, page: 1 }
  await scrollToPageTop()
  data.value = await getDataList()
})

// Computed metadata that updates when data changes
const metadata = computed(() => {
  const { section, base, lesson, test_type, edu_year, edu_month } = route.query
  const firstElement = data.value[0]

  let monthTitle = edu_month
    ? dayjs()
        .month(edu_month - 1)
        .format('MMMM')
    : ''
  if (base && edu_month) {
    const monthList = specialMonths[base]
    if (monthList && monthList.length > 0) {
      const month = monthList.filter(item => item.id == edu_month)
      if (month.length > 0) {
        monthTitle = month[0].title
      }
    }
  }

  const titles = {
    boardTitle:
      section && firstElement ? firstElement.section_title : undefined,
    gradeTitle: section && base && firstElement ? firstElement.base_title : '',
    subjectTitle:
      section && base && lesson && firstElement
        ? firstElement.lesson_title
        : '',
    classificationTitle: '',
    yearTitle: edu_year ? edu_year : '',
    monthTitle: monthTitle,
    is_paper: firstElement?.is_paper,
  }

  if (
    (getEquivalentOldType(route.query.type) == 'test' || getEquivalentOldType(route.query.type) == 'azmoon')
    && test_type
  ) {
    titles.classificationTitle
      = firstElement?.test_type_title || firstElement?.azmoon_type_title
  }

  const joinTextTitles = `${titles.boardTitle} ${titles.gradeTitle} ${titles.subjectTitle} ${titles.classificationTitle} ${titles.monthTitle} ${titles.yearTitle}`

  // Generate title
  let appendText = ''
  if (titles.is_paper) {
    appendText = 'Past Papers'
  }

  const titleTemplates = {
    learnfiles: {
      dynamic: `${joinTextTitles} multimedia`,
      fallback:
        'Multimedia Interactive Educational Content; PowerPoint, Video, Class Voice, GamaTrain',
    },
    test: {
      dynamic: `${joinTextTitles} ${appendText}`,
      fallback: 'Educational Resources | K12 Education Papers and Materials',
    },
    question: {
      dynamic: `${joinTextTitles} Forum`,
      fallback:
        'Seek Clarification, Expand Your Understanding: GamaTrain\'s Q&A Forum',
    },
    azmoon: {
      dynamic: `${joinTextTitles} Online test`,
      fallback: 'Online Exams, Free Exams for Improving Education',
    },
    dars: {
      dynamic: `${joinTextTitles} Textbook`,
      fallback:
        'Master Concepts, Enhance Learning: GamaTrain\'s Online Tutorials',
    },
    teacher: {
      dynamic: 'Teacher directory',
      fallback: 'Teacher directory',
    },
    default: {
      dynamic: `${joinTextTitles} Past Papers`,
      fallback: 'Educational Resources | K12 Education Papers and Materials',
    },
  }

  const template = titleTemplates[getEquivalentOldType(route.query.type)] || titleTemplates.default
  const title = titles.boardTitle ? template.dynamic : template.fallback

  // Generate description
  let descAppendText = ''
  if (titles.is_paper) {
    descAppendText = 'Includes mark scheme for exam preparation.'
  }
  else {
    descAppendText = 'Useful for study, practice, and exam preparation.'
  }

  const pageDescriptions = {
    learnfiles:
      'Elevate your learning experience with GamaTrain\'s captivating multimedia content, including PowerPoint presentations, informative videos, and diverse educational materials.',
    test: 'Enhance your learning with GamaTrain\'s extensive collection of online documents and texts, carefully curated to enrich your academic journey.',
    question:
      'Engage in active learning and gain deeper insights through GamaTrain\'s interactive Q&A platform, where you can pose questions and seek support from fellow learners and experts.',
    azmoon:
      'Hone your skills and assess your knowledge with GamaTrain\'s online exams, designed to enhance your exam preparation and boost your confidence.',
    dars: 'Complement your studies with GamaTrain\'s comprehensive online tutorials, providing step-by-step guidance and practice opportunities to refine your understanding.',
    teacher: 'Browse qualified teachers and explore their profiles, experience, and subjects they teach.',
  }

  const descriptionTemplates = {
    learnfiles: {
      dynamic: `Download list of ${joinTextTitles}  multimedia. ${descAppendText}`,
    },
    test: {
      dynamic: `Download list of ${joinTextTitles} ${appendText}. ${descAppendText}`,
    },
    question: {
      dynamic: `Download list of ${joinTextTitles} Forum. ${descAppendText}`,
    },
    azmoon: {
      dynamic: `Download list of ${joinTextTitles} Online test. ${descAppendText}`,
    },
    dars: {
      dynamic: `Download list of ${joinTextTitles} Textbook. ${descAppendText}`,
    },
    teacher: {
      dynamic: `Browse qualified teachers and explore their profiles, experience, and subjects they teach.`,
    },
    default: {
      dynamic: `Download list of ${joinTextTitles} ${appendText}. ${descAppendText}`,
    },
  }

  const descTemplate
    = descriptionTemplates[getEquivalentOldType(route.query.type)] || descriptionTemplates.default
  const description = titles.boardTitle
    ? descTemplate.dynamic
    : pageDescriptions[getEquivalentOldType(route.query.type)] || pageDescriptions.test

  return { title, description }
})

// Reactive useHead that updates when metadata changes
useHead(() => ({
  title: metadata.value.title,
  meta: [
    {
      hid: 'apple-mobile-web-app-title',
      name: 'apple-mobile-web-app-title',
      content: metadata.value.title,
    },
    {
      hid: 'og:title',
      name: 'og:title',
      content: metadata.value.title,
    },
    {
      hid: 'og:site_name',
      name: 'og:site_name',
      content: 'GamaTrain',
    },
    {
      hid: 'description',
      name: 'description',
      content: metadata.value.description,
    },
    {
      hid: 'og:description',
      name: 'og:description',
      content: metadata.value.description,
    },
  ],
}))

const createLinkAddConent = () => {
  const auth = useAuth()
  const router = useRouter()
  if (!auth.isAuthenticated.value)

    router.push({ query: { auth_form: 'login' } })
  else {
    const type = getEquivalentOldType(route.query.type)
    let link = ''
    switch (type) {
      case 'test':
        link = '/user/paper/create'
        break
      case 'learnfiles':
        link = '/user/multimedia/create'
        break
      case 'azmoon':
        link = '/test-maker/create'
        break
      case 'question':
        link = '/user/question/create'
        break
      case 'dars':
        link = '/user/paper/create'
        break

      default:
        link = '/user/paper/create'
        break
    }
    navigateTo(link)
  }
}

onMounted(() => {
  const oldType = ['test', 'learnfiles', 'azmoon', 'question', 'dars']
  const normalizedType = getEquivalentNewType(route.query.type)
  if (!route.query.type || oldType.includes(route.query.type)) {
    router.replace({
      query: {
        ...route.query,
        type: normalizedType,
      },
    })
  }
})
</script>

<style scoped>
.top-info-div {
  display: contents !important;
}
.margin-top-handle {
  min-height: 100vh;
}

:deep(.inline-filter-group) {
  border: 0;
  max-width: 100%;
  flex-direction: row;
  align-items: stretch;
  gap: 24px;
}

:deep(.inline-filter-group > .inline-filter-grouped-row) {
  width: auto;
  min-width: 0;
  flex: 0 1 auto;
  padding: 0;
}

:deep(.inline-filter-group > .inline-filter-grouped-row + .inline-filter-grouped-row) {
  padding-left: 24px;
  border-left: 1px solid rgb(var(--v-theme-grey300));
}

:deep(.inline-filter-grouped-row .inline-filter-row-content) {
  height: 100%;
  grid-template-columns: max-content minmax(0, 1fr);
  column-gap: 16px;
}

:deep(.inline-filter-group .inline-filter-divider) {
  display: none;
}

:deep(.inline-filter-group .inline-filter-option:not(.inline-filter-option-selected):not(:disabled):hover) {
  --v-hover-opacity: 0;
  border-color: #397f7b !important;
  background-color: #f2f9f8 !important;
}

@media (max-width: 959px) {
  :deep(.inline-filter-group) {
    flex-direction: column;
    gap: 16px;
  }

  :deep(.inline-filter-group > .inline-filter-grouped-row + .inline-filter-grouped-row) {
    padding-top: 16px;
    padding-left: 0;
    border-top: 1px solid rgb(var(--v-theme-grey300));
    border-left: 0;
  }
}

:deep(.custom-search-text-field .v-field__outline__start) {
  border-radius: 24px 0 0 24px !important;
  flex: 0 0 30px !important;
}
:deep(.custom-search-text-field .v-field__outline__end) {
  border-radius: 0 4px 4px 0 !important;
}

:deep(.height-badge .v-badge__wrapper .v-badge__badge) {
  height: 20px !important;
}
.max-width-container {
  max-width: 1200px;
}
.search-results-heading {
  min-width: 0;
  margin-bottom: 16px;
}
.search-results-title {
  min-width: 0;
  margin: 0;
  font-size: 24px;
  font-weight: 700;
  line-height: 32px;
  text-align: left;
}
.search-results-count-number {
  color: #397f7b;
  font-size: 24px;
  font-weight: 700;
  line-height: 32px;
}
.search-results-count-label {
  color: rgb(var(--v-theme-grey700));
  font-size: 16px;
  font-weight: 500;
  line-height: 24px;
}

</style>
