<template>
  <img src="./assets/paynow-logo-2-01.png" ref="logoRef" class="logo-source" alt="" />

  <div class="page-shell" :class="{ 'image-mode': isImageMode }">
    <section v-if="isImageMode" class="image-mode-shell">
      <img
        v-if="qrcodeDataURL"
        :src="qrcodeDataURL"
        class="api-image"
        alt="Generated PayNow QR code"
      />
      <p v-else class="image-mode-error">{{ validationMessage || "Unable to generate QR code." }}</p>
    </section>

    <template v-else>
    <section class="hero no-print">
      <p class="eyebrow">Singapore PayNow</p>
      <h1>Generate a print-ready PayNow QR in seconds.</h1>
      <p class="hero-copy">
        Create a clean static QR for garage sales, hawker stalls, pop-ups, and counters. Add
        an optional amount, keep a reference number, then print or download the final PNG.
      </p>
    </section>

    <main class="workspace">
      <section class="panel form-panel no-print">
        <div class="panel-header">
          <p class="panel-kicker">Setup</p>
          <h2>Payment details</h2>
          <p>Switch between a mobile number or UEN and the preview updates automatically.</p>
        </div>

        <div class="mode-picker" role="radiogroup" aria-label="PayNow destination type">
          <button
            type="button"
            class="mode-pill"
            :class="{ active: mode === 'phone' }"
            @click="mode = 'phone'"
          >
            Phone number
          </button>
          <button
            type="button"
            class="mode-pill"
            :class="{ active: mode === 'uen' }"
            @click="mode = 'uen'"
          >
            UEN
          </button>
        </div>

        <div class="field-group">
          <label class="field-label" for="target">
            {{ mode === "phone" ? "Singapore mobile number" : "UEN" }}
          </label>
          <input
            id="target"
            :type="mode === 'phone' ? 'tel' : 'text'"
            :inputmode="mode === 'phone' ? 'numeric' : 'text'"
            :placeholder="mode === 'phone' ? '91234567 or +6591234567' : '201912345A'"
            v-model.trim="target"
            :aria-invalid="targetError ? 'true' : 'false'"
          />
          <p class="field-hint">
            {{
              mode === "phone"
                ? "Local 8-digit numbers are automatically saved as +65 format."
                : "Enter the company UEN exactly as registered."
            }}
          </p>
          <p v-if="targetError" class="field-error">{{ targetError }}</p>
        </div>

        <div class="field-group">
          <label class="field-label" for="reference">Reference number</label>
          <input
            id="reference"
            type="text"
            maxlength="25"
            placeholder="Optional bill or stall reference"
            v-model.trim="reference"
            :aria-invalid="referenceError ? 'true' : 'false'"
          />
          <p class="field-hint">
            Optional, but useful when you want a short bill, booth, or order reference.
          </p>
          <p v-if="referenceError" class="field-error">{{ referenceError }}</p>
        </div>

        <div class="field-group">
          <label class="field-label" for="amount">Amount (SGD)</label>
          <input
            id="amount"
            type="text"
            inputmode="decimal"
            placeholder="Optional fixed amount, e.g. 12.50"
            v-model.trim="amount"
            :aria-invalid="amountError ? 'true' : 'false'"
          />
          <p class="field-hint">
            Leave blank if the payer should key in the amount themselves.
          </p>
          <p v-if="amountError" class="field-error">{{ amountError }}</p>
        </div>

        <div class="summary-strip">
          <div>
            <span class="summary-label">Destination</span>
            <strong>{{ mode === "phone" ? "Phone" : "UEN" }}</strong>
          </div>
          <div>
            <span class="summary-label">Amount</span>
            <strong>{{ normalizedAmount ? `S$${normalizedAmount}` : "Editable" }}</strong>
          </div>
        </div>

        <div class="field-group api-link-group">
          <label class="field-label" for="api-url">Image URL</label>
          <input id="api-url" type="text" readonly :value="apiImageUrl" />
          <p class="field-hint">
            Use this static GitHub Pages-friendly URL to open the QR in dedicated image mode.
          </p>
        </div>
      </section>

      <section class="panel preview-panel" :class="{ invalid: Boolean(validationMessage) }">
        <div class="preview-card">
          <div class="preview-badge">
            {{ normalizedAmount ? "Fixed amount QR" : "Open amount QR" }}
          </div>
          <h2>Scan to pay</h2>
          <p class="preview-copy">
            {{
              validationMessage
                ? validationMessage
                : "The QR is ready for display, download, or printing."
            }}
          </p>

          <div class="qr-frame" :class="{ disabled: !qrcodeDataURL }">
            <img
              v-if="qrcodeDataURL"
              :src="qrcodeDataURL"
              class="generated-qr"
              alt="Generated PayNow QR code"
            />
            <div v-else class="empty-state">
              Enter a valid {{ mode === "phone" ? "phone number" : "UEN" }} to generate a QR
              code.
            </div>
          </div>

          <div class="caption-block">
            <p class="caption-label">PayNow to</p>
            <p class="caption-value">{{ displayTarget }}</p>
            <p v-if="reference" class="caption-meta">Ref: {{ reference }}</p>
            <p v-if="normalizedAmount" class="caption-meta">Amount: S${{ normalizedAmount }}</p>
          </div>

          <div class="action-row no-print">
            <button type="button" class="primary-action" @click="print" :disabled="!qrcodeDataURL">
              Print QR
            </button>
            <a
              class="secondary-action"
              :class="{ disabled: !qrcodeDataURL }"
              :href="qrcodeDataURL || undefined"
              :download="downloadFilename"
              :aria-disabled="!qrcodeDataURL"
            >
              Download PNG
            </a>
          </div>
        </div>
      </section>
    </main>

    <footer class="reference-panel no-print">
      <h2>Notes</h2>
      <p>
        Amount support is encoded using the EMV transaction amount field. When an amount is
        provided, the QR is generated as a fixed-amount payment; otherwise the payer can edit the
        amount in their banking app.
      </p>
      <p>
        You can also pass values by URL query string using
        <code>mode</code>, <code>target</code>, <code>reference</code>, and
        <code>amount</code>. Use <code>image.html</code> for the dedicated static image page, or
        add <code>view=image</code> to the main page for compatibility.
      </p>
    </footer>
    </template>

    <canvas ref="canvasRef" :width="imageWidth" :height="imageWidth" class="hidden-canvas"></canvas>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, watch } from "vue";
import { sortBy, throttle } from "lodash";
import * as qrcode from "qrcode";
import crc16ccitt from "crc/calculators/crc16ccitt";

type PayNowMode = "phone" | "uen";
type QRDataComponentValue = null | string | QRData;

class QRData {
  components: Record<string, QRDataComponentValue>;

  constructor(components: Record<string, QRDataComponentValue>) {
    this.components = components;
  }

  toString(): string {
    return sortBy(Object.entries(this.components), ([key]) => key)
      .map(([key, component]) => {
        if (!/^[0-9]{2}$/.test(key)) {
          throw new Error("Key should be a 2-digit numeric key code");
        }

        if (component === null) {
          return "";
        }

        const encodedValue = typeof component === "string" ? component : component.toString();

        if (encodedValue.length > 99) {
          throw new Error("Encoded length should not exceed 99");
        }

        return `${key}${encodedValue.length.toString().padStart(2, "0")}${encodedValue}`;
      })
      .join("");
  }

  toStringWithCRC(): string {
    const result = `${this.toString()}6304`;
    const checksum = crc16ccitt(new TextEncoder().encode(result))
      .toString(16)
      .padStart(4, "0")
      .toUpperCase();

    return `${result}${checksum}`;
  }
}

const storagePrefix = "paynow-qr-generator:";
const imageWidth = 2000;

type QueryState = {
  mode: PayNowMode;
  target: string;
  reference: string;
  amount: string;
  view: "app" | "image";
};

function getQueryState(): QueryState | null {
  const params = new URLSearchParams(window.location.search);
  const hasKnownParams = ["mode", "target", "reference", "amount", "view"].some((key) =>
    params.has(key)
  );

  if (!hasKnownParams) {
    return null;
  }

  return {
    mode: params.get("mode") === "uen" ? "uen" : "phone",
    target: params.get("target") ?? "",
    reference: params.get("reference") ?? "",
    amount: params.get("amount") ?? "",
    view: isImagePagePath() || params.get("view") === "image" ? "image" : "app",
  };
}

function isImagePagePath(): boolean {
  return window.location.pathname.endsWith("/image.html") || window.location.pathname.endsWith("image.html");
}

const initialQueryState = getQueryState();

const canvasRef = ref<HTMLCanvasElement | null>(null);
const logoRef = ref<HTMLImageElement | null>(null);

const mode = ref<PayNowMode>(initialQueryState?.mode ?? getStoredMode());
const target = ref(initialQueryState?.target ?? getStoredValue("target"));
const reference = ref(initialQueryState?.reference ?? getStoredValue("reference"));
const amount = ref(initialQueryState?.amount ?? getStoredValue("amount"));
const isImageMode = ref(initialQueryState?.view === "image");
const qrcodeDataURL = ref("");
const qrcodeDataTarget = ref("");

function getStoredValue(key: string): string {
  try {
    return window.localStorage.getItem(`${storagePrefix}${key}`) ?? "";
  } catch {
    return "";
  }
}

function getStoredMode(): PayNowMode {
  const stored = getStoredValue("mode");
  return stored === "uen" ? "uen" : "phone";
}

function setStoredValue(key: string, value: string): void {
  try {
    window.localStorage.setItem(`${storagePrefix}${key}`, value);
  } catch {
    // Local storage can be unavailable in private contexts; ignore that gracefully.
  }
}

const standardizedTarget = computed(() => {
  if (mode.value !== "phone") {
    return target.value.toUpperCase();
  }

  const cleaned = target.value.replace(/[\s-]/g, "");
  if (/^[0-9]{8}$/.test(cleaned)) {
    return `+65${cleaned}`;
  }

  if (/^\+65[0-9]{8}$/.test(cleaned)) {
    return cleaned;
  }

  return cleaned;
});

const normalizedAmount = computed(() => {
  const trimmed = amount.value.trim();
  if (!trimmed) {
    return "";
  }

  if (!/^\d+(\.\d{1,2})?$/.test(trimmed)) {
    return "";
  }

  const parsed = Number(trimmed);
  if (!Number.isFinite(parsed) || parsed <= 0) {
    return "";
  }

  return parsed.toFixed(2);
});

const targetError = computed(() => {
  if (!target.value.trim()) {
    return `Enter a ${mode.value === "phone" ? "phone number" : "UEN"} first.`;
  }

  if (mode.value === "phone") {
    return /^\+65[0-9]{8}$/.test(standardizedTarget.value)
      ? ""
      : "Use an 8-digit Singapore mobile number, with or without the +65 prefix.";
  }

  return /^[0-9A-Z]{8,15}$/.test(standardizedTarget.value)
    ? ""
    : "Use 8 to 15 uppercase letters or digits for the UEN.";
});

const referenceError = computed(() => {
  if (!reference.value) {
    return "";
  }

  return /^[A-Za-z0-9\-/. ]{1,25}$/.test(reference.value)
    ? ""
    : "Reference can only contain letters, numbers, spaces, and - / . characters.";
});

const amountError = computed(() => {
  if (!amount.value.trim()) {
    return "";
  }

  if (!/^\d+(\.\d{1,2})?$/.test(amount.value.trim())) {
    return "Amount must be a positive number with up to 2 decimal places.";
  }

  if (!normalizedAmount.value || Number(normalizedAmount.value) <= 0) {
    return "Amount must be greater than zero.";
  }

  return normalizedAmount.value.length <= 13
    ? ""
    : "Amount is too long for the QR payload.";
});

const validationMessage = computed(
  () => targetError.value || referenceError.value || amountError.value || ""
);

const displayTarget = computed(() => {
  if (!standardizedTarget.value) {
    return mode.value === "phone" ? "Waiting for phone number" : "Waiting for UEN";
  }

  return mode.value === "phone"
    ? standardizedTarget.value.replace(/^\+65/, "")
    : standardizedTarget.value;
});

const downloadFilename = computed(() => {
  const identifier = displayTarget.value.replace(/[^A-Za-z0-9]+/g, "-").replace(/^-|-$/g, "");
  return `${mode.value}-${identifier || "paynow"}-qr.png`;
});

const apiImageUrl = computed(() => {
  const url = new URL(window.location.href);
  url.pathname = `${import.meta.env.BASE_URL}image.html`;
  url.searchParams.set("mode", mode.value);
  url.searchParams.set("target", target.value);

  if (reference.value) {
    url.searchParams.set("reference", reference.value);
  } else {
    url.searchParams.delete("reference");
  }

  if (amount.value) {
    url.searchParams.set("amount", amount.value);
  } else {
    url.searchParams.delete("amount");
  }
  url.searchParams.delete("view");
  return url.toString();
});

const qrPayload = computed(() => {
  if (validationMessage.value) {
    return "";
  }

  const hasFixedAmount = Boolean(normalizedAmount.value);

  return new QRData({
    "00": "01",
    "01": hasFixedAmount ? "12" : "11",
    "26": new QRData({
      "00": "SG.PAYNOW",
      "01": mode.value === "phone" ? "0" : "2",
      "02": standardizedTarget.value,
      "03": hasFixedAmount ? "0" : "1",
      "05": reference.value || null,
    }),
    "52": "0000",
    "53": "702",
    "54": normalizedAmount.value || null,
    "58": "SG",
    "59": "PAYNOW",
    "60": "Singapore",
    "62": new QRData({
      "01": reference.value || null,
    }),
  }).toStringWithCRC();
});

const renderQRCode = throttle(async () => {
  setStoredValue("mode", mode.value);
  setStoredValue("target", target.value);
  setStoredValue("reference", reference.value);
  setStoredValue("amount", amount.value);
  syncUrlState();

  if (!canvasRef.value || !qrPayload.value) {
    qrcodeDataURL.value = "";
    qrcodeDataTarget.value = "";
    return;
  }

  try {
    await qrcode.toCanvas(canvasRef.value, qrPayload.value, {
      errorCorrectionLevel: "M",
      width: imageWidth,
      margin: 2,
      color: {
        dark: "#7f234f",
        light: "#fffaf4",
      },
    });

    const context = canvasRef.value.getContext("2d");
    const logoElement = logoRef.value;

    if (context && logoElement?.complete && logoElement.naturalWidth > 0) {
      const sourceWidth = 1323;
      const sourceHeight = 894;
      const targetWidth = imageWidth / 3.8;
      const targetHeight = (sourceHeight / sourceWidth) * targetWidth;
      const midpoint = imageWidth / 2;

      context.imageSmoothingEnabled = true;
      context.fillStyle = "#fffaf4";
      context.fillRect(
        midpoint - targetWidth / 2,
        midpoint - targetHeight / 2,
        targetWidth,
        targetHeight
      );
      context.drawImage(
        logoElement,
        midpoint - targetWidth / 2,
        midpoint - targetHeight / 2,
        targetWidth,
        targetHeight
      );
    }

    qrcodeDataURL.value = canvasRef.value.toDataURL("image/png");
    qrcodeDataTarget.value = standardizedTarget.value;
  } catch {
    qrcodeDataURL.value = "";
    qrcodeDataTarget.value = "";
  }
}, 200);

watch([mode, target, reference, amount, qrPayload, canvasRef, logoRef], renderQRCode, {
  immediate: true,
});

onMounted(() => {
  document.body.classList.toggle("image-mode-body", isImageMode.value);
  renderQRCode();

  if (logoRef.value && !logoRef.value.complete) {
    logoRef.value.addEventListener("load", () => renderQRCode(), { once: true });
  }
});

watch(isImageMode, (value) => {
  document.body.classList.toggle("image-mode-body", value);
});

const print = () => window.print();

function syncUrlState(): void {
  const url = new URL(window.location.href);
  url.searchParams.set("mode", mode.value);
  url.searchParams.set("target", target.value);

  if (reference.value) {
    url.searchParams.set("reference", reference.value);
  } else {
    url.searchParams.delete("reference");
  }

  if (amount.value) {
    url.searchParams.set("amount", amount.value);
  } else {
    url.searchParams.delete("amount");
  }

  if (isImageMode.value) {
    url.searchParams.set("view", "image");
  } else {
    url.searchParams.delete("view");
  }

  window.history.replaceState({}, "", url.toString());
}
</script>

<style scoped>
.logo-source,
.hidden-canvas {
  display: none;
}

.page-shell {
  width: min(1120px, calc(100% - 32px));
  margin: 0 auto;
  padding: 32px 0 56px;
}

.page-shell.image-mode {
  width: 100%;
  min-height: 100vh;
  display: grid;
  place-items: center;
  padding: 0;
}

.image-mode-shell {
  display: grid;
  place-items: center;
  width: 100%;
  min-height: 100vh;
  background: #ffffff;
}

.api-image {
  display: block;
  width: min(100vw, 960px);
  height: auto;
}

.image-mode-error {
  max-width: 28ch;
  padding: 20px 24px;
  border-radius: 18px;
  background: rgba(255, 250, 244, 0.96);
  text-align: center;
  color: #8d2f47;
}

.hero {
  margin-bottom: 24px;
  padding: 32px;
  border-radius: 28px;
  background:
    radial-gradient(circle at top right, rgba(238, 165, 120, 0.28), transparent 36%),
    linear-gradient(135deg, rgba(127, 35, 79, 0.96), rgba(42, 22, 31, 0.96));
  color: #fff8f1;
  box-shadow: 0 28px 60px rgba(45, 22, 33, 0.25);
}

.eyebrow,
.panel-kicker,
.summary-label,
.caption-label,
.preview-badge {
  letter-spacing: 0.12em;
  text-transform: uppercase;
  font-size: 0.74rem;
  font-weight: 700;
}

.hero h1,
.panel-header h2,
.preview-card h2,
.reference-panel h2 {
  margin: 0;
  font-family: Georgia, "Times New Roman", serif;
  line-height: 1.05;
}

.hero h1 {
  max-width: 10ch;
  font-size: clamp(2.6rem, 4vw, 4.6rem);
}

.hero-copy {
  max-width: 56ch;
  margin: 16px 0 0;
  font-size: 1.05rem;
  line-height: 1.7;
  color: rgba(255, 248, 241, 0.86);
}

.workspace {
  display: grid;
  grid-template-columns: minmax(0, 1.05fr) minmax(320px, 0.95fr);
  gap: 24px;
  align-items: start;
}

.panel,
.reference-panel {
  border: 1px solid rgba(127, 35, 79, 0.1);
  border-radius: 28px;
  background: rgba(255, 250, 244, 0.94);
  box-shadow: 0 18px 45px rgba(71, 37, 49, 0.08);
}

.form-panel {
  padding: 28px;
}

.panel-header p,
.preview-copy,
.reference-panel p,
.field-hint,
.caption-meta {
  color: var(--muted-ink);
}

.panel-header p {
  margin: 10px 0 0;
  line-height: 1.6;
}

.mode-picker {
  display: flex;
  gap: 12px;
  margin: 24px 0;
}

.mode-pill,
.primary-action,
.secondary-action {
  appearance: none;
  border: 0;
  border-radius: 999px;
  cursor: pointer;
  text-decoration: none;
  transition:
    transform 160ms ease,
    box-shadow 160ms ease,
    background-color 160ms ease,
    color 160ms ease,
    border-color 160ms ease;
}

.mode-pill {
  padding: 12px 18px;
  background: #f7e8dd;
  color: #6d2a48;
  font-weight: 700;
}

.mode-pill.active {
  background: #7f234f;
  color: #fffaf4;
  box-shadow: 0 10px 24px rgba(127, 35, 79, 0.24);
}

.field-group + .field-group {
  margin-top: 18px;
}

.api-link-group {
  margin-top: 22px;
}

.field-label {
  display: block;
  margin-bottom: 10px;
  font-weight: 700;
  color: #2c1722;
}

input {
  width: 100%;
  padding: 14px 16px;
  border: 1px solid rgba(127, 35, 79, 0.16);
  border-radius: 16px;
  background: #fffdfa;
  color: #2c1722;
  font-size: 1rem;
  box-sizing: border-box;
}

input:focus {
  outline: 2px solid rgba(127, 35, 79, 0.24);
  border-color: rgba(127, 35, 79, 0.45);
}

.field-hint,
.field-error {
  margin: 8px 0 0;
  font-size: 0.93rem;
  line-height: 1.45;
}

.field-error {
  color: #b33246;
}

.summary-strip {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px;
  margin-top: 24px;
  padding: 18px;
  border-radius: 20px;
  background: linear-gradient(135deg, rgba(247, 232, 221, 0.86), rgba(255, 250, 244, 0.96));
}

.summary-strip strong {
  display: block;
  margin-top: 4px;
  color: #411d30;
}

.preview-panel {
  padding: 22px;
}

.preview-panel.invalid {
  border-color: rgba(179, 50, 70, 0.22);
}

.preview-card {
  padding: 22px;
  border-radius: 24px;
  background:
    radial-gradient(circle at top left, rgba(255, 214, 186, 0.48), transparent 30%),
    linear-gradient(180deg, rgba(255, 255, 255, 0.95), rgba(255, 248, 241, 0.95));
}

.preview-badge {
  display: inline-block;
  padding: 8px 12px;
  border-radius: 999px;
  background: rgba(127, 35, 79, 0.1);
  color: #7f234f;
}

.preview-card h2 {
  margin-top: 14px;
  font-size: clamp(2rem, 3.2vw, 3rem);
  color: #24131b;
}

.preview-copy {
  margin: 10px 0 24px;
  line-height: 1.6;
}

.qr-frame {
  display: grid;
  place-items: center;
  min-height: 340px;
  padding: 18px;
  border-radius: 24px;
  background: #fffaf4;
  border: 1px solid rgba(127, 35, 79, 0.08);
}

.qr-frame.disabled {
  border-style: dashed;
}

.generated-qr {
  width: 100%;
  max-width: 380px;
  height: auto;
  display: block;
}

.empty-state {
  max-width: 26ch;
  text-align: center;
  color: var(--muted-ink);
  line-height: 1.6;
}

.caption-block {
  margin-top: 22px;
  text-align: center;
}

.caption-label {
  margin: 0;
  color: #966077;
}

.caption-value {
  margin: 8px 0 0;
  font-size: clamp(1.8rem, 4vw, 2.5rem);
  font-weight: 700;
  color: #2b1620;
}

.caption-meta {
  margin: 8px 0 0;
}

.action-row {
  display: flex;
  gap: 12px;
  margin-top: 24px;
}

.primary-action,
.secondary-action {
  flex: 1;
  padding: 14px 18px;
  text-align: center;
  font-weight: 700;
}

.primary-action {
  background: #7f234f;
  color: #fffaf4;
  box-shadow: 0 14px 28px rgba(127, 35, 79, 0.2);
}

.secondary-action {
  border: 1px solid rgba(127, 35, 79, 0.16);
  background: transparent;
  color: #6d2a48;
}

.primary-action:disabled,
.secondary-action.disabled {
  opacity: 0.45;
  cursor: not-allowed;
  pointer-events: none;
  box-shadow: none;
}

.mode-pill:hover,
.primary-action:hover,
.secondary-action:hover {
  transform: translateY(-1px);
}

.reference-panel {
  margin-top: 24px;
  padding: 24px 28px;
}

.reference-panel p {
  margin: 10px 0 0;
  line-height: 1.65;
}

@media (max-width: 900px) {
  .workspace {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 640px) {
  .page-shell {
    width: min(100% - 20px, 1120px);
    padding-top: 20px;
  }

  .hero,
  .form-panel,
  .preview-panel,
  .reference-panel,
  .preview-card {
    padding: 20px;
  }

  .mode-picker,
  .action-row,
  .summary-strip {
    grid-template-columns: 1fr;
    flex-direction: column;
  }

  .qr-frame {
    min-height: 280px;
  }
}

@media print {
  .no-print {
    display: none !important;
  }

  .page-shell {
    width: 100%;
    margin: 0;
    padding: 0;
  }

  .workspace {
    display: block;
  }

  .preview-panel,
  .preview-card,
  .qr-frame {
    border: 0;
    box-shadow: none;
    background: #ffffff;
    padding: 0;
  }

  .preview-card h2,
  .preview-copy,
  .preview-badge {
    display: none;
  }

  .generated-qr {
    max-width: 82mm;
    margin: 0 auto;
  }

  .caption-value {
    font-size: 20pt;
  }

  .caption-meta {
    color: #333333;
  }
}
</style>
