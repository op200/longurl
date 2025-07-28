<script setup lang="ts">
import { computed, onMounted, ref } from 'vue';


const long_url_codec_time = 2

function encode_long_url(url: string): string {
  const base64 = btoa(url);
  return base64.repeat(long_url_codec_time);
}

function decode_long_url(longUrl: string): string {
  if (!longUrl) return '';
  // base64编码长度
  const base64Length = btoa('a').length * longUrl.length;
  // 取原始base64部分
  const base64 = longUrl.slice(0, longUrl.length / long_url_codec_time);
  try {
    return atob(base64);
  } catch {
    return '';
  }
}


const current_url = new URL(window.location.href);


const input_url = ref<string>('');
const output_url = computed(() => {
  if (!input_url.value)
    return '';

  return encode_long_url(input_url.value)
})

const jump_url = computed(() => {
  if (!output_url.value)
    return '';

  return `${current_url.protocol}//${current_url.hostname}:${current_url.port}/?url=${output_url.value}`;
});


onMounted(() => {
  const params = new URLSearchParams(window.location.search);
  const encodedUrl = params.get('url');
  if (encodedUrl) {
    const decoded = decode_long_url(encodedUrl);
    console.info('Decoded URL:', decoded);
    if (decoded) {
      window.location.href = decoded;
    }
  }
});
</script>

<template>
  <div style="margin: auto;width: 100%;text-align: center;">
    <div class="line">
      <label for="">输入 URL</label>
      <input type="text" name="" id="" v-model="input_url">
    </div>

    <div>
      <div class="line">
        <p>输入的 URL:</p>
        <p>{{ input_url }}</p>
      </div>

      <div class="line">
        <p>输出的 URL:</p>
        <p>{{ output_url }}</p>
      </div>

      <div class="line">
        <p>解码预览 URL:</p>
        <p>{{ decode_long_url(output_url) }}</p>
      </div>
    </div>

    <div class="line">
      <a :href="jump_url">{{ jump_url }}</a>
    </div>

  </div>
</template>

<style scoped>
.line {
  width: 100%;

  display: flex;
  justify-content: center;
  gap: 1rem;

  text-align: center;
}
</style>
