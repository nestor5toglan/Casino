<script setup>
import { inject, ref } from 'vue'
const emit = defineEmits(['login'])
const t = inject('t')
const username = ref('')
const error = ref('')

function submit() {
  const value = username.value.trim()
  if (value.length < 2) {
    error.value = 'Minimum 2 characters'
    return
  }
  emit('login', value)
}
</script>

<template>
  <main class="login-screen">
    <section class="login-card glass-card">
      <div class="crown">♛</div>
      <p class="eyebrow">{{ t('freeCredits') }}</p>
      <h1>{{ t('welcome') }} — {{ t('appTitle') }}</h1>
      <p class="muted">{{ t('loginIntro') }}</p>
      <form @submit.prevent="submit" class="login-form">
        <label>{{ t('username') }}</label>
        <input v-model="username" :placeholder="t('enterUsername')" maxlength="24" autofocus />
        <p v-if="error" class="error-text">{{ error }}</p>
        <button class="gold-button" type="submit">{{ t('startGame') }}</button>
      </form>
    </section>
  </main>
</template>
