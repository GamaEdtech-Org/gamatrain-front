<template>
  <v-badge
    class="height-badge d-inline-flex align-center justify-center"
    offset-x="5"
    offset-y="-5"
    :color="count == 0 ? 'transparent' : 'lightError'"
    :content="count == 0 ? '' : count"
  >
    <v-btn
      rounded="xl"
      variant="outlined"
      color="grey300"
      density="comfortable"
      @click="emit('click')"
    >
      <span class="text-grey700 d-flex align-center ga-2 text-h6 font-weight-bold">
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
:deep(.height-badge .v-badge__wrapper .v-badge__badge) {
  height: 20px !important;
}
</style>
