<template>
  <button
    class="
      px-5
      py-2.5
      rounded-lg
      bg-lime-400
      disabled:bg-slate-500
    "
    :disabled="disabled"
    :alt="alt"
  >
    <slot></slot>
  </button>
</template>

<script setup>
  const { disabled, alt } = defineProps({
    disabled: Boolean,
    alt: String,
  })
</script>
