<template>
  <nav
    class="category-navigation"
    aria-label="Search categories"
  >
    <div class="category-navigation__items">
      <div
        v-for="category in categories"
        :key="category.id"
        class="category-navigation__tab"
      >
      <v-btn
        class="category-navigation__item"
        :class="{ 'category-navigation__item--active': activeCategory === category.id }"
        :style="{ '--category-color': category.color }"
        :aria-pressed="activeCategory === category.id"
        aria-controls="search-category-filters"
        variant="outlined"
        @click="emit('change', category.id)"
      >
        <span
          class="category-navigation__icon"
          :style="{ color: activeCategory === category.id ? 'inherit' : category.color }"
          aria-hidden="true"
        >
          <img
            v-if="category.image"
            :src="category.image"
            alt=""
          >
          <span
            v-else
            :class="category.icon"
          />
        </span>
        <span>{{ category.title }}</span>
      </v-btn>
      </div>
    </div>
  </nav>
</template>

<script setup>
defineProps({
  activeCategory: {
    type: String,
    default: 'paper',
  },
})

const emit = defineEmits(['change'])

const categories = [
  { id: 'paper', title: 'Past Papers', icon: 'stat-icon icon-paper', color: '#2e90fa' },
  { id: 'study-materials', title: 'Study Materials', image: '/images/study-materials.svg', color: 'rgb(18, 183, 106)' },
  { id: 'quizhub', title: 'Exam Hub', icon: 'stat-icon icon-exam', color: '#7c4dff' },
  { id: 'tutorial', title: 'Tutorial', icon: 'stat-icon icon-tutorial', color: '#2e90fa' },
]
</script>

<style scoped>
.category-navigation {
  width: 100%;
  max-width: 1200px;
  min-width: 0;
  padding: 12px 0 0;
  border-bottom: 1px solid color-mix(in srgb, var(--search-category-color, #2e90fa) 50%, rgb(var(--v-theme-surface)));
  overflow-x: auto;
  scrollbar-width: none;
}

.category-navigation::-webkit-scrollbar {
  display: none;
}

.category-navigation__items {
  display: flex;
  width: max-content;
  gap: 12px;
  margin-inline: auto;
}

.category-navigation__tab {
  position: relative;
  display: flex;
  flex: 0 0 200px;
}

.category-navigation__item {
  width: 200px;
  height: 52px !important;
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

.category-navigation__item--active {
  color: white;
  background-color: var(--category-color);
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
}

.category-navigation__item :deep(.v-btn__content) {
  width: 100%;
  justify-content: flex-start;
  gap: 8px;
}

.category-navigation__icon {
  display: inline-flex;
  width: 28px;
  height: 28px;
  flex: 0 0 28px;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  line-height: 1;
}

.category-navigation__icon img {
  box-sizing: border-box;
  width: 100%;
  height: 100%;
  padding: 3px;
  object-fit: contain;
}

.category-navigation__item--active .category-navigation__icon img {
  filter: brightness(0) invert(1);
}

@media (min-width: 960px) {
  .category-navigation {
    overflow: visible;
  }

  .category-navigation__items {
    width: 100%;
    max-width: 836px;
  }

  .category-navigation__tab {
    flex: 1 1 0;
    min-width: 0;
  }

  .category-navigation__item {
    width: 100%;
    min-width: 0;
  }
}
</style>
