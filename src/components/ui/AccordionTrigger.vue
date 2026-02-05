<template>
    <div class="flex">
      <button
        type="button"
        :class="classes"
        @click="accordion.toggle(id)"
        :aria-expanded="accordion.isOpen(id)"
      >
        <span><slot /></span>
        <span
          class="h-4 w-4 shrink-0 transition-transform duration-200"
          :class="accordion.isOpen(id) ? 'rotate-180' : ''"
          aria-hidden="true"
        >
          <!-- simple chevron -->
          <svg viewBox="0 0 24 24" fill="none" class="w-4 h-4">
            <path d="M6 9l6 6 6-6" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </span>
      </button>
    </div>
  </template>
  
  <script setup lang="ts">
  import { computed, inject } from "vue";
  
  const props = defineProps<{ id: string; class?: string }>();
  
  const accordion = inject<any>("accordion");
  if (!accordion) throw new Error("AccordionTrigger must be used inside Accordion");
  
  const classes = computed(() => [
    "flex flex-1 items-center justify-between py-4 font-medium transition-all hover:underline",
    props.class,
  ]);
  </script>
