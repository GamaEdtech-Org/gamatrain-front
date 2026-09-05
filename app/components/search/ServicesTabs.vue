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
        :style="{ '--service-color': service.color }"
        :aria-pressed="activeService === service.id"
        aria-controls="search-service-filters"
        variant="outlined"
        @click="emit('change', service.id)"
      >
        <span
          class="services-navigation__icon"
          :style="{ color: activeService === service.id ? 'inherit' : service.color }"
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
  { id: 'paper', title: 'Past Papers', icon: 'stat-icon icon-paper', color: '#2e90fa' },
  { id: 'study-materials', title: 'Study Materials', image: '/images/study-materials.svg', color: 'rgb(18, 183, 106)' },
  { id: 'quizhub', title: 'Exam Hub', icon: 'stat-icon icon-exam', color: '#7c4dff' },
  { id: 'tutorial', title: 'Tutorial', icon: 'stat-icon icon-tutorial', color: '#2e90fa' },
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
  padding: 12px 0 0;
  border-bottom: 1px solid color-mix(in srgb, var(--search-service-color, #2e90fa) 50%, rgb(var(--v-theme-surface)));
  overflow-x: auto;
  scrollbar-width: none;
}

.services-navigation::-webkit-scrollbar {
  display: none;
}

.services-navigation__items {
  display: flex;
  width: max-content;
  gap: 12px;
  margin-inline: 0;
}

.services-navigation__tab {
  position: relative;
  display: flex;
  flex: 0 0 200px;
}

.services-navigation__item {
  width: 200px;
  height: 64px !important;
  flex: 0 0 200px;
  padding: 0 16px;
  border: 0;
  border-radius: 8px;
  color: rgb(var(--v-theme-grey700));
  background: rgb(var(--v-theme-surface));
  font-size: 14px;
  font-weight: 600;
  letter-spacing: normal;
  text-transform: none;
}

.services-navigation__item--active {
  color: white;
  background-color: var(--service-color);
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
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

.services-navigation__copy {
  display: flex;
  min-width: 0;
  flex-direction: column;
  align-items: flex-start;
  line-height: 1.15;
}

.services-navigation__count {
  min-height: 16px;
  font-size: 13px;
  font-weight: 700;
}

.services-navigation__item:not(.services-navigation__item--active) .services-navigation__count {
  color: rgb(var(--v-theme-grey500));
}

.services-navigation__title {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
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
