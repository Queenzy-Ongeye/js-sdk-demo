<template>
    <span class="badge" :class="config.className">
      <span class="icon" :class="{ spin: status === 'processing' }">
        <slot name="icon" />
      </span>
      {{ config.label }}
    </span>
  </template>
  
  <script setup lang="ts">
  import { computed } from "vue";
  
  export type Status = "pending" | "processing" | "completed" | "error";
  
  const props = defineProps<{ status: Status }>();
  
  const config = computed(() => {
    const map: Record<Status, { label: string; className: string }> = {
      pending: { label: "Pending", className: "bg-muted text-muted-foreground" },
      processing: { label: "Processing", className: "bg-primary/10 text-primary" },
      completed: { label: "Verified", className: "bg-success/10 text-success" },
      error: { label: "Failed", className: "bg-destructive/10 text-destructive" },
    };
    return map[props.status];
  });
  </script>
  
  <style scoped>
  .badge {
    display: inline-flex;
    align-items: center;
    gap: 0.375rem;
    padding: 0.375rem 0.75rem;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 600;
    animation: popIn 160ms ease-out both;
  }
  @keyframes popIn {
    from { transform: scale(0.9); opacity: 0; }
    to { transform: scale(1); opacity: 1; }
  }
  .icon {
    width: 0.875rem;
    height: 0.875rem;
    display: inline-flex;
  }
  .spin {
    animation: spin 900ms linear infinite;
  }
  @keyframes spin {
    to { transform: rotate(360deg); }
  }
  </style>
  