<template>
    <div>
      <slot />
    </div>
  </template>
  
  <script setup lang="ts">
  import { computed, provide } from "vue";
  
  type AccordionType = "single" | "multiple";
  
  const props = withDefaults(
    defineProps<{
      modelValue?: string | string[];
      type?: AccordionType;
    }>(),
    { type: "single" },
  );
  
  const emit = defineEmits<{
    (e: "update:modelValue", v: string | string[]): void;
  }>();
  
  const open = computed(() => props.modelValue);
  
  function isOpen(id: string) {
    if (props.type === "multiple") return Array.isArray(open.value) && open.value.includes(id);
    return typeof open.value === "string" && open.value === id;
  }
  
  function toggle(id: string) {
    if (props.type === "multiple") {
      const curr = Array.isArray(open.value) ? open.value : [];
      const next = curr.includes(id) ? curr.filter((x) => x !== id) : [...curr, id];
      emit("update:modelValue", next);
    } else {
      emit("update:modelValue", open.value === id ? "" : id);
    }
  }
  
  provide("accordion", { isOpen, toggle });
  </script>
  