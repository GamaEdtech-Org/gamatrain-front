<template>
  <v-dialog
    v-model="dialogModel"
    :max-width="maxWidth"
    :fullscreen="variant === 'publish' ? width <= 767 : !mdAndUp"
    @click="clickOnOverlay"
  >
    <div
      class="w-100 d-flex flex-column bg-white rounded-xl overflow-y-auto mobile-style py-6 px-3 px-sm-6"
      :class="{ 'modal-panel--publish': variant === 'publish' }"
      @click="clickOnModal"
    >
      <div class="modal-header w-100 d-flex align-center justify-space-between">
        <div class="d-flex flex-column align-start justify-start ga-1">
          <span class="modal-title text-h5 text-sm-h3 font-weight-bold text-grey700">{{ title }}</span>
          <span
            v-if="subtitle"
            class="text-subtitle-1 text-sm-h6 font-weight-medium text-grey400"
          >{{ subtitle }}</span>
        </div>
        <v-icon
          class="ml-4"
          :class="{ 'modal-close--publish': variant === 'publish' }"
          size="x-large"
          :color="variant === 'publish' ? undefined : 'grey300'"
          aria-label="Close dialog"
          @click="closeModal"
        >
          md:cancel
        </v-icon>
      </div>

      <div class="modal-body w-100 d-flex">
        <slot />
      </div>
    </div>
  </v-dialog>
</template>

<script setup lang="ts">
import { useDisplay } from 'vuetify'

interface IModalBase {
  title: string
  subtitle?: string
  showDialog?: boolean
  maxWidth?: number
  variant?: 'default' | 'publish'
}

const props = withDefaults(defineProps<IModalBase>(), {
  showDialog: false,
  maxWidth: 400,
  subtitle: '',
  variant: 'default',
})

const { mdAndUp, width } = useDisplay()

const emit = defineEmits(['update:showDialog'])

const dialogModel = computed({
  get: () => props.showDialog,
  set: value => emit('update:showDialog', value),
})

const closeModal = () => {
  emit('update:showDialog', false)
}

const clickOnOverlay = () => {
  if (!mdAndUp.value) {
    emit('update:showDialog', false)
  }
}

const clickOnModal = (event: MouseEvent) => {
  event.stopPropagation()
}
</script>

<style scoped>
.mobile-style{
  max-height: 90%;
}
.modal-title {
  line-height: 1.3 !important;
  padding-bottom: 2px;
}

.modal-panel--publish {
  width: 100%;
  max-height: min(90vh, 860px);
  padding: 32px !important;
  color: #1e2a44;
  background: #fcfcfd !important;
  border: 1px solid #d8dee8;
  border-radius: 20px !important;
  box-shadow: 0 24px 64px rgb(30 42 68 / 18%);
}

.modal-panel--publish .modal-header {
  gap: 24px;
  padding-bottom: 8px;
}

.modal-panel--publish .modal-title {
  color: #1e2a44 !important;
  font-size: clamp(22px, 2.5vw, 28px) !important;
  font-weight: 700 !important;
  line-height: 1.2 !important;
}

.modal-panel--publish .modal-header span:not(.modal-title) {
  max-width: 520px;
  color: #667085 !important;
  font-size: clamp(13px, 1.6vw, 15px) !important;
  line-height: 1.5 !important;
}

.modal-panel--publish .modal-close--publish {
  flex: 0 0 40px;
  width: 40px;
  height: 40px;
  margin-left: 16px !important;
  color: #1e2a44 !important;
  background: #f7f8fa;
  border: 1px solid #d8dee8;
  border-radius: 12px;
  cursor: pointer;
  transition: background-color 180ms ease, border-color 180ms ease, transform 180ms ease;
}

.modal-panel--publish .modal-close--publish:hover {
  background: #eef1f5;
  border-color: #b8c1cf;
}

.modal-panel--publish .modal-close--publish:active {
  transform: scale(.96);
}

.modal-panel--publish .modal-close--publish:focus-visible {
  outline: 3px solid rgb(244 180 0 / 30%);
  outline-offset: 2px;
}

@media only screen and (min-width: 768px) and (max-width: 1023px) {
  .modal-panel--publish {
    position: static;
    width: 90%;
    max-width: 680px;
    margin: auto;
    padding: 24px !important;
    border-radius: 20px !important;
  }
}

@media only screen and (max-width: 767px) {
  .modal-panel--publish {
    width: 100%;
    max-height: 92vh;
    padding: 24px 16px max(20px, env(safe-area-inset-bottom)) !important;
    border-radius: 20px 20px 0 0 !important;
  }

  .modal-panel--publish .modal-header {
    gap: 16px;
  }

  .modal-panel--publish .modal-close--publish {
    flex-basis: 48px;
    width: 48px;
    height: 48px;
    margin-left: 8px !important;
  }
}

@media (prefers-reduced-motion: reduce) {
  .modal-panel--publish .modal-close--publish {
    transition: none;
  }
}
@media only screen and (max-width: 960px) {
  .mobile-style {
    position: absolute;
    bottom: 0;
    border-radius: 24px 24px 0 0 !important;
  }
}
</style>
