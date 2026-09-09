<template>
  <div
    class="card-search w-100 rounded-xl position-relative"
    role="link"
    tabindex="0"
    :aria-label="information?.title"
    @click="openCard"
    @keydown.enter="openCard"
    @keydown.space.prevent="openCard"
  >
    <div class="card-content d-flex align-stretch">
      <div class="cover-wrap d-flex align-center justify-center flex-shrink-0">
        <v-img
          v-if="information.lesson_pic"
          :alt="information?.title"
          cover
          :src="information.lesson_pic"
          class="cover-image"
        />
        <div
          v-else
          class="cover-fallback d-flex align-center justify-center flex-column text-center"
        >
          <span class="font-weight-bold">{{ fallbackSubject.name }}</span>
          <span v-if="fallbackSubject.code" class="font-weight-bold">
            {{ fallbackSubject.code }}
          </span>
        </div>
      </div>

      <div class="card-body d-flex flex-column min-width-0">
        <div class="card-top d-flex align-start justify-space-between ga-4">
          <div class="publisher d-flex align-center ga-2 min-width-0">
            <v-img
              :src="information.avatar || '/images/default-user.svg'"
              :alt="publisherName"
              width="26"
              height="26"
              cover
              class="publisher-avatar rounded-circle flex-shrink-0"
            />
            <span class="publisher-name text-truncate">{{ publisherName }}</span>
          </div>

          <div
            class="card-indicators d-none d-md-flex align-center flex-shrink-0"
            aria-label="Resource information"
          >
            <DifficultyIndicator :level="information.level" :size="16" />
            <span class="indicator indicator-library" title="Resource available">
              <img :src="libraryCheckIcon" alt="" class="status-icon">
            </span>
            <span
              class="indicator indicator-pdf"
              :class="{ 'indicator-muted': !information.q_file }"
              title="PDF availability"
            >
              <img :src="pdfCardIcon" alt="" class="status-icon">
            </span>
            <span
              class="indicator indicator-word"
              :class="{ 'indicator-muted': !information.q_file_word }"
              title="Word file availability"
            >
              <img :src="wordCardIcon" alt="" class="status-icon">
            </span>
            <span class="indicator indicator-fire" title="Featured resource">
              <img :src="fireCardIcon" alt="" class="status-icon">
            </span>
            <QualityIndicator :score="qualityScore" :size="16" />
          </div>
        </div>

        <h2 class="card-title text-black font-weight-medium text-truncate">
          {{ information?.title }}
        </h2>
        <p
          v-if="description"
          class="card-description text-grey500 text-truncate"
        >
          {{ description }}
        </p>

        <div class="subject-tags d-flex align-center justify-start flex-wrap ga-2">
          <v-chip
            v-show="information.section_title"
            :prefetch="false"
            variant="flat"
            class="tag-chip"
            color="grey100"
            :to="`/search?type=${route.query.type || 'paper'}&section=${information.section}`"
          >
            <span class="text-grey500">{{ information?.section_title }}</span>
          </v-chip>
          <v-chip
            v-show="information.base_title"
            :prefetch="false"
            variant="flat"
            class="tag-chip"
            color="grey100"
            :to="`/search?type=${route.query.type || 'paper'}&section=${information.section}&base=${information.base}`"
          >
            <span class="text-grey500">{{ information?.base_title }}</span>
          </v-chip>
          <v-chip
            v-show="information.lesson_title"
            :prefetch="false"
            variant="flat"
            class="tag-chip"
            color="grey100"
            :to="`/search?type=${route.query.type || 'paper'}&section=${information.section}&base=${information.base}&lesson=${information.lesson}`"
          >
            <span class="text-grey500">{{ information.lesson_title }}</span>
          </v-chip>
        </div>

        <div class="metadata d-flex align-center flex-wrap ga-4 text-grey500">
          <span
            v-if="information.ext && route.query.type == 'learnfiles'"
            class="metadata-item"
          >
            <v-icon size="12" color="grey300">md:sticky_note_outlined</v-icon>
            {{ information.ext }}
          </span>
          <span v-if="information.test_type_title" class="metadata-item">
            <svg
              width="12"
              height="12"
              viewBox="0 0 16 16"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
              style="color: #1E2A44"
              aria-hidden="true"
            >
              <path
                d="M7.5 10C7.77614 10 8 9.77614 8 9.5C8 9.22386 7.77614 9 7.5 9C7.22386 9 7 9.22386 7 9.5C7 9.77614 7.22386 10 7.5 10Z"
                fill="currentColor"
              />
              <path
                d="M13.85 4.64999L10.35 1.14999C10.3062 1.10092 10.2521 1.06206 10.1916 1.03614C10.1312 1.01022 10.0658 0.997883 10 0.999986H5C4.73503 1.00078 4.48113 1.10639 4.29377 1.29375C4.1064 1.48112 4.00079 1.73501 4 1.99999V6.99999H3C2.73478 6.99999 2.48043 7.10534 2.29289 7.29288C2.10536 7.48042 2 7.73477 2 7.99999V11C2 11.2652 2.10536 11.5196 2.29289 11.7071C2.48043 11.8946 2.73478 12 3 12H4V14C4.00079 14.265 4.1064 14.5189 4.29377 14.7062C4.48113 14.8936 4.73503 14.9992 5 15H13C13.265 14.9992 13.5189 14.8936 13.7062 14.7062C13.8936 14.5189 13.9992 14.265 14 14V4.99999C14.0018 4.93427 13.9893 4.86896 13.9634 4.80854C13.9375 4.74812 13.8988 4.69403 13.85 4.64999ZM10 2.19999L12.8 4.99999H10V2.19999ZM3 7.99999H7.7985L9.5 9.49999L7.8035 11H3V7.99999ZM13 14H5V12H7.8045C8.04591 11.9998 8.2791 11.9122 8.461 11.7535L10.153 10.258C10.2615 10.1644 10.3486 10.0486 10.4085 9.91838C10.4684 9.7882 10.4996 9.64667 10.5001 9.50338C10.5006 9.36008 10.4703 9.21835 10.4113 9.08777C10.3522 8.95719 10.2659 8.84081 10.158 8.74649L8.456 7.24199C8.27488 7.08613 8.04394 7.00029 7.805 6.99999H5V1.99999H9V4.99999C9.00079 5.26496 9.1064 5.51885 9.29377 5.70622C9.48113 5.89358 9.73503 5.99919 10 5.99999H13V14Z"
                fill="currentColor"
              />
            </svg>
            {{ information.test_type_title }}
          </span>
          <span
            v-if="information.tests_num && route.query.type == 'azmoon'"
            class="metadata-item"
          >
            <v-icon size="12" color="grey300">md:list</v-icon>
            {{ information.tests_num }}
          </span>
          <span v-if="information.views" class="metadata-item">
            <v-icon size="12" color="#1E2A44">md:visibility_outlined</v-icon>
            {{ information.views }}
          </span>
          <span
            v-if="information.reply_num && route.query.type == 'question'"
            class="metadata-item"
          >
            <v-icon size="12" color="grey300">md:reply</v-icon>
            {{ information.reply_num }}
          </span>
          <span class="metadata-item">
            <v-icon size="12" color="#1E2A44">md:calendar_month_outlined</v-icon>
            {{ formattedDate }}
          </span>
        </div>

        <div class="card-indicators mobile-indicators d-flex d-md-none align-center">
          <DifficultyIndicator :level="information.level" :size="16" />
          <span class="indicator indicator-library"><img :src="libraryCheckIcon" alt="" class="status-icon"></span>
          <span class="indicator indicator-pdf" :class="{ 'indicator-muted': !information.q_file }"><img :src="pdfCardIcon" alt="" class="status-icon"></span>
          <span class="indicator indicator-word" :class="{ 'indicator-muted': !information.q_file_word }"><img :src="wordCardIcon" alt="" class="status-icon"></span>
          <span class="indicator indicator-fire"><img :src="fireCardIcon" alt="" class="status-icon"></span>
          <QualityIndicator :score="qualityScore" :size="16" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import DifficultyIndicator from './DifficultyIndicator.vue'
import QualityIndicator from './QualityIndicator.vue'
import fireCardIcon from '~/assets/images/search-card/fire.svg'
import libraryCheckIcon from '~/assets/images/search-card/library-check.svg'
import pdfCardIcon from '~/assets/images/search-card/pdf.svg'
import wordCardIcon from '~/assets/images/search-card/word.svg'

const route = useRoute()
const router = useRouter()

const props = defineProps({
  information: {
    type: Object,
    default: () => ({}),
  },
})

const publisherName = computed(() => {
  const name = [props.information.first_name, props.information.last_name]
    .filter(Boolean)
    .join(' ')
    .trim()
  return name || props.information.username || 'GamaTrain'
})

const fallbackSubject = computed(() => {
  const title = String(props.information.lesson_title || '').trim()
  const subjectMatch = title.match(/^(.*?)\s*(\(\d+\))$/)

  return {
    name: subjectMatch?.[1]?.trim() || title,
    code: subjectMatch?.[2] || '',
  }
})

const description = computed(() => String(props.information.description || props.information.summary || '')
  .replace(/<[^>]*>/g, ' ')
  .replace(/&amp;/gi, '&')
  .replace(/&nbsp;/gi, ' ')
  .replace(/&#39;/gi, "'")
  .replace(/&quot;/gi, '"')
  .replace(/\s+/g, ' ')
  .trim())

const qualityScore = computed(() => {
  const score = Number(props.information.referee_score ?? props.information.ref_score ?? 0)
  return Number.isFinite(score) ? Math.min(5, Math.max(0, Math.round(score))) : 0
})

const formattedDate = computed(() => {
  if (!props.information.subdate) return ''
  const date = new Date(props.information.subdate)
  return Number.isNaN(date.getTime()) ? props.information.subdate : date.toLocaleDateString()
})

const getEquivalentOldType = (type) => {
  switch (type) {
    case 'paper':
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
    default:
      return 'test'
  }
}

const createLinkCard = (information) => {
  let idType = ''
  switch (getEquivalentOldType(route.query.type)) {
    case 'test':
      idType = 'paper'
      break
    case 'question':
      idType = 'qa'
      break
    case 'dars':
      idType = 'tutorial'
      break
    case 'azmoon':
      idType = 'exam'
      break
    case 'learnfiles':
      idType = 'multimedia'
      break
    default:
      idType = 'paper'
      break
  }
  return `/${idType}/${information.id}/${information.title_url}`
}

const openCard = (event) => {
  const interactiveElement = event.target instanceof Element
    ? event.target.closest('a, button, input, select, textarea, [role="button"]')
    : null

  if (interactiveElement && interactiveElement !== event.currentTarget) return
  router.push(createLinkCard(props.information))
}
</script>

<style scoped>
.card-search {
  overflow: hidden;
  height: 174px;
  max-width: 1200px;
  cursor: pointer;
  border: 1px solid #d8dee8;
  border-radius: 16px !important;
  background: #fcfcfd;
  box-shadow: 0 1px 2px rgb(30 42 68 / 7%);
  transition: border-color 180ms ease, box-shadow 180ms ease, transform 180ms ease;
}

.card-search:hover {
  border-color: #d8dee8;
  box-shadow: 0 6px 18px rgb(30 42 68 / 12%);
  transform: translateY(-2px);
}

.card-search:focus-visible {
  outline: 3px solid rgb(244 180 0 / 34%);
  outline-offset: 2px;
}

.card-content {
  height: 100%;
  min-height: 0;
  padding: 0;
  color: inherit;
}

.cover-wrap {
  width: auto;
  min-width: 0;
  height: 100%;
  min-height: 100%;
  max-height: 100%;
  aspect-ratio: 63 / 74;
  overflow: hidden;
  border-radius: 15px 0 0 15px;
  background: #fff8e1;
}

.cover-image,
.cover-fallback {
  width: 100%;
  height: 100%;
}

.cover-fallback {
  padding: 12px;
  background: #eef1f5;
  color: rgb(var(--v-theme-grey600));
}

.card-body {
  flex: 1;
  padding: 16px 16px 8px;
}

.min-width-0 { min-width: 0; }

.publisher-avatar { border: 1px solid #d8dee8; }

.card-top { margin-bottom: 8px; }

.publisher-name {
  max-width: 260px;
  color: rgb(30 42 68 / 68%);
  font-size: 13px;
  font-weight: 600;
  line-height: 18px;
}

.card-title {
  max-width: 100%;
  margin: 0 0 12px;
  color: #1e2a44 !important;
  font-size: 18px;
  font-weight: 650 !important;
  line-height: 26px;
}

.card-description {
  max-width: 100%;
  margin: 0;
  color: rgb(30 42 68 / 68%) !important;
  font-size: 13px;
  line-height: 20px;
}

.subject-tags {
  gap: 4px !important;
  margin-block: 8px;
}

.tag-chip {
  z-index: 2;
  height: 24px;
  padding: 4px 8px !important;
  color: rgb(30 42 68 / 68%) !important;
  background: #f7f8fa !important;
  border: 1px solid #eef1f5;
  border-radius: 6px !important;
  font-size: 11px;
  line-height: 16px;
}

.tag-chip:hover {
  background: #eef1f5 !important;
  border-color: #d8dee8;
}

.tag-chip :deep(.v-chip__content) {
  color: inherit;
}

.metadata {
  gap: 12px !important;
  min-height: 18px;
  padding-top: 0;
  color: rgb(30 42 68 / 68%);
  font-size: 12px;
  line-height: 18px;
}

.metadata-item {
  display: inline-flex;
  align-items: center;
  gap: 4px;
}

.card-indicators { gap: 8px; }

.indicator {
  display: inline-flex;
  width: 16px;
  height: 16px;
  align-items: center;
  justify-content: center;
}

.status-icon {
  display: block;
  width: 100%;
  height: 100%;
}

.indicator-muted { opacity: 0.32; }

.mobile-indicators {
  gap: 8px;
  margin-top: 10px;
}

@media (min-width: 960px) {
  .card-indicators { gap: 24px; }
}

@media (min-width: 1280px) {
  .metadata {
    gap: 20px !important;
  }
}

@media (max-width: 959px) {
  .card-search { height: 156px; }
}

@media (max-width: 599px) {
  .card-search { height: 174px; }

  .card-body {
    padding: 10px 12px 8px;
  }

  .card-top {
    flex-shrink: 0;
    margin-bottom: 4px;
  }

  .publisher-avatar {
    width: 26px !important;
    height: 26px !important;
  }

  .publisher-name {
    max-width: 150px;
    font-size: 12px;
  }

  .card-title {
    flex-shrink: 0;
    margin-bottom: 2px;
    font-size: 14px;
    line-height: 20px;
  }

  .card-description {
    flex-shrink: 0;
    font-size: 12px;
    line-height: 18px;
  }

  .subject-tags {
    flex-wrap: nowrap !important;
    flex-shrink: 0;
    margin-block: 4px;
    overflow: hidden;
  }

  .metadata {
    flex-shrink: 0;
    gap: 8px !important;
  }

  .mobile-indicators {
    flex-shrink: 0;
    margin-top: 4px;
  }
}
</style>
