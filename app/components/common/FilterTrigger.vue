<template>
  <v-badge
    class="height-badge d-inline-flex align-center justify-center"
    offset-x="5"
    offset-y="-5"
    :color="count == 0 ? 'transparent' : 'lightError'"
    :content="count == 0 ? '' : count"
  >
    <v-btn
      class="filter-trigger"
      rounded="xl"
      variant="outlined"
      density="comfortable"
      @click="emit('click')"
    >
      <span class="filter-trigger__content d-flex align-center ga-2 font-weight-bold">
        <v-icon>md:tune</v-icon>
        Filter
      </span>
    </v-btn>
  </v-badge>
</template>

<script setup>
defineProps({
  count: {
    type: [Number, String],
    default: 0,
  },
})

const emit = defineEmits(['click'])
</script>

<style scoped>
.filter-trigger {
  color: #171633;
  background: #ffffff;
  border-color: #dcdde5;
  border-radius: 12px !important;
}

.filter-trigger:hover {
  background: #fff3c4;
  border-color: #dfb536;
}

.filter-trigger__content {
  color: #202238;
  font-size: 14px;
  line-height: 20px;
}

:deep(.height-badge .v-badge__wrapper .v-badge__badge) {
  height: 20px !important;
}
</style>
