<script setup lang='ts'>
defineProps({
  icon: {
    type: String,
    required: true,
  },
  active: {
    type: Boolean,
    default: false,
  },
  none: {
    type: Boolean,
    default: false,
  },
  text: {
    type: String,
    default: '',
  },
})
</script>

<template>
  <div
    class="icon-button m-auto"
    :class="none ? '' : active ? 'opacity-65 hover:opacity-100' : 'opacity-65 hover:opacity-100'"
  >
    <Icon
      :icon="icon"
      class="inline-block align-middle"
    />
    <div
      v-if="text"
      class="text-xs ml-2 inline-block align-middle"
    >
      {{ text }}
    </div>
  </div>
</template>
