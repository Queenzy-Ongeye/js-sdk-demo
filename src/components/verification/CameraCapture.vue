<template>
    <div class="relative fade-in">
      <div class="smart-camera-container bg-muted/50 rounded-xl p-2">
        <smart-camera-web ref="cameraEl" theme-color="#0d9488" :capture-id="captureId" />
      </div>
    </div>
  </template>
  
  <script setup lang="ts">
    import { onMounted, ref, watch } from "vue";
    import "@smileid/web-components/smart-camera-web";
    
    const props = withDefaults(defineProps<{ captureId?: boolean }>(), { captureId: false });
    
    const cameraEl = ref<HTMLElement | null>(null);
    
    defineExpose({ cameraEl });
    
    function syncAttr() {
      const el = cameraEl.value;
      if (!el) return;
      if (props.captureId) el.setAttribute("capture-id", "");
      else el.removeAttribute("capture-id");
    }
    
    onMounted(syncAttr);
    watch(() => props.captureId, syncAttr);
    </script>
    
  
  <style scoped>
  .fade-in {
    animation: fadeIn 200ms ease-out;
  }
  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }
  
  .smart-camera-container {
    width: 100%;
    border-radius: 12px;
    overflow: hidden;
  }
  smart-camera-web {
    width: 100%;
    display: block;
    border-radius: 8px;
    overflow: hidden;
    min-height: 400px;
  }
  </style>
  