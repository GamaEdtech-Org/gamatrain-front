<template>
  <div class="publish-options w-100 d-flex align-center justify-center flex-column">
    <div class="container-card w-100">
      <v-btn
        v-for="item in addOptions"
        :key="item.title"
        :to="item.path"
        :disabled="item.disabled"
        :aria-label="`${item.title} — ${item.typeFile}`"
        variant="plain"
        class="card-add-option"
      >
        <div class="publish-option-content w-100 d-flex flex-column align-start justify-start">
          <div class="icon-div d-flex align-center justify-center">
            <span
              v-if="item.icon"
              class="icon-add text-grey700"
              :class="item.icon"
            />
            <v-icon
              v-if="item.iconMd"
              color="#1E2A44"
              size="20"
            >
              {{ item.iconMd }}
            </v-icon>
          </div>
          <span class="card-option-title w-100">{{ item.title }}</span>

          <span class="chip-type-file">
            {{ item.typeFile }}
          </span>
        </div>
      </v-btn>
    </div>

    <div class="info-card w-100 d-flex align-center justify-start">
      <div class="info-card__icon d-flex align-center justify-center">
        <v-icon
          color="#1E2A44"
          size="16"
        >
          md:star
        </v-icon>
      </div>
      <div class="info-card__copy d-flex flex-column align-start justify-start">
        <span class="info-card__title">Turn your expertise into reputation and income</span>
        <span class="info-card__description">Publish educational content to boost your score and earn from sales.</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { DEFAULT_BOARD_ID, MULTIMEDIA_CONTENT_TYPE_IDS, PAPER_CLASSIFICATION_IDS } from '@/constants'

const route = useRoute()
const emit = defineEmits(['close'])

interface AddOption {
  path: string
  title: string
  typeFile: string
  icon?: string
  iconMd?: string
  disabled: boolean
}

const { user } = useUser()
const userBoardId = computed(() => user.value?.board ?? DEFAULT_BOARD_ID)
const canAddEducationalContent = computed(() => user.value?.group === 5)

const addOptions = computed<AddOption[]>(() => [
  {
    path: `/user/paper/create?board=${userBoardId.value}&classification=${PAPER_CLASSIFICATION_IDS.WORKSHEET}`,
    title: 'Worksheet',
    iconMd: 'md:description_outlined',
    typeFile: 'PDF · DOCX',
    disabled: !canAddEducationalContent.value,
  },
  {
    path: `/user/paper/create?board=${userBoardId.value}&classification=${PAPER_CLASSIFICATION_IDS.PREDICTED_PAPER}`,
    title: 'Predicted Paper',
    icon: 'icon-paper',
    typeFile: 'PDF · DOCX',
    disabled: !canAddEducationalContent.value,
  },
  {
    path: `/user/paper/create?board=${userBoardId.value}&classification=${PAPER_CLASSIFICATION_IDS.STUDY_GUIDE}`,
    title: 'Study Guide',
    iconMd: 'md:menu_book',
    typeFile: 'PDF · DOCX',
    disabled: !canAddEducationalContent.value,
  },
  {
    path: `/user/paper/create?board=${userBoardId.value}&classification=${PAPER_CLASSIFICATION_IDS.TOPICAL_QUESTIONS}`,
    title: 'Topical Questions',
    iconMd: 'md:quiz_outlined',
    typeFile: 'PDF · DOCX',
    disabled: !canAddEducationalContent.value,
  },
  {
    path: `/user/multimedia/create?contentType=${MULTIMEDIA_CONTENT_TYPE_IDS.VIDEO}`,
    title: 'Video',
    iconMd: 'md:videocam',
    typeFile: 'MP4',
    disabled: !canAddEducationalContent.value,
  },
  {
    path: `/user/multimedia/create?contentType=${MULTIMEDIA_CONTENT_TYPE_IDS.PRESENTATION}`,
    title: 'Presentation',
    iconMd: 'md:slideshow',
    typeFile: 'PPTX',
    disabled: !canAddEducationalContent.value,
  },
  {
    path: '/school/add',
    title: 'School',
    icon: 'icon-school',
    typeFile: 'INFO',
    disabled: false,
  },
  {
    path: '/user/question/create',
    title: 'Q&A',
    icon: 'icon-q-a',
    typeFile: 'TEXT',
    disabled: false,
  },
  {
    path: '/user/blogs/create',
    title: 'Blogs',
    iconMd: 'md:art_track',
    typeFile: 'HTML',
    disabled: false,
  },
])

watch(
  () => route.fullPath,
  () => {
    emit('close')
  },
)
</script>

<style scoped>
.publish-options {
  gap: 24px;
  padding-top: 24px;
}

.container-card {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 16px;
}

.card-add-option {
  width: 100%;
  min-width: 0;
  min-height: 132px;
  height: auto !important;
  padding: 16px !important;
  color: #1e2a44;
  background: #fff;
  border: 1px solid #d8dee8;
  border-radius: 14px;
  box-shadow: 0 2px 8px rgb(30 42 68 / 5%);
  text-decoration: none;
  opacity: 1;
  transition: transform 200ms ease, border-color 200ms ease, box-shadow 200ms ease, background-color 200ms ease;
}

.card-add-option :deep(.v-btn__content) {
  width: 100%;
  height: 100%;
  display: flex;
  white-space: normal !important;
}

.publish-option-content {
  gap: 10px;
  text-align: left;
}

.card-add-option:not(.v-btn--disabled):hover {
  transform: translateY(-2px);
  border-color: #e8a800;
  box-shadow: 0 10px 24px rgb(30 42 68 / 12%);
}

.card-add-option:not(.v-btn--disabled):active {
  transform: scale(.99);
}

.card-add-option:focus-visible {
  outline: 3px solid rgb(244 180 0 / 32%);
  outline-offset: 2px;
}

.card-add-option.v-btn--active {
  background: #fff8e1;
  border-color: #f4b400;
  box-shadow: 0 0 0 1px #f4b400, 0 8px 20px rgb(30 42 68 / 10%);
}

.card-add-option.v-btn--active::after {
  position: absolute;
  top: 10px;
  right: 10px;
  display: grid;
  width: 22px;
  height: 22px;
  color: #1e2a44;
  font-size: 14px;
  font-weight: 800;
  content: '\2713';
  background: #f4b400;
  border-radius: 50%;
  place-items: center;
}

.card-add-option.v-btn--disabled {
  cursor: not-allowed;
  opacity: .48;
}

.card-add-option.v-btn--disabled:hover {
  transform: none;
  border-color: #d8dee8;
  box-shadow: 0 2px 8px rgb(30 42 68 / 5%);
}

.icon-div {
  width: 40px;
  height: 40px;
  flex: 0 0 40px;
  color: #1e2a44;
  background: #fff8e1;
  border-radius: 10px;
}

.icon-add {
  font-size: 22px;
}

.card-option-title {
  color: #1e2a44;
  font-size: 15px;
  font-weight: 700;
  line-height: 1.35;
}

.chip-type-file {
  display: inline-flex;
  align-items: center;
  min-height: 22px;
  padding: 2px 8px;
  color: #667085;
  font-size: 11px;
  font-weight: 700;
  line-height: 1.2;
  background: #f7f8fa;
  border: 1px solid #d8dee8;
  border-radius: 999px;
}

.info-card {
  gap: 12px;
  padding: 14px 16px;
  background: #f7f8fa;
  border: 1px solid #d8dee8;
  border-radius: 14px;
}

.info-card__icon {
  width: 36px;
  height: 36px;
  flex: 0 0 36px;
  background: #fff8e1;
  border-radius: 10px;
}

.info-card__copy {
  gap: 2px;
}

.info-card__title {
  color: #1e2a44;
  font-size: 14px;
  font-weight: 700;
  line-height: 1.35;
}

.info-card__description {
  color: #667085;
  font-size: 12px;
  font-weight: 500;
  line-height: 1.45;
}

@media only screen and (min-width: 768px) and (max-width: 1023px) {
  .publish-options {
    gap: 20px;
    padding-top: 20px;
  }

  .container-card {
    gap: 12px;
  }

  .card-add-option {
    min-height: 124px;
    padding: 14px !important;
  }
}

@media only screen and (max-width: 767px) {
  .publish-options {
    gap: 16px;
    padding-top: 20px;
  }

  .container-card {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 12px;
  }

  .card-add-option {
    min-height: 116px;
    padding: 12px !important;
  }

  .publish-option-content {
    gap: 8px;
  }

  .info-card {
    align-items: flex-start !important;
    padding: 12px;
  }
}

@media only screen and (max-width: 359px) {
  .container-card {
    gap: 8px;
  }

  .card-option-title {
    font-size: 14px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .card-add-option {
    transition: none;
  }
}
</style>
