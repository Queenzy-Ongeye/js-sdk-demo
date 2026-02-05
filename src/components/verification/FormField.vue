<template>
    <div class="space-y-2 slide-up" :style="{ animationDelay: `${delay}s` }">
      <Label class="text-xs font-medium uppercase tracking-wider text-muted-foreground">
        {{ label }}
      </Label>
  
      <Input
        :type="type"
        v-model="localValue"
        :placeholder="placeholder"
        class="bg-background border-input focus:border-primary focus:ring-2 focus:ring-primary/20 transition-all duration-200"
      />
    </div>
  </template>
  
  <script setup lang="ts">
  import { computed } from "vue";
  import Label from "../ui/labels.vue";
  import Input from "../ui/input.vue";
  
  const props = withDefaults(
    defineProps<{
      label: string;
      modelValue: string;
      placeholder?: string;
      type?: string;
      delay?: number;
    }>(),
    { type: "text", delay: 0 },
  );
  
  const emit = defineEmits<{
    (e: "update:modelValue", v: string): void;
  }>();
  
  const localValue = computed({
    get: () => props.modelValue,
    set: (v: string) => emit("update:modelValue", v),
  });
  </script>
  
  <style scoped>
  .slide-up {
    animation: slideUp 300ms ease-out both;
  }
  @keyframes slideUp {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }
  </style>
  