<template>
    <component :is="animate ? 'div' : 'div'" :class="classes">
      <Transition name="card" appear v-if="animate">
        <div>
          <slot />
        </div>
      </Transition>
      <template v-else>
        <slot />
      </template>
    </component>
  </template>
  
  <script setup lang="ts">
  import { computed } from "vue";
  
  const props = withDefaults(
    defineProps<{ class?: string; animate?: boolean }>(),
    { animate: true },
  );
  
  const classes = computed(() => [
    "bg-card rounded-xl shadow-card transition-shadow duration-300 hover:shadow-card-hover overflow-hidden",
    props.class,
  ]);
  </script>
  
  <style scoped>
  .card-enter-active {
    transition: opacity 400ms ease, transform 400ms ease;
  }
  .card-enter-from {
    opacity: 0;
    transform: translateY(20px);
  }
  .card-enter-to {
    opacity: 1;
    transform: translateY(0);
  }
  </style>
  