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
  padding: 12px 16px;
  margin-top: 0;
  color: #202238;
  text-decoration: none;
  background: #f7f7f4;
  border: 1px solid #171633;
  border-radius: 12px;
  box-shadow: 0 1px 2px rgb(36 41 47 / 8%);
  transition: transform 180ms ease-out, box-shadow 180ms ease-out;
}

.subject-directory-card__main {
  display: flex;
  min-width: 0;
  align-items: center;
  gap: 16px;
}

.subject-directory-card__icon {
  display: inline-flex;
  width: 44px;
  height: 44px;
  flex: 0 0 44px;
  align-items: center;
  justify-content: center;
  color: #ffffff;
  background: #171633;
  border-radius: 10px;
}

@media (hover: hover) and (pointer: fine) {
  .subject-directory-card:hover {
    transform: translateY(-1px);
    border-color: #dfb536;
    box-shadow: 0 6px 18px rgb(36 41 47 / 14%);
  }
}

@media (prefers-reduced-motion: reduce) {
  .subject-directory-card,
  .subject-directory-card__icon {
    transition: none;
  }
}

.subject-directory-card__icon svg {
  width: 22px;
  height: 22px;
}

.subject-directory-card__copy {
  display: flex;
  min-width: 0;
  flex-direction: column;
  gap: 2px;
}

.subject-directory-card__title {
  overflow: hidden;
  color: #202238;
  font-size: 18px;
  font-weight: 700;
  line-height: 24px;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.subject-directory-card__subtitle {
  color: #65697a;
  font-size: 14px;
  font-weight: 400;
  line-height: 20px;
}

.subject-directory-card__action {
  display: inline-flex;
  min-height: 40px;
  flex: 0 0 auto;
  align-items: center;
  gap: 8px;
  padding: 8px 14px;
  color: #171633;
  font-size: 14px;
  font-weight: 600;
  line-height: 20px;
  background: #f2c94c;
  border: 1px solid #f2c94c;
  border-radius: 10px;
}

.subject-directory-card:hover .subject-directory-card__action {
  background: #dfb536;
  border-color: #dfb536;
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
    font-size: 16px;
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
