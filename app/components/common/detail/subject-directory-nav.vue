<template>
  <nuxt-link
    :to="`/subject-directory?board=${contentData?.section}&grade=${contentData?.base}&subject=${contentData?.lesson}`"
    class="subject-directory-card"
  >
    <div class="subject-directory-card__main">
      <span
        class="subject-directory-card__icon"
        aria-hidden="true"
      >
        <svg
          viewBox="0 0 24 24"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            d="M5 4.5V19.5M9.5 4.5V19.5M14 5L16.5 19.5M4 19.5H18"
            stroke="currentColor"
            stroke-width="1.5"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </span>

      <div class="subject-directory-card__copy">
        <span class="subject-directory-card__title">
          {{ contentData?.lesson_title }} Subject directory
        </span>
        <span class="subject-directory-card__subtitle">
          All resources in one place.
        </span>
      </div>
    </div>

    <span class="subject-directory-card__action">
      Open directory <span aria-hidden="true">→</span>
    </span>
  </nuxt-link>
</template>

<script setup>
defineProps({
  contentData: Object,
})
</script>

<style scoped>
.subject-directory-card {
  display: flex;
  width: 100%;
  min-height: 0;
  align-items: center;
  justify-content: space-between;
  gap: 24px;
  padding: 8px 20px;
  margin-top: 0;
  color: #111827;
  text-decoration: none;
  background: #f2f9f8;
  border: 1px solid #397f7b;
  border-radius: 16px;
  transition: transform 200ms ease-out, border-color 200ms ease-out;
}

.subject-directory-card__main {
  display: flex;
  min-width: 0;
  align-items: center;
  gap: 16px;
}

.subject-directory-card__icon {
  display: inline-flex;
  width: 46px;
  height: 46px;
  flex: 0 0 46px;
  align-items: center;
  justify-content: center;
  color: #ffffff;
  background: #7dbfba;
  border-radius: 12px;
  transition: background-color 200ms ease-out;
}

@media (hover: hover) and (pointer: fine) {
  .subject-directory-card:hover {
    transform: scale(1.02);
    border-color: #2d6662;
  }

  .subject-directory-card:hover .subject-directory-card__icon {
    background-color: #5a9f99;
  }
}

@media (prefers-reduced-motion: reduce) {
  .subject-directory-card,
  .subject-directory-card__icon {
    transition: none;
  }
}

.subject-directory-card__icon svg {
  width: 24px;
  height: 24px;
}

.subject-directory-card__copy {
  display: flex;
  min-width: 0;
  flex-direction: column;
  gap: 2px;
}

.subject-directory-card__title {
  overflow: hidden;
  color: #0b0b0b;
  font-size: 20px;
  font-weight: 700;
  line-height: 28px;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.subject-directory-card__subtitle {
  color: #8a94a6;
  font-size: 17px;
  font-weight: 400;
  line-height: 24px;
}

.subject-directory-card__action {
  display: inline-flex;
  min-height: 36px;
  flex: 0 0 auto;
  align-items: center;
  gap: 8px;
  padding: 6px 12px;
  color: #111111;
  font-size: 16px;
  font-weight: 400;
  line-height: 22px;
  background: #ffffff;
  border: 1px solid #d9d9d9;
  border-radius: 14px;
}

@media (max-width: 600px) {
  .subject-directory-card {
    align-items: flex-start;
    flex-wrap: wrap;
    gap: 14px;
    padding: 8px 16px;
  }

  .subject-directory-card__main {
    width: 100%;
  }

  .subject-directory-card__title {
    font-size: 18px;
    line-height: 24px;
    white-space: normal;
  }

  .subject-directory-card__subtitle {
    font-size: 15px;
    line-height: 21px;
  }

  .subject-directory-card__action {
    margin-left: 62px;
  }
}
</style>
