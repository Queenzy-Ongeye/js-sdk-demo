<template>
  <div class="space-y-6 fade-up">
    <!-- Header -->
    <div class="flex items-center justify-between">
      <h3 class="text-lg font-semibold text-foreground">
        {{ status === "completed" ? "Verification Complete" : "Processing Verification" }}
      </h3>

      <StatusBadge :status="status">
        <template #icon>
          <!-- optional icon slot -->
        </template>
      </StatusBadge>
    </div>

    <!-- Error -->
    <Transition name="expand">
      <div
        v-if="error"
        class="flex items-start gap-3 p-4 bg-destructive/10 border border-destructive/20 rounded-lg"
      >
        <div class="w-5 h-5 text-destructive flex-shrink-0 mt-0.5">
          <!-- optional error icon -->
        </div>
        <div class="flex-1">
          <p class="text-sm text-destructive font-medium">{{ error }}</p>
          <button
            type="button"
            @click="onReset"
            class="text-xs font-semibold text-destructive/80 underline mt-2 hover:text-destructive"
          >
            Try Again
          </button>
        </div>
      </div>
    </Transition>

    <!-- Details -->
    <div class="space-y-3">
      <DetailRow
        label="Job ID"
        :value="jobId || '...'"
        :copyable="!!jobId"
        :copied="copiedField === 'jobId'"
        @copy="jobId && copyToClipboard(jobId, 'jobId')"
      />
      <DetailRow
        label="User ID"
        :value="userId || '...'"
        :copyable="!!userId"
        :copied="copiedField === 'userId'"
        mono
        @copy="userId && copyToClipboard(userId, 'userId')"
      />
    </div>

    <!-- Saved images -->
    <Transition name="fade">
      <div v-if="savedImages.length > 0" class="space-y-3">
        <p class="text-sm font-medium text-foreground">Saved Images</p>
        <ul class="space-y-2">
          <li
            v-for="(img, index) in savedImages"
            :key="img.url || index"
            class="flex items-center gap-2 text-sm bg-muted/50 rounded-lg px-3 py-2"
          >
            <code class="text-xs text-muted-foreground font-mono">
              {{ img.image_type_id }}
            </code>
            <span class="text-muted-foreground">—</span>
            <a
              :href="img.url"
              target="_blank"
              rel="noreferrer"
              class="text-primary hover:underline flex items-center gap-1 truncate flex-1"
            >
              <span class="truncate">{{ img.url }}</span>
              <span class="w-3 h-3 flex-shrink-0" aria-hidden="true">
                <!-- optional external-link icon -->
              </span>
            </a>
          </li>
        </ul>
      </div>
    </Transition>

    <!-- Submit response -->
    <Transition name="fade">
      <div v-if="submitResponse !== undefined && submitResponse !== null" class="space-y-3">
        <p class="text-sm font-medium text-foreground">Submit Response</p>
        <pre class="bg-muted/50 rounded-lg p-4 text-xs overflow-auto max-h-48 whitespace-pre-wrap font-mono">{{ submitText }}</pre>
      </div>
    </Transition>

    <!-- Job status -->
    <Transition name="fade">
      <div v-if="jobStatus" class="space-y-3">
        <p class="text-sm font-medium text-foreground">Job Status</p>
        <pre class="bg-muted/50 rounded-lg p-4 text-xs overflow-auto max-h-48 whitespace-pre-wrap font-mono">{{ JSON.stringify(jobStatus, null, 2) }}</pre>
      </div>
    </Transition>

    <!-- Reset -->
    <div v-if="status === 'completed' || status === 'error'" class="fade-in">
      <Button variant="outline" class="w-full gap-2" @click="onReset">
        Start New Verification
      </Button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, defineComponent, ref } from "vue";
import StatusBadge, { type Status } from "./StatusBadge.vue";
import Button from "../ui/Button.vue";

type SavedImage = { image_type_id: string; url: string };
type JobStatus = { status?: string; [key: string]: unknown };

const props = withDefaults(
  defineProps<{
    status: Status;
    jobId?: string | null;
    userId?: string | null;
    error?: string | null;
    savedImages?: SavedImage[];
    submitResponse?: unknown;
    jobStatus?: JobStatus | null;
    onReset: () => void;
  }>(),
  { savedImages: () => [] },
);

const copiedField = ref<string | null>(null);

function copyToClipboard(text: string, field: string) {
  navigator.clipboard.writeText(text);
  copiedField.value = field;
  window.setTimeout(() => (copiedField.value = null), 2000);
}

const submitText = computed(() => {
  const v = props.submitResponse;
  if (typeof v === "string") return v;
  return JSON.stringify(v, null, 2);
});

/** Local component */
const DetailRow = defineComponent({
  name: "DetailRow",
  props: {
    label: { type: String, required: true },
    value: { type: String, required: true },
    copyable: { type: Boolean, default: false },
    copied: { type: Boolean, default: false },
    mono: { type: Boolean, default: false },
  },
  emits: ["copy"],
  template: `
    <div class="flex items-center justify-between py-3 border-b border-border last:border-0">
      <span class="text-sm text-muted-foreground">{{ label }}</span>
      <div class="flex items-center gap-2">
        <span :class="['text-sm font-medium text-foreground', mono ? 'font-mono text-xs' : '']">
          {{ value }}
        </span>
        <button
          v-if="copyable"
          type="button"
          @click="$emit('copy')"
          class="p-1.5 rounded-md hover:bg-muted transition-colors"
        >
          <span v-if="copied" class="text-success text-xs font-semibold">Copied</span>
          <span v-else class="text-muted-foreground text-xs font-semibold">Copy</span>
        </button>
      </div>
    </div>
  `,
});
</script>

<style scoped>
.fade-up {
  animation: fadeUp 220ms ease-out both;
}
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
.fade-in {
  animation: fadeIn 200ms ease-out both;
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.fade-enter-active, .fade-leave-active { transition: opacity 180ms ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.expand-enter-active, .expand-leave-active { transition: all 200ms ease; }
.expand-enter-from, .expand-leave-to { opacity: 0; max-height: 0; }
.expand-enter-to, .expand-leave-from { opacity: 1; max-height: 200px; }
</style>
