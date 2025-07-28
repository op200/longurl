<script setup lang="ts">
import { computed, onBeforeMount, ref } from 'vue';

// 多语言支持
const languageMap = {
  en: {
    title: "URL Converter",
    inputLabel: "Enter URL",
    placeholder: "https://example.com",
    error: "Please enter a valid URL (e.g. https://example.com)",
    resultLabel: "Converted URL",
    copyButton: "Copy Link",
    copied: "✓ Copied",
    previewOriginal: "Original URL",
    previewEncoded: "Encoded Result",
    characterCount: "Character Count",
    originalCount: "Original:",
    encodedCount: "Encoded:",
    language: "Language"
  },
  zh: {
    title: "URL 转换器",
    inputLabel: "输入 URL",
    placeholder: "https://example.com",
    error: "请输入有效的URL (如：https://example.com)",
    resultLabel: "转换结果",
    copyButton: "复制链接",
    copied: "✓ 已复制",
    previewOriginal: "原始 URL",
    previewEncoded: "编码结果",
    characterCount: "字数统计",
    originalCount: "原始长度:",
    encodedCount: "编码后长度:",
    language: "语言"
  },
  es: {
    title: "Conversor de URL",
    inputLabel: "Ingrese URL",
    placeholder: "https://ejemplo.com",
    error: "Por favor ingrese una URL válida (ej. https://ejemplo.com)",
    resultLabel: "Resultado convertido",
    copyButton: "Copiar enlace",
    copied: "✓ Copiado",
    previewOriginal: "URL original",
    previewEncoded: "Resultado codificado",
    characterCount: "Recuento de caracteres",
    originalCount: "Original:",
    encodedCount: "Codificado:",
    language: "Idioma"
  },
  ja: {
    title: "URL コンバーター",
    inputLabel: "URL 入力",
    placeholder: "https://example.com",
    error: "有効な URL を入力してください (例: https://example.com)",
    resultLabel: "変換結果",
    copyButton: "リンクをコピー",
    copied: "✓ コピー済み",
    previewOriginal: "元の URL",
    previewEncoded: "エンコード結果",
    characterCount: "文字数カウント",
    originalCount: "元の長さ:",
    encodedCount: "エンコード後:",
    language: "言語"
  },
  de: {
    title: "URL-Konverter",
    inputLabel: "URL eingeben",
    placeholder: "https://beispiel.de",
    error: "Bitte geben Sie eine gültige URL ein (z.B. https://beispiel.de)",
    resultLabel: "Konvertiertes Ergebnis",
    copyButton: "Link kopieren",
    copied: "✓ Kopiert",
    previewOriginal: "Original-URL",
    previewEncoded: "Verschlüsseltes Ergebnis",
    characterCount: "Zeichenzählung",
    originalCount: "Original:",
    encodedCount: "Verschlüsselt:",
    language: "Sprache"
  },
  ru: {
    title: "Конвертер URL",
    inputLabel: "Введите URL",
    placeholder: "https://example.com",
    error: "Пожалуйста, введите действительный URL (например: https://example.com)",
    resultLabel: "Результат конвертации",
    copyButton: "Копировать ссылку",
    copied: "✓ Скопировано",
    previewOriginal: "Исходный URL",
    previewEncoded: "Зашифрованный результат",
    characterCount: "Подсчет символов",
    originalCount: "Оригинальная длина:",
    encodedCount: "Закодированная длина:",
    language: "Язык"
  }
};

const availableLanguages = Object.keys(languageMap);
const userLang = navigator.language.substring(0, 2) as keyof typeof languageMap;
const currentLang = ref<keyof typeof languageMap>(
  availableLanguages.includes(userLang) ? userLang : 'en'
);
const t = computed(() => languageMap[currentLang.value]);

function switchLanguage() {
  const index = availableLanguages.indexOf(currentLang.value);
  currentLang.value = availableLanguages[(index + 1) % availableLanguages.length] as keyof typeof languageMap;
}

// URL 功能
const URL_REGEX = /^(https?|ftp):\/\/[^\s/$.?#].[^\s]*$/i;

const current_url = new URL(window.location.href);
const input_url = ref<string>('');
const copied = ref(false);
const error = ref<string | null>(null);

const output_url = computed(() =>
  input_url.value ? encode_long_url(input_url.value) : ''
);

const jump_url = computed(() =>
  output_url.value
    ? `${current_url.protocol}//${current_url.host}${current_url.pathname}?url=${output_url.value}`
    : ''
);

const long_url_codec_time = 2;

function encode_long_url(url: string): string {
  const base64 = btoa(url);
  return base64.repeat(long_url_codec_time);
}

function decode_long_url(longUrl: string): string {
  if (!longUrl) return '';
  const base64 = longUrl.slice(0, longUrl.length / long_url_codec_time);
  try {
    return atob(base64);
  } catch {
    return '';
  }
}

function validateUrl() {
  if (input_url.value && !URL_REGEX.test(input_url.value)) {
    error.value = t.value.error;
    return false;
  }
  error.value = null;
  return true;
}

function copy_jump_url() {
  if (!validateUrl()) return;

  navigator.clipboard.writeText(jump_url.value);
  copied.value = true;
  setTimeout(() => { copied.value = false; }, 2000);
}

// 字数统计
const originalCount = computed(() => input_url.value.length);
const encodedCount = computed(() => output_url.value.length);

onBeforeMount(() => {
  const params = new URLSearchParams(window.location.search);
  const encodedUrl = params.get('url');
  if (encodedUrl) {
    const decoded = decode_long_url(encodedUrl);
    if (decoded && URL_REGEX.test(decoded)) {
      window.location.href = decoded;
    }
  }
});
</script>

<template>
  <div class="container">
    <div class="header-row">
      <h1>{{ t.title }}</h1>
      <button @click="switchLanguage" class="lang-btn">
        {{ t.language }}: {{ currentLang.toUpperCase() }}
      </button>
    </div>

    <div class="input-section">
      <label>{{ t.inputLabel }}</label>
      <input type="text" v-model="input_url" @input="validateUrl" @blur="validateUrl" :placeholder="t.placeholder"
        :class="{ error: error }">
      <div v-if="error" class="error-msg">{{ error }}</div>
    </div>

    <div class="result-section">
      <div class="result-card">
        <p class="result-label">{{ t.resultLabel }}</p>
        <a :href="jump_url" class="result-url">{{ jump_url }}</a>

        <div class="action-row">
          <button @click="copy_jump_url" :class="{ copied }">
            {{ copied ? t.copied : t.copyButton }}
          </button>
        </div>
      </div>

      <div class="preview-section">
        <div class="preview-card">
          <p class="preview-label">{{ t.previewOriginal }}</p>
          <p class="preview-content">{{ input_url || 'N/A' }}</p>
        </div>

        <div class="preview-card">
          <p class="preview-label">{{ t.previewEncoded }}</p>
          <p class="preview-content">{{ output_url || 'N/A' }}</p>
        </div>

        <div class="preview-card">
          <p class="preview-label">{{ t.characterCount }}</p>
          <div class="count-display">
            <div>
              <span class="count-label">{{ t.originalCount }}</span>
              <span class="count-value">{{ originalCount }}</span>
            </div>
            <div>
              <span class="count-label">{{ t.encodedCount }}</span>
              <span class="count-value">{{ encodedCount }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 基础平面化设计样式 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.container {
  max-width: 800px;
  padding: 2rem 1.5rem;
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

h1 {
  color: #2c3e50;
  margin-bottom: 1rem;
  font-weight: 600;
  font-size: 2rem;
}

/* 语言切换按钮 */
.lang-btn {
  background: none;
  border: 1.5px solid #3498db;
  color: #3498db;
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 0.9rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s;
  outline: none;
}

.lang-btn:hover {
  background: #3498db;
  color: white;
}

/* 输入区域样式 */
.input-section {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

label {
  font-weight: 500;
  color: #34495e;
}

input {
  padding: 14px 16px;
  border-radius: 8px;
  border: 2px solid #e0e0e0;
  font-size: 1rem;
  transition: border-color 0.3s;
}

input:focus {
  outline: none;
  border-color: #3498db;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
}

input.error {
  border-color: #e74c3c;
}

.error-msg {
  color: #e74c3c;
  font-size: 0.85rem;
  padding: 4px 8px;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-5px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 结果区域样式 */
.result-section {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.result-card {
  background: white;
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.result-label {
  font-size: 1.1rem;
  font-weight: 600;
  color: #2c3e50;
}

.result-url {
  word-break: break-all;
  padding: 12px 16px;
  background: #f8f9fa;
  border-radius: 8px;
  color: #3498db;
  text-decoration: none;
  transition: background-color 0.3s;
  border: 1px solid #e9ecef;
  line-height: 1.4;
}

.result-url:hover {
  background-color: #e3f2fd;
  text-decoration: underline;
}

/* 按钮样式 */
button {
  background: #3498db;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.3s;
  width: 100%;
  max-width: 200px;
  margin: 0 auto;
}

button:hover {
  background: #2980b9;
}

button.copied {
  background: #27ae60;
}

.action-row {
  display: flex;
  justify-content: center;
  margin-top: 0.5rem;
}

/* 预览区域样式 */
.preview-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
}

.preview-card {
  background: white;
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.preview-label {
  font-weight: 500;
  color: #34495e;
  font-size: 1.05rem;
}

.preview-content {
  word-break: break-all;
  padding: 12px;
  background: #f8f9fa;
  border-radius: 8px;
  color: #7f8c8d;
  min-height: 60px;
  border: 1px solid #e9ecef;
  overflow: auto;
}

.count-display {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
  padding: 8px 0;
}

.count-display>div {
  display: flex;
  justify-content: space-between;
  padding-bottom: 6px;
  border-bottom: 1px dashed #e0e0e0;
}

.count-label {
  font-weight: 500;
  color: #7f8c8d;
}

.count-value {
  font-weight: 600;
  color: #2c3e50;
  font-family: monospace;
  background: #eef7ff;
  padding: 2px 8px;
  border-radius: 4px;
  min-width: 50px;
  text-align: center;
}

/* 响应式设计 */
@media (max-width: 600px) {
  .preview-section {
    grid-template-columns: 1fr;
  }

  .result-card {
    padding: 1.2rem;
  }

  .container {
    padding: 1.5rem 1rem;
  }

  .header-row {
    flex-direction: column;
    align-items: flex-start;
    gap: 1rem;
  }

  .lang-btn {
    align-self: flex-end;
  }

  button {
    max-width: 100%;
  }
}
</style>