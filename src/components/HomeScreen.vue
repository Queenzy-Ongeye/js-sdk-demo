<template>
  <div class="min-h-screen bg-background py-8 px-4 sm:px-6 lg:px-8">
    <div class="max-w-xl mx-auto">
      <div class="text-center mb-8 header-enter">
        <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-primary mb-4">
          <Shield class="w-8 h-8 text-primary-foreground" />
        </div>

        <h1 class="text-3xl font-bold text-foreground tracking-tight">
          Identity Verification
        </h1>

        <p class="mt-2 text-muted-foreground flex items-center justify-center gap-1.5">
          <Sparkles class="w-4 h-4" />
          Powered by SmileID
        </p>

        <div class="flex items-center justify-center gap-2 mt-6">
          <div
            v-for="s in [1, 2, 3]"
            :key="s"
            class="h-1.5 rounded-full transition-all duration-300"
            :class="progressClass(s)"
          />
        </div>
      </div>

      <Transition name="step" mode="out-in">
        <VerificationCard v-if="step === 1" key="step1">
          <div class="p-6">
            <h2 class="text-lg font-semibold text-foreground mb-1">
              Select Verification Type
            </h2>
            <p class="text-sm text-muted-foreground mb-6">
              Choose the type of verification that suits your needs
            </p>

            <div class="space-y-3">
              <VerificationTypeCard
                v-for="type in verificationTypes"
                :key="type.id"
                :id="type.id"
                :name="type.name"
                :description="type.description"
                :selected="kycType === type.id"
                @select="handleTypeChange"
              >
                <template #icon>
                  <component :is="type.icon" class="w-5 h-5" />
                </template>
              </VerificationTypeCard>
            </div>

            <Button class="w-full mt-6 gap-2" @click="step = 2">
              Continue
              <ChevronRight class="w-4 h-4" />
            </Button>
          </div>
        </VerificationCard>

        <VerificationCard v-else-if="step === 2" key="step2">
          <div class="p-6">
            <h2 class="text-lg font-semibold text-foreground mb-1">
              {{ selectedType?.name }} Details
            </h2>
            <p class="text-sm text-muted-foreground mb-6">
              Enter the required information below
            </p>

            <div
              v-if="!isAml && kycType !== 'document'"
              class="grid grid-cols-1 sm:grid-cols-2 gap-4"
            >
              <FormField v-model="userInputs.country" label="Country" placeholder="e.g. KE" :delay="0" />
              <FormField v-model="userInputs.id_type" label="ID Type" placeholder="e.g. NATIONAL_ID" :delay="0.05" />
              <FormField v-model="userInputs.id_number" label="ID Number" placeholder="Your ID number" :delay="0.1" />
              <FormField v-model="userInputs.first_name" label="First Name" placeholder="John" :delay="0.15" />
              <FormField v-model="userInputs.last_name" label="Last Name" placeholder="Doe" :delay="0.2" />
              <FormField v-model="userInputs.dob" label="Date of Birth" placeholder="YYYY-MM-DD" :delay="0.25" />
            </div>

            <div
              v-else-if="kycType === 'document'"
              class="grid grid-cols-1 sm:grid-cols-2 gap-4"
            >
              <FormField v-model="docInputs.country" label="Country" placeholder="e.g. KE" :delay="0" />
              <FormField v-model="docInputs.id_type" label="ID Type" placeholder="e.g. NATIONAL_ID" :delay="0.05" />
            </div>

            <div v-else class="space-y-4">
              <FormField
                v-model="amlCountriesText"
                label="Countries (comma-separated)"
                placeholder="e.g. KE, UG"
                :delay="0"
              />
              <FormField v-model="amlInputs.full_name" label="Full Name" placeholder="John Doe" :delay="0.05" />
              <FormField v-model="amlInputs.birth_year" label="Birth Year" placeholder="e.g. 1998" :delay="0.1" />
            </div>
          </div>

          <div v-if="!isAml" class="border-t border-border bg-muted/30 p-6">
            <div class="relative fade-in">
              <div class="smart-camera-container bg-muted/50 rounded-xl p-2">
                <smart-camera-web
                  v-if="needsDocumentCapture"
                  ref="cameraEl"
                  theme-color="#0d9488"
                  capture-id
                ></smart-camera-web>

                <smart-camera-web
                  v-else
                  ref="cameraEl"
                  theme-color="#0d9488"
                ></smart-camera-web>
              </div>
            </div>

            <p class="text-center text-xs text-muted-foreground mt-4 italic">
              Camera will automatically submit upon capture approval
            </p>
          </div>

          <div class="p-6 pt-0 flex gap-3">
            <Button variant="outline" class="flex-1" @click="step = 1">
              Back
            </Button>

            <Button v-if="isAml" class="flex-1 gap-2" :disabled="loading" @click="submitAml">
              <span v-if="loading">Processing...</span>
              <span v-else>Run AML Check</span>
              <ChevronRight v-if="!loading" class="w-4 h-4" />
            </Button>
          </div>
        </VerificationCard>

        <VerificationCard v-else key="step3">
          <div class="p-6">
            <ProcessingResults
              :status="statusType"
              :jobId="jobId"
              :userId="userId"
              :error="error"
              :savedImages="displayedSavedImages"
              :submitResponse="submitResponse"
              :jobStatus="status"
              :onReset="handleReset"
            />
          </div>
        </VerificationCard>
      </Transition>

      <p class="text-center text-xs text-muted-foreground mt-8 footer-enter">
        Your data is encrypted and securely processed
      </p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from "vue";
import "@smileid/web-components/smart-camera-web";
import {
  Shield,
  UserCheck,
  Search,
  FileText,
  Fingerprint,
  ChevronRight,
  Sparkles,
} from "lucide-vue-next";

import VerificationCard from "../components/verification/VerificationCard.vue";
import VerificationTypeCard from "../components/verification/VerificationTypeCard.vue";
import FormField from "../components/verification/FormField.vue";
import ProcessingResults from "../components/verification/ProcessingResults.vue";
import Button from "../components/ui/Button.vue";

type KycType = "basic" | "enhanced" | "aml" | "document" | "biometric";

interface SavedImage {
  image_type_id: string;
  url: string;
}
interface SubmitResponse {
  job_id?: string;
  user_id?: string;
  saved_images?: SavedImage[];
  submit_response?: unknown;
  error?: string;
}
interface JobStatus {
  status?: string;
  [key: string]: unknown;
}

const verificationTypes = [
  { id: "basic", name: "Basic KYC", description: "Standard identity verification with name and ID number", icon: UserCheck },
  { id: "enhanced", name: "Enhanced KYC", description: "Comprehensive verification with additional data points", icon: Shield },
  { id: "aml", name: "AML Check", description: "Anti-money laundering screening against global watchlists", icon: Search },
  { id: "document", name: "Document Verification", description: "Verify identity documents like passports and IDs", icon: FileText },
  { id: "biometric", name: "Biometric KYC", description: "Face recognition and liveness detection", icon: Fingerprint },
] as const;

const kycType = ref<KycType>("basic");
const step = ref<1 | 2 | 3>(1);

const loading = ref(false);
const status = ref<JobStatus | null>(null);
const error = ref<string | null>(null);

const jobId = ref<string | null>(null);
const userId = ref<string | null>(null);
const submitResponse = ref<unknown>(null);
const savedImages = ref<SavedImage[]>([]);

let pollTimer: number | null = null;

const userInputs = ref({
  country: "",
  id_type: "",
  id_number: "",
  first_name: "",
  last_name: "",
  dob: "",
});

const amlInputs = ref({
  countries: [] as string[],
  full_name: "",
  birth_year: "",
});

const amlCountriesText = computed({
  get: () => amlInputs.value.countries.join(", "),
  set: (v: string) => {
    amlInputs.value.countries = v
      .split(",")
      .map((s) => s.trim())
      .filter(Boolean);
  },
});

const docInputs = ref({
  country: "",
  id_type: "",
});

const isAml = computed(() => kycType.value === "aml");
const needsDocumentCapture = computed(() => kycType.value === "document");
const selectedType = computed(() => verificationTypes.find((t) => t.id === kycType.value));

const shouldShowSavedImages = computed(() => {
  return kycType.value === "document" || kycType.value === "biometric";
});

const displayedSavedImages = computed(() => {
  return shouldShowSavedImages.value ? savedImages.value : [];
});

function resetUi() {
  error.value = null;
  jobId.value = null;
  userId.value = null;
  submitResponse.value = null;
  savedImages.value = [];
  status.value = null;
}

function stopPolling() {
  if (pollTimer) {
    clearInterval(pollTimer);
    pollTimer = null;
  }
}

function startPolling(id: string) {
  stopPolling();
  pollTimer = window.setInterval(async () => {
    try {
      const resp = await fetch(`/api/smile/status/${id}`);
      const text = await resp.text();
      const statusData = text ? (JSON.parse(text) as JobStatus) : null;
      status.value = statusData;

      if (statusData?.status === "completed" || statusData?.status === "failed") {
        stopPolling();
      }
    } catch {
      // ignore transient polling errors
    }
  }, 2000);
}

async function readJson(resp: Response): Promise<SubmitResponse | null> {
  const text = await resp.text();
  try {
    return text ? (JSON.parse(text) as SubmitResponse) : null;
  } catch {
    throw new Error(`Non-JSON response (HTTP ${resp.status}): ${text.slice(0, 200)}`);
  }
}

const cameraEl = ref<HTMLElement | null>(null);

watch(cameraEl, (newEl, oldEl) => {
  if (oldEl) oldEl.removeEventListener("smart-camera-web.publish", handlePublish as any);
  if (newEl) newEl.addEventListener("smart-camera-web.publish", handlePublish as any);
});

async function handlePublish(event: Event) {
  if (kycType.value === "aml") {
    error.value = "AML does not use image capture. Use the AML submit button.";
    return;
  }

  const capturePayload = (event as CustomEvent).detail;

  loading.value = true;
  resetUi();
  step.value = 3;

  try {
    const payloadUserInputs = kycType.value === "document" ? docInputs.value : userInputs.value;

    const resp = await fetch("/api/smile/submit", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        product: kycType.value,
        userInputs: payloadUserInputs,
        capturePayload,
      }),
    });

    const json = await readJson(resp);

    if (!resp.ok) {
      error.value = json?.error || `Submit failed (HTTP ${resp.status})`;
      submitResponse.value = json;
      return;
    }

    jobId.value = json?.job_id || null;
    userId.value = json?.user_id || null;
    savedImages.value = json?.saved_images || [];
    submitResponse.value = json?.submit_response ?? null;

    if (json?.job_id) startPolling(json.job_id);
  } catch (err) {
    console.error(err);
    error.value = err instanceof Error ? err.message : "Network/Server error";
  } finally {
    loading.value = false;
  }
}

async function submitAml() {
  loading.value = true;
  resetUi();
  step.value = 3;

  try {
    const resp = await fetch("/api/smile/submit", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        product: "aml",
        userInputs: amlInputs.value,
      }),
    });

    const json = await readJson(resp);

    if (!resp.ok) {
      error.value = json?.error || `AML submit failed (HTTP ${resp.status})`;
      submitResponse.value = json;
      return;
    }

    jobId.value = json?.job_id || null;
    userId.value = json?.user_id || null;
    submitResponse.value = json?.submit_response ?? null;

    if (json?.job_id) startPolling(json.job_id);
  } catch (err) {
    console.error(err);
    error.value = err instanceof Error ? err.message : "Network/Server error";
  } finally {
    loading.value = false;
  }
}

function handleReset() {
  stopPolling();
  step.value = 1;
  kycType.value = "basic";
  loading.value = false;
  resetUi();

  userInputs.value = { country: "", id_type: "", id_number: "", first_name: "", last_name: "", dob: "" };
  amlInputs.value = { countries: [], full_name: "", birth_year: "" };
  docInputs.value = { country: "", id_type: "" };
}

function handleTypeChange(id: string) {
  kycType.value = id as KycType;
  resetUi();
}

const statusType = computed<"pending" | "processing" | "completed" | "error">(() => {
  if (!status.value?.status) return loading.value ? "processing" : "pending";
  if (status.value.status === "completed") return "completed";
  if (status.value.status === "failed") return "error";
  return "processing";
});

function progressClass(s: number) {
  if (s === step.value) return "w-8 bg-primary";
  if (s < step.value) return "w-4 bg-primary/50";
  return "w-4 bg-muted";
}

onUnmounted(() => {
  stopPolling();
  if (cameraEl.value) {
    cameraEl.value.removeEventListener("smart-camera-web.publish", handlePublish as any);
  }
});
</script>

<style scoped>
.step-enter-active,
.step-leave-active {
  transition: opacity 220ms ease, transform 220ms ease;
}

.step-enter-from,
.step-leave-to {
  opacity: 0;
  transform: translateY(12px);
}

.step-enter-to,
.step-leave-from {
  opacity: 1;
  transform: translateY(0);
}

.header-enter {
  animation: headerIn 260ms ease-out both;
}

@keyframes headerIn {
  from {
    opacity: 0;
    transform: translateY(-12px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.footer-enter {
  animation: footerIn 260ms ease-out both;
}

@keyframes footerIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.fade-in {
  animation: fadeIn 200ms ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

smart-camera-web {
  width: 100%;
  display: block;
  border-radius: 8px;
  overflow: hidden;
  min-height: 400px;
}
</style>