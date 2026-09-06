<template>
  <div class="card-skeleton w-100 rounded-xl d-flex align-stretch">
    <v-skeleton-loader
      type="image"
      class="skeleton-cover flex-shrink-0"
    />
    <div class="skeleton-body d-flex flex-column flex-grow-1">
      <div class="d-flex justify-space-between align-center ga-4">
        <div class="d-flex align-center ga-2">
          <v-skeleton-loader type="avatar" width="34" height="34" />
          <v-skeleton-loader type="text" width="120" />
        </div>
        <v-skeleton-loader
          class="d-none d-md-block"
          type="text"
          width="250"
        />
      </div>
      <v-skeleton-loader type="heading" width="55%" height="30" />
      <v-skeleton-loader type="text" width="82%" />
      <div class="d-flex ga-2">
        <v-skeleton-loader type="chip" width="86" />
        <v-skeleton-loader type="chip" width="86" />
        <v-skeleton-loader type="chip" width="120" />
      </div>
      <v-skeleton-loader type="text" width="340" class="mt-auto" />
    </div>
  </div>
</template>

<style scoped>
.card-skeleton {
  min-height: 168px;
  max-width: 1200px;
  padding: 12px;
  border: 1px solid rgb(var(--v-theme-grey200));
  background: #fff;
}

.skeleton-cover {
  width: 126px;
  min-width: 126px;
  min-height: 144px;
  overflow: hidden;
  border-radius: 10px;
}

.skeleton-body {
  gap: 6px;
  min-width: 0;
  padding: 2px 8px 2px 16px;
}

@media (max-width: 599px) {
  .card-skeleton {
    min-height: 134px;
    padding: 10px;
  }

  .skeleton-cover {
    width: 88px;
    min-width: 88px;
    min-height: 112px;
  }

  .skeleton-body { padding-left: 10px; }
}
</style>
