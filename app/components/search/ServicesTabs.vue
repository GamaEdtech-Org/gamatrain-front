<template>
  <nav
    class="services-navigation"
    aria-label="Search services"
  >
    <div class="services-navigation__items">
      <div
        v-for="service in services"
        :key="service.id"
        class="services-navigation__tab"
      >
      <v-btn
        class="services-navigation__item"
        :class="{ 'services-navigation__item--active': activeService === service.id }"
        :aria-pressed="activeService === service.id"
        aria-controls="search-service-filters"
        variant="outlined"
        @click="emit('change', service.id)"
      >
        <span
          class="services-navigation__icon"
          aria-hidden="true"
        >
          <img
            v-if="service.image"
            :src="service.image"
            alt=""
          >
          <span
            v-else
            :class="service.icon"
          />
        </span>
        <span class="services-navigation__copy">
          <span class="services-navigation__count">
            {{ formatCount(service.id) }}
          </span>
          <span class="services-navigation__title">{{ service.title }}</span>
        </span>
      </v-btn>
      </div>
    </div>
  </nav>
</template>

<script setup>
const props = defineProps({
  activeService: {
    type: String,
    default: 'paper',
  },
  serviceCounts: {
    type: Object,
    default: () => ({}),
  },
})

const emit = defineEmits(['change'])

const services = [
  { id: 'paper', title: 'Past Papers', icon: 'stat-icon icon-paper' },
  { id: 'study-materials', title: 'Study Materials', image: '/images/study-materials.svg' },
  { id: 'quizhub', title: 'Exam Hub', icon: 'stat-icon icon-exam' },
  { id: 'tutorial', title: 'Tutorial', icon: 'stat-icon icon-tutorial' },
]

const formatCount = (serviceId) => {
  const count = props.serviceCounts[serviceId]
  return count == null ? '—' : new Intl.NumberFormat().format(count)
}
</script>

<style scoped>
.services-navigation {
  width: 100%;
  max-width: 1200px;
  min-width: 0;
  padding: 0;
  border-bottom: 1px solid #d8dee8;
  overflow-x: auto;
  scrollbar-width: none;
}

.services-navigation::-webkit-scrollbar {
  display: none;
}

.services-navigation__items {
  display: flex;
  width: max-content;
  gap: 8px;
  margin-inline: 0;
}

.services-navigation__tab {
  position: relative;
  display: flex;
  flex: 0 0 216px;
}

.services-navigation__item {
  width: 216px;
  height: 72px !important;
  flex: 0 0 216px;
  padding: 0 20px;
  border: 0;
  border-radius: 12px 12px 0 0;
  color: #1e2a44;
  background: #fcfcfd;
  font-size: 15px;
  font-weight: 600;
  letter-spacing: normal;
  text-transform: none;
  transition: background-color 180ms ease, color 180ms ease, box-shadow 180ms ease;
}

.services-navigation__item--active {
  color: #ffffff;
  background-color: #1e2a44;
  box-shadow: inset 0 -3px 0 #f4b400;
}

.services-navigation__item:not(.services-navigation__item--active):hover {
  background: #f7f8fa;
}

.services-navigation__item:focus-visible {
  outline: 3px solid rgb(244 180 0 / 30%);
  outline-offset: -3px;
}

.services-navigation__item :deep(.v-btn__content) {
  width: 100%;
  justify-content: flex-start;
  gap: 8px;
}

.services-navigation__icon {
  box-sizing: border-box;
  display: inline-flex;
  width: 36px;
  height: 36px;
  flex: 0 0 36px;
  align-items: center;
  justify-content: center;
  padding: 0;
  margin: 0;
  overflow: hidden;
  font-size: 36px;
  letter-spacing: 0;
  line-height: 1;
  text-indent: 0;
  color: #1e2a44;
}

.services-navigation__icon img {
  box-sizing: border-box;
  width: 36px;
  height: 36px;
  padding: 0;
  object-fit: contain;
}

.services-navigation__icon > span {
  box-sizing: border-box;
  display: inline-flex;
  width: 36px;
  height: 36px;
  align-items: center;
  justify-content: center;
  padding: 0;
  margin: 0;
  overflow: hidden;
  font-size: 36px;
  letter-spacing: 0;
  line-height: 36px;
  text-indent: 0;
}

.services-navigation__icon > span::before {
  box-sizing: border-box;
  display: block;
  width: 36px;
  height: 36px;
  padding: 0;
  margin: 0;
  overflow: hidden;
  font-size: 36px;
  letter-spacing: 0;
  line-height: 36px;
  text-indent: 0;
}

.services-navigation__item--active .services-navigation__icon img {
  filter: brightness(0) invert(1);
}

.services-navigation__item--active .services-navigation__icon {
  color: #ffffff;
}

.services-navigation__icon img {
  filter: brightness(0) saturate(100%) invert(12%) sepia(7%) saturate(1051%) hue-rotate(169deg) brightness(93%) contrast(90%);
}

.services-navigation__copy {
  display: flex;
  min-width: 0;
  flex-direction: column;
  align-items: flex-start;
  line-height: 1.15;
}

.services-navigation__count {
  min-height: 16px;
  color: #f4b400;
  font-size: 14px;
  font-weight: 700;
  line-height: 18px;
}

.services-navigation__item:not(.services-navigation__item--active) .services-navigation__count {
  color: rgb(30 42 68 / 68%);
}

.services-navigation__title {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  line-height: 20px;
}

@media (min-width: 960px) {
  .services-navigation {
    overflow: visible;
  }

  .services-navigation__items {
    width: 100%;
    max-width: 836px;
  }

  .services-navigation__tab {
    flex: 1 1 0;
    min-width: 0;
  }

  .services-navigation__item {
    width: 100%;
    min-width: 0;
  }
}
</style>
