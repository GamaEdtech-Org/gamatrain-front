<template>
  <section
    id="search-category-filters"
    class="category-filter-container"
    aria-label="Filters for the selected category"
  >
    <slot />
  </section>
</template>

<style scoped>
.category-filter-container {
  box-sizing: border-box;
  display: flex;
  width: 100%;
  max-width: 1200px;
  min-width: 0;
  flex-wrap: wrap;
  justify-content: center;
  padding: 16px;
  border: 0;
  background: rgb(var(--v-theme-surface));
}
</style>
