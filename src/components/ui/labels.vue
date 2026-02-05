<template>
    <label :for="forId" :class="classes">
      <slot />
    </label>
  </template>
  
  <script setup lang="ts">
  import { computed } from "vue";
  
  const props = defineProps<{
    forId?: string;
    class?: string;
  }>();
  
  const classes = computed(() => [
    "text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70",
    props.class,
  ]);
  </script>
  