<template>
  <v-container
    v-if="contentData"
    class="d-flex flex-column mt-16"
  >
    <v-row>
      <lazy-widgets-breadcrumb
        background-color="white"
        :breads="breads"
      />
    </v-row>
    <v-row>
      <v-col
        cols="12"
        class="d-flex align-center ga-1"
      >
        <v-icon color="primary">
          md:chevron_backward
        </v-icon>
        <h1 class="text-h4 font-weight-bold">
          {{ contentData.title }}
        </h1>
        <v-icon color="primary">
          md:chevron_forward
        </v-icon>

        <v-btn
          v-if="
            isAuthenticated
              && user
              && contentData.user_
              && Number(contentData.user_) === user?.coreId
          "
          color="info"
          class="rounded-circle"
          size="24"
          flat
          variant="tonal"
          @click="openEditModal = true"
        >
          <v-icon size="16">
            md:edit
          </v-icon>
        </v-btn>
      </v-col>
      <v-col
        cols="12"
        md="4"
        class="d-flex justify-center justify-md-start"
      >
        <lazy-paper-preview-action-card
          :id="contentData.id"
          :thumb-pic="contentData.thumb_pic"
          :title="contentData.title"
          :alt="pageTitle"
          :views="contentData.views"
          :score="contentData.ref_score"
          @share="openShare = true"
        />
      </v-col>

      <v-col
        cols="12"
        md="8"
        class="d-flex h-100 align-start flex-wrap"
      >
        <lazy-paper-content-details-section :content-data="contentData" />

        <lazy-paper-answer-availability-notice
          v-if="
            (contentData.answer_type === '1' || contentData.answer_type === '2')
              && !contentData.files?.answer?.exist
          "
        />

        <lazy-paper-download-and-purchase-buttons
          :id="contentData.id"
          :files="contentData.files"
          :year="contentData.edu_year"
          :title="contentData.title"
          :title-url="contentData.title_url"
          :section="contentData.section"
          :base="contentData.base"
          :lesson="contentData.lesson"
          :exams="contentData.exams"
          :test-type="contentData.test_type"
          :is-paper="contentData.is_paper"
        />
        <lazy-common-detail-subject-directory-nav :content-data="contentData" />
      </v-col>

      <lazy-common-box-random-question :lesson="contentData.lesson" />

      <v-col cols="12">
        <lazy-common-related-content
          :id="contentData.id"
          source="test"
          :request="[`test`, `file`, `exam`, `question`, `tutorial`]"
        />
      </v-col>

      <v-col
        cols="12"
        class="mt-6"
      >
        <span
          class="d-flex align-center ga-1 text-h5 cursor-pointer text-lightError"
          @click="openCrashReport = true"
        >
          <v-icon
            color="lightError"
            class="mb-1"
          >md:warning_outlined</v-icon>
          Crash report
        </span>
      </v-col>

      <ClientOnly>
        <v-col
          cols="12"
          class="text-center mt-10"
        >
          <lazy-common-ad-banner
            v-model="isAdsLoad"
            adslot="7199289937"
          />
        </v-col>
      </ClientOnly>
    </v-row>

    <lazy-common-modal-base
      v-model:show-dialog="openCrashReport"
      title="Crash Report"
    >
      <lazy-common-modal-crash-report
        :id="contentData.id"
        type-crash-report="test"
        @close="openCrashReport = false"
      />
    </lazy-common-modal-base>

    <lazy-common-modal-base
      v-model:show-dialog="openShare"
      title="Share"
    >
      <lazy-common-modal-share :title="contentData.title" />
    </lazy-common-modal-base>

    <lazy-common-modal-base
      v-if="openEditModal"
      v-model:show-dialog="openEditModal"
      title="Edit"
    >
      <lazy-paper-modal-edit
        :id="contentData.id"
        :title="contentData.title"
        :description="contentData.description"
        @close="openEditModal = false"
        @success="editSuccessfully"
      />
    </lazy-common-modal-base>
  </v-container>
</template>

<script setup lang="ts">
import type { ApiResult, PastPaperDTO, FilesDTO, AppError } from '@/types'
import { CAMBRIDGE_BOARD_ID } from '@/constants'

interface BreadCrumb {
  text: string
  disabled: boolean
  href: string
}

const route = useRoute()
const router = useRouter()
const { user } = useUser()
const { isAuthenticated } = useAuth()

const { buildSchema } = useSeoSchema()

const requestURL = ref(useRequestURL().host)
const pageDescribe = ref('')
const pageTitle = ref('')
const breads = ref<BreadCrumb[]>([])
const openCrashReport = ref(false)
const openShare = ref(false)
const openEditModal = ref(false)
const isAdsLoad = ref(false)

const { data: contentData } = await useAsyncData(
  `paper-${route.params.id}`,
  async () => {
    try {
      const response = (await useApiService.get(
        `/api/v1/tests/${route.params.id}`,
        undefined,
        {
          public: true,
          proxy: true,
        },
      )) as ApiResult<PastPaperDTO>

      if (response.data) {
        return response.data
      }
      else {
        showError({
          statusCode: 404,
          statusMessage: 'Page Not Founded!',
        })
        return null
      }
    }
    catch (e: unknown) {
      showError({
        statusCode: 404,
        statusMessage: 'Page Not Founded!',
      })
      const error = e as AppError
      if (error?.status === 404) {
        router.push('/search?type=paper')
      }
      throw error
    }
    finally {
      // Reset loading states if needed
    }
  },
)

const schema = computed(() => {
  if (!contentData.value) return null

  const dto = contentData.value

  const url = `https://${requestURL.value}/paper/${dto.id}/${dto.title_url}`

  const title = pageTitle.value
  const description = pageDescribe.value

  return buildSchema({
    dto,
    url,
    title,
    description,
    breadcrumbs: breads.value,
    types: ['article', 'breadcrumb'],
  })
})

const CAMBRIDGE_SESSION_MAP: Record<
  string,
  Record<string, { display: string, code: string }>
> = {
  22: {
    3: { display: 'February March', code: 'F/M' },
    6: { display: 'May June', code: 'M/J' },
    11: { display: 'October November', code: 'O/N' },
  },
  4161: {
    3: { display: 'February March', code: 'F/M' },
    6: { display: 'May June', code: 'M/J' },
    11: { display: 'October November', code: 'O/N' },
  },
  6535: {
    3: { display: 'February March', code: 'F/M' },
    6: { display: 'May June', code: 'M/J' },
    11: { display: 'October November', code: 'O/N' },
  },
  23: {
    6: { display: 'May June', code: 'M/J' },
    11: { display: 'October November', code: 'O/N' },
  },
  6374: {
    6: { display: 'May June', code: 'M/J' },
    11: { display: 'October November', code: 'O/N' },
  },
  6533: {
    6: { display: 'May June', code: 'M/J' },
    11: { display: 'October November', code: 'O/N' },
  },
  6635: {
    3: { display: 'February March', code: 'F/M' },
    4: { display: 'April May', code: 'A/M' },
    5: { display: 'April May', code: 'A/M' },
    10: { display: 'October November', code: 'O/N' },
  },
  6639: {
    3: { display: 'February March', code: 'F/M' },
    4: { display: 'April May', code: 'A/M' },
    5: { display: 'April May', code: 'A/M' },
    10: { display: 'October November', code: 'O/N' },
  },
}

interface CambridgeFileType {
  code: string
  label: string
}

const CAMBRIDGE_FILENAME_SESSION_CODES: Record<string, string> = {
  'M/J': 's',
  'A/M': 's',
  'O/N': 'w',
  'F/M': 'm',
}

const CAMBRIDGE_EXTRA_FILE_TYPES: Record<string, CambridgeFileType> = {
  'insert': { code: 'in', label: 'insert' },
  'audio file': { code: 'sf', label: 'sound file (SF)' },
  'confidential instructions': { code: 'ci', label: 'confidential instructions' },
  'confidental instructions': { code: 'ci', label: 'confidential instructions' },
  "teacher's notes": { code: 'tn', label: "teacher's notes" },
  'teacher notes': { code: 'tn', label: "teacher's notes" },
  'transcript': { code: 'qr', label: 'transcript (QR)' },
  'map file': { code: 'i2', label: 'map file' },
  'pre released material': { code: 'pm', label: 'pre released material' },
}

const getCambridgeFilenameSessionCode = (seoSessionCode: string) =>
  CAMBRIDGE_FILENAME_SESSION_CODES[seoSessionCode]

const getCambridgeAvailableFiles = (
  files: FilesDTO,
  filenamePrefix: string,
  paperVariantCode: string,
) => {

  const availableFiles: string[] = []

  if (files.pdf.exist) {
    availableFiles.push(
      `question paper (QP) ${filenamePrefix}_qp_${paperVariantCode}`,
    )
  }

  if (
    files.answer.exist
    && String(files.answer.ext).toLowerCase() !== 'word'
  ) {
    availableFiles.push(
      `mark scheme (MS) ${filenamePrefix}_ms_${paperVariantCode}`,
    )
  }

  const addedExtraFileCodes = new Set<string>()

  for (const extraFile of files.extra ?? []) {
    const normalizedTypeTitle = extraFile.type_title
      ?.trim()
      .toLowerCase()
      .replaceAll('’', "'")

    if (!normalizedTypeTitle) continue

    const fileType = CAMBRIDGE_EXTRA_FILE_TYPES[normalizedTypeTitle]

    if (!fileType || addedExtraFileCodes.has(fileType.code)) continue

    availableFiles.push(
      `${fileType.label} ${filenamePrefix}_${fileType.code}_${paperVariantCode}`,
    )
    addedExtraFileCodes.add(fileType.code)
  }

  return availableFiles.length > 0 ? availableFiles : null
}

const setMetaData = () => {
  if (!contentData.value) return

  const dto: PastPaperDTO = contentData.value
  const {
    section,
    base,
    section_title,
    base_title,
    lesson_title,
    test_type_title,
    edu_year,
    edu_month,
    variant_title,
    title,
    is_paper,
  } = dto

  // Build title parts safely from DTO
  const titleParts = [
    section_title,
    base_title,
    title,
  ].filter(Boolean)

  const baseTitle = titleParts.join(' ')

  if (is_paper) {
    pageTitle.value = `${baseTitle} past paper`
    pageDescribe.value = `Download ${baseTitle} past paper with mark scheme (MS). Access a full collection of past papers for study, revision, and exam practice.`

    if (String(section) === String(CAMBRIDGE_BOARD_ID)) {
      const subjectMatch = lesson_title
        ?.trim()
        .match(/^(.+?)\s*\((\d{4})\)$/)
      const paperMatch = test_type_title
        ?.trim()
        .match(/^Paper\s+(\d+)$/)
      const normalizedYear = edu_year?.trim() ?? ''
      const session = CAMBRIDGE_SESSION_MAP[String(base)]?.[String(edu_month)]
      const gradeTitle = base_title?.trim()

      if (
        subjectMatch
        && paperMatch
        && /^\d{4}$/.test(normalizedYear)
        && session
        && gradeTitle
      ) {
        const [, subjectName, subjectCode] = subjectMatch
        const paperNumber = paperMatch[1]
        const normalizedVariantTitle = variant_title?.trim() ?? ''
        const paperVariantCode = /^\d+$/.test(normalizedVariantTitle)
          ? `${paperNumber}${normalizedVariantTitle}`
          : paperNumber.padStart(2, '0')
        const shortYear = normalizedYear.slice(-2)

        pageTitle.value = `${pageTitle.value} ${subjectCode}/${paperVariantCode}/${session.code}/${shortYear}`

        const currentCambridgeDescription = `Download Cambridge ${gradeTitle} ${subjectName} ${subjectCode}/${paperVariantCode} ${session.display} ${normalizedYear} question paper(QP) with mark scheme (MS) pdf. Access paper ${paperNumber}, ${subjectCode}/${paperVariantCode}/${session.code}/${shortYear} question paper pdf with answers for your upcoming exam series preparation.`

        pageDescribe.value = currentCambridgeDescription

        const filenameSessionCode = getCambridgeFilenameSessionCode(session.code)

        if (filenameSessionCode) {
          const filenamePrefix = `${subjectCode}_${filenameSessionCode}${shortYear}`
          const availableFiles = getCambridgeAvailableFiles(
            dto.files,
            filenamePrefix,
            paperVariantCode,
          )

          if (availableFiles) {
            pageDescribe.value = `Download ${subjectCode}/${paperVariantCode}/${session.code}/${shortYear} Cambridge ${gradeTitle} ${subjectName.trim()} ${subjectCode}/${paperVariantCode} Paper ${paperNumber} ${session.display} ${normalizedYear} past paper. In this page you can find ${availableFiles.join(', ')} pdfs available for your upcoming exam series preparation.`
          }
        }
      }
    }
  }
  else {
    pageTitle.value = baseTitle
    pageDescribe.value = `Download ${title} – ${base_title}, ${section_title} curriculum. Ideal for quick revision, practice, and exam prep.`
  }

  const ogImage = dto.thumb_pic
    ? dto.thumb_pic
    : null

  useHead({
    title: pageTitle.value,
    meta: [
      {
        name: 'apple-mobile-web-app-title',
        content: pageTitle.value,
      },
      {
        name: 'og:title',
        content: pageTitle.value,
      },
      {
        name: 'og:site_name',
        content: 'GamaTrain',
      },
      {
        name: 'description',
        content: pageDescribe.value,
      },
      {
        name: 'og:description',
        content: pageDescribe.value,
      },
      {
        property: 'og:image',
        content: ogImage,
      },
      {
        name: 'twitter:card',
        content: 'summary_large_image',
      },
      {
        name: 'twitter:title',
        content: pageTitle.value,
      },
      {
        name: 'twitter:description',
        content: pageDescribe.value,
      },
      {
        name: 'twitter:image',
        content: ogImage,
      },
    ],
    script: schema.value
      ? [
          {
            key: 'json-ld-schema',
            type: 'application/ld+json',
            innerHTML: JSON.stringify(schema.value),
          },
        ]
      : [],
    link: [
      {
        rel: 'canonical',
        href: dto
          ? `https://${requestURL.value}/paper/${dto.id}/${dto.title_url}`
          : `https://${requestURL.value}/paper/${route.params.id}`,
      },
    ],
  })
}

const initBreadCrumb = () => {
  if (!contentData.value) return
  breads.value = []
  breads.value.push({
    text: 'Paper',
    disabled: false,
    href: '/search?type=paper',
  })
  breads.value.push(
    {
      text: contentData.value.section_title,
      disabled: false,
      href: `/search?type=paper&section=${contentData.value.section}`,
    },
    {
      text: contentData.value.base_title,
      disabled: false,
      href: `/search?type=paper&section=${contentData.value.section}&base=${contentData.value.base}`,
    },
    {
      text: contentData.value.lesson_title,
      disabled: false,
      href: `/search?type=paper&section=${contentData.value.section}&base=${contentData.value.base}&lesson=${contentData.value.lesson}`,
    },
  )
}

if (contentData.value) {
  initBreadCrumb()
  setMetaData()
}

const editSuccessfully = (data: {
  title: string
  description: string
}) => {
  if (contentData.value) {
    contentData.value = {
      ...contentData.value,
      title: data.title,
      description: data.description,
    }
    setMetaData()
  }
}
</script>

<style scoped>
</style>
