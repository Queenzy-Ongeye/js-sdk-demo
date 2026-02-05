<template>
    <Transition name="accordion">
      <div v-if="accordion.isOpen(id)" class="overflow-hidden text-sm">
        <div :class="['pb-4', 'pt-0', $props.class]">
          <slot />
        </div>
      </div>
    </Transition>
  </template>
  
  <script setup lang="ts">
  import { inject } from "vue";
  
  defineProps<{ id: string; class?: string }>();
  
  const accordion = inject<any>("accordion");
  if (!accordion) throw new Error("AccordionContent must be used inside Accordion");
  </script>
  
  <style scoped>
  .accordion-enter-active,
  .accordion-leave-active {
    transition: max-height 200ms ease, opacity 200ms ease;
  }
  .accordion-enter-from,
  .accordion-leave-to {
    max-height: 0;
    opacity: 0;
  }
  .accordion-enter-to,
  .accordion-leave-from {
    max-height: 500px;
    opacity: 1;
  }
  </style>
  