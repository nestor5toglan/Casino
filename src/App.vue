<script setup>
import { computed, provide, ref, watch } from 'vue'
import Login from './components/Login.vue'
import Game from './components/Game.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'
import { createTranslator } from './i18n'

const storedLang = localStorage.getItem('casino_lang') || 'en'
const lang = ref(storedLang)
const t = createTranslator(lang)

const session = ref(JSON.parse(localStorage.getItem('casino_session') || 'null'))
const hasSession = computed(() => !!session.value?.username)

provide('lang', lang)
provide('t', t)

watch(lang, value => {
  localStorage.setItem('casino_lang', value)
  document.documentElement.lang = value
}, { immediate: true })

function login(username) {
  session.value = {
    username,
    balance: 1000,
    lastPrize: null,
    createdAt: Date.now()
  }
  saveSession()
}

function updateSession(payload) {
  session.value = { ...session.value, ...payload }
  saveSession()
}

function saveSession() {
  localStorage.setItem('casino_session', JSON.stringify(session.value))
}

function logout() {
  session.value = null
  localStorage.removeItem('casino_session')
}
</script>

<template>
  <div class="app-shell">
    <div class="ambient ambient-one"></div>
    <div class="ambient ambient-two"></div>

    <header class="topbar">
      <div class="brand">
        <div class="brand-mark">◆</div>
        <div>
          <strong>{{ t('appTitle') }}</strong>
          <span>{{ t('appSubtitle') }}</span>
        </div>
      </div>
      <LanguageSwitcher />
    </header>

    <Login v-if="!hasSession" @login="login" />
    <Game v-else :session="session" @update-session="updateSession" @logout="logout" />

    <footer class="app-footer">{{ t('footerNote') }}</footer>
  </div>
</template>
