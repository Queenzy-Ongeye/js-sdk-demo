<template>
    <input
      :type="type"
      :class="classes"
      v-bind="$attrs"
      :value="modelValue"
      @input="onInput"
    />
  </template>
  
  <script setup lang="ts">
  import { computed } from "vue";
  
  defineOptions({ inheritAttrs: false });
  
  const props = defineProps<{
    modelValue?: string | number;
    type?: string;
    class?: string;
  }>();
  
  const emit = defineEmits<{
    (e: "update:modelValue", v: string): void;
  }>();
  
  const classes = computed(() => [
    "flex h-10 w-full rounded-md border border-input bg-background px-3 py-2 text-base ring-offset-background",
    "file:border-0 file:bg-transparent file:text-sm file:font-medium file:text-foreground",
    "placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2",
    "disabled:cursor-not-allowed disabled:opacity-50 md:text-sm",
    props.class,
  ]);
  
  function onInput(e: Event) {
    emit("update:modelValue", (e.target as HTMLInputElement).value);
  }
  </script>
  