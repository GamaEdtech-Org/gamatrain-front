<template>
  <section
    id="search-service-filters"
    class="services-filter-container"
    aria-label="Filters for the selected service"
  >
    <slot />
  </section>
</template>

<style scoped>
.services-filter-container {
  box-sizing: border-box;
  display: flex;
  width: 100%;
  max-width: 1200px;
  min-width: 0;
  flex-wrap: wrap;
  justify-content: center;
  padding: 0 20px 16px;
  border: 0;
  background: transparent;
}

.services-filter-container :deep(.search-filter-control.open-style-btn:not(.search-filter-empty)),
.services-filter-container :deep(.search-filter-control.dependent-selected-btn),
.services-filter-container :deep(.search-filter-control.open-style-btn:not(.search-filter-empty):hover),
.services-filter-container :deep(.search-filter-control.dependent-selected-btn:hover) {
  background-color: #eef1f5 !important;
  border-color: transparent !important;
}
</style>
