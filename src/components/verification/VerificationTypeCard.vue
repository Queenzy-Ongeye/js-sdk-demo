<template>
    <button
      type="button"
      class="relative w-full p-4 rounded-xl text-left transition-all duration-200 border-2 bg-card"
      :class="selected ? 'border-primary shadow-card-hover' : 'border-transparent shadow-card hover:border-primary/30'"
      @click="$emit('select', id)"
    >
      <div v-if="selected" class="absolute inset-0 bg-primary/5 rounded-xl" />
  
      <div class="relative flex items-start gap-4">
        <div
          class="flex-shrink-0 w-10 h-10 rounded-lg flex items-center justify-center transition-colors duration-200"
          :class="selected ? 'bg-primary text-primary-foreground' : 'bg-muted text-muted-foreground'"
        >
          <slot name="icon" />
        </div>
  
        <div class="flex-1 min-w-0">
          <h3 class="font-semibold text-foreground">{{ name }}</h3>
          <p class="text-sm text-muted-foreground mt-0.5 line-clamp-2">
            {{ description }}
          </p>
        </div>
  
        <div
          class="flex-shrink-0 w-5 h-5 rounded-full border-2 transition-all duration-200 flex items-center justify-center"
          :class="selected ? 'border-primary bg-primary' : 'border-muted-foreground/30'"
        >
          <div v-if="selected" class="w-2 h-2 rounded-full bg-primary-foreground scale-in" />
        </div>
      </div>
    </button>
  </template>
  
  <script setup lang="ts">
  defineProps<{
    id: string;
    name: string;
    description: string;
    selected: boolean;
  }>();
  
  defineEmits<{
    (e: "select", id: string): void;
  }>();
  </script>
  
  <style scoped>
  .scale-in {
    animation: scaleIn 160ms ease-out both;
  }
  @keyframes scaleIn {
    from { transform: scale(0); }
    to { transform: scale(1); }
  }
  </style>
  