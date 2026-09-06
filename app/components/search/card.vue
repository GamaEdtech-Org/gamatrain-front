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
          <span class="font-weight-bold">{{ information.lesson_title }}</span>
          <small>Gamatrain.com</small>
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
            <v-icon size="12" color="grey300">md:folder_outlined</v-icon>
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
            <v-icon size="12" color="grey300">md:visibility_outlined</v-icon>
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
            <v-icon size="12" color="grey300">md:calendar_month_outlined</v-icon>
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
  border: 1px solid rgb(var(--v-theme-grey200));
  background: #fff;
  transition: border-color 180ms ease, box-shadow 180ms ease, transform 180ms ease;
}

.card-search:hover {
  border-color: #7dbfba;
  box-shadow: 0 8px 24px rgb(57 127 123 / 10%);
  transform: translateY(-1px);
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
  border-radius: 23px 0 0 23px;
  background: #f9f3f3;
}

.cover-image,
.cover-fallback {
  width: 100%;
  height: 100%;
}

.cover-fallback {
  padding: 12px;
  color: rgb(var(--v-theme-grey600));
}

.card-body {
  flex: 1;
  padding: 16px 16px 8px;
}

.min-width-0 { min-width: 0; }

.publisher-avatar { border: 1px solid rgb(var(--v-theme-grey100)); }

.card-top { margin-bottom: 12px; }

.publisher-name {
  max-width: 260px;
  color: rgb(var(--v-theme-grey700));
  font-size: 12px;
  font-weight: 500;
  line-height: 18px;
}

.card-title {
  max-width: 100%;
  margin: 0 0 12px;
  font-size: 14px;
  line-height: 24px;
}

.card-description {
  max-width: 100%;
  margin: 0;
  font-size: 12px;
  line-height: 18px;
}

.subject-tags {
  gap: 4px !important;
  margin-block: 8px;
}

.tag-chip {
  z-index: 2;
  height: 22px;
  padding: 4px !important;
  border-radius: 4px !important;
  font-size: 10px;
  line-height: 14px;
}

.metadata {
  gap: 8px !important;
  min-height: 18px;
  padding-top: 0;
  font-size: 10px;
  line-height: 15px;
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
  .card-title { font-size: 18px; }

  .metadata {
    gap: 40px !important;
    font-size: 12px;
    line-height: 18px;
  }
}

@media (max-width: 959px) {
  .card-search { height: 156px; }
}

@media (max-width: 599px) {
  .card-search { height: 134px; }

  .publisher-avatar {
    width: 26px !important;
    height: 26px !important;
  }

  .publisher-name {
    max-width: 150px;
    font-size: 12px;
  }

  .card-title {
    font-size: 14px;
    line-height: 24px;
  }

  .card-description {
    font-size: 12px;
    line-height: 18px;
  }

  .subject-tags {
    flex-wrap: nowrap !important;
    overflow: hidden;
  }

  .metadata { gap: 8px !important; }
}
</style>
