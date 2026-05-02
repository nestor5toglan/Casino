<script setup>
import { inject } from 'vue'
const lang = inject('lang')
const t = inject('t')
</script>

<template>
  <div class="language-switcher" :aria-label="t('language')">
    <button :class="{ active: lang === 'en' }" @click="lang = 'en'">🇬🇧 EN</button>
    <button :class="{ active: lang === 'fr' }" @click="lang = 'fr'">🇫🇷 FR</button>
  </div>
</template>
