<script setup>
  import { ref, computed, watch, onBeforeUnmount } from "vue";
  import "@smileid/web-components/smart-camera-web";
  
  const kycType = ref("basic");
  const loading = ref(false);
  const error = ref(null);
  
  const jobId = ref(null);
  const userId = ref(null);
  const submitResponse = ref(null);
  const savedImages = ref([]);
  const status = ref(null);
  
  const cameraEl = ref(null);
  let pollTimer = null;
  
  const needsDocumentCapture = computed(() => kycType.value === "document");
  const isAml = computed(() => kycType.value === "aml");
  
  // Inputs for Basic/Enhanced
  const userInputs = ref({
    // You can keep these minimal, but Smile often rejects empties.
    country: "",
    id_type: "",
    id_number: "",
    first_name: "",
    last_name: "",
    dob: "",
  });
  
  // Inputs for AML (/v1/aml) — NO camera
  const amlInputs = ref({
    countries: [],
    full_name: "",
    birth_year: "",
  });
  
  // Inputs for Document Verification (/v1/upload)
  // Must include country + id_type at minimum for the document
  const docInputs = ref({
    country: "",
    id_type: "",
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
    if (pollTimer) clearInterval(pollTimer);
    pollTimer = null;
  }
  
  function startPolling(id) {
    stopPolling();
    pollTimer = setInterval(async () => {
      try {
        const resp = await fetch(`/api/smile/status/${id}`);
        const text = await resp.text();
        status.value = text ? JSON.parse(text) : null;
        if (status.value?.status === "completed" || status.value?.status === "failed") stopPolling();
      } catch {
        // ignore transient polling errors
      }
    }, 2000);
  }
  
  async function readJson(resp) {
    const text = await resp.text();
    try {
      return text ? JSON.parse(text) : null;
    } catch {
      throw new Error(`Non-JSON response (HTTP ${resp.status}): ${text.slice(0, 200)}`);
    }
  }
  
  // Fired by smart-camera-web when user approves images
  async function handlePublish(event) {
    if (kycType.value === "aml") {
      error.value = "AML does not use image capture. Use the AML submit button.";
      return;
    }
  
    const capturePayload = event.detail;
  
    loading.value = true;
    resetUi();
  
    try {
      const payloadUserInputs =
        kycType.value === "document" ? docInputs.value : userInputs.value;
  
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
  
      jobId.value = json.job_id;
      userId.value = json.user_id;
      savedImages.value = json.saved_images || [];
      submitResponse.value = json.submit_response;
  
      startPolling(jobId.value);
    } catch (err) {
      console.error(err);
      error.value = err?.message || "Network/Server error";
    } finally {
      loading.value = false;
    }
  }
  
  // AML submit button
  async function submitAml() {
    loading.value = true;
    resetUi();
  
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
  
      jobId.value = json.job_id;
      userId.value = json.user_id;
      submitResponse.value = json.submit_response;
  
      startPolling(jobId.value);
    } catch (err) {
      console.error(err);
      error.value = err?.message || "Network/Server error";
    } finally {
      loading.value = false;
    }
  }
  
  // Rebind event when component is recreated by v-if/v-else
  watch(cameraEl, (newEl, oldEl) => {
    if (oldEl) oldEl.removeEventListener("smart-camera-web.publish", handlePublish);
    if (newEl) newEl.addEventListener("smart-camera-web.publish", handlePublish);
  });
  
  onBeforeUnmount(() => {
    stopPolling();
    if (cameraEl.value) {
      cameraEl.value.removeEventListener("smart-camera-web.publish", handlePublish);
    }
  });
  </script>
  
  <template>
    <div style="max-width: 620px; margin: 24px auto; font-family: system-ui;">
      <h2>SmileID Capture (Vue)</h2>
  
      <label>
        Product:
        <select v-model="kycType">
          <option value="basic">Basic</option>
          <option value="enhanced">Enhanced</option>
          <option value="aml">AML</option>
          <option value="document">Document Verification</option>
          <option value="biometric">Biometric KYC</option>
        </select>
      </label>
  
      <hr style="margin: 16px 0;" />
  
      <!-- BASIC / ENHANCED -->
      <div v-if="!isAml && kycType !== 'document'">
        <h3>User Inputs</h3>
        <div style="display: grid; gap: 8px;">
          <input v-model="userInputs.country" placeholder="country e.g. KE" />
          <input v-model="userInputs.id_type" placeholder="id_type e.g. NATIONAL_ID" />
          <input v-model="userInputs.id_number" placeholder="id_number" />
          <input v-model="userInputs.first_name" placeholder="first_name" />
          <input v-model="userInputs.last_name" placeholder="last_name" />
          <input v-model="userInputs.dob" placeholder="dob YYYY-MM-DD" />
        </div>
  
        <hr style="margin: 16px 0;" />
  
        <smart-camera-web ref="cameraEl" theme-color="#007bff"></smart-camera-web>
        <p style="margin-top: 10px;">Capture auto-submits when you approve images.</p>
      </div>
  
      <!-- DOCUMENT VERIFICATION -->
      <div v-else-if="kycType === 'document'">
        <h3>Document Verification Inputs</h3>
        <div style="display: grid; gap: 8px;">
          <input v-model="docInputs.country" placeholder="country of issuance e.g. KE" />
          <input v-model="docInputs.id_type" placeholder="document type e.g. NATIONAL_ID" />
        </div>
  
        <hr style="margin: 16px 0;" />
  
        <!-- capture-id enables document capture flow -->
        <smart-camera-web
          ref="cameraEl"
          theme-color="#007bff"
          capture-id
        ></smart-camera-web>
  
        <p style="margin-top: 10px;">
          Capture should include selfie + document front (and optionally back), then auto-submit.
        </p>
      </div>
  
      <!-- AML -->
      <div v-else>
        <h3>AML Inputs (no camera)</h3>
        <div style="display: grid; gap: 8px;">
          <input
            :value="amlInputs.countries.join(',')"
            @input="amlInputs.countries = $event.target.value.split(',').map(s => s.trim()).filter(Boolean)"
            placeholder="countries e.g. KE,UG"
          />
          <input v-model="amlInputs.full_name" placeholder="full_name" />
          <input v-model="amlInputs.birth_year" placeholder="birth_year e.g. 1998" />
        </div>
  
        <button
          @click="submitAml"
          :disabled="loading"
          style="margin-top: 12px; padding: 10px 12px; cursor: pointer;"
        >
          Submit AML
        </button>
      </div>
  
      <p v-if="loading" style="margin-top: 12px;">Submitting…</p>
      <p v-if="error" style="color: red; margin-top: 12px;">{{ error }}</p>
  
      <div v-if="jobId" style="margin-top: 12px;">
        <h3>Job ID</h3>
        <code>{{ jobId }}</code>
      </div>
  
      <div v-if="userId" style="margin-top: 12px;">
        <h3>User ID</h3>
        <code>{{ userId }}</code>
      </div>
  
      <div v-if="savedImages.length" style="margin-top: 12px;">
        <h3>Saved Images</h3>
        <ul>
          <li v-for="img in savedImages" :key="img.url">
            <code>{{ img.image_type_id }}</code> —
            <a :href="img.url" target="_blank" rel="noreferrer">{{ img.url }}</a>
          </li>
        </ul>
      </div>
  
      <div v-if="submitResponse" style="margin-top: 12px;">
        <h3>Submit Response</h3>
        <pre style="white-space: pre-wrap;">{{ JSON.stringify(submitResponse, null, 2) }}</pre>
      </div>
  
      <div v-if="status" style="margin-top: 12px;">
        <h3>Local Job Status</h3>
        <pre style="white-space: pre-wrap;">{{ JSON.stringify(status, null, 2) }}</pre>
      </div>
    </div>
  </template>
  