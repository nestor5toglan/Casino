<script setup>
import { computed, inject, ref } from 'vue'
import WheelCanvas from './WheelCanvas.vue'
import ResultModal from './ResultModal.vue'

const props = defineProps({ session: { type: Object, required: true } })
const emit = defineEmits(['update-session', 'logout'])
const t = inject('t')

const bet = ref(50)
const spinning = ref(false)
const error = ref('')
const result = ref(null)
const wheelRef = ref(null)

const balance = computed(() => props.session.balance)

const segments = [
  { id: 1, type: 'money', label: '10', icon: '💰', value: 10, weight: 15, colorA: '#d4af37', colorB: '#7a1414' },
  { id: 2, type: 'lose', label: 'LOSE', icon: '☠️', value: 0, weight: 18, colorA: '#111827', colorB: '#3f0d12' },
  { id: 3, type: 'money', label: '50', icon: '💰', value: 50, weight: 12, colorA: '#941b1b', colorB: '#f2c14e' },
  { id: 4, type: 'motorcycle', label: 'MOTO', icon: '🏍️', value: 700, weight: 3, colorA: '#161616', colorB: '#b8860b' },
  { id: 5, type: 'money', label: '100', icon: '💰', value: 100, weight: 10, colorA: '#b8860b', colorB: '#250909' },
  { id: 6, type: 'lose', label: 'LOSE', icon: '☠️', value: 0, weight: 18, colorA: '#2b0f14', colorB: '#0f172a' },
  { id: 7, type: 'money', label: '250', icon: '💰', value: 250, weight: 7, colorA: '#f6d365', colorB: '#7f1d1d' },
  { id: 8, type: 'car', label: 'CAR', icon: '🚗', value: 1500, weight: 2, colorA: '#141414', colorB: '#c99b28' },
  { id: 9, type: 'money', label: '500', icon: '💰', value: 500, weight: 5, colorA: '#991b1b', colorB: '#fbbf24' },
  { id: 10, type: 'lose', label: 'LOSE', icon: '☠️', value: 0, weight: 15, colorA: '#0b1120', colorB: '#381010' },
  { id: 11, type: 'money', label: '1000', icon: '💰', value: 1000, weight: 3, colorA: '#d4af37', colorB: '#991b1b' },
  { id: 12, type: 'jackpot', label: 'JACKPOT', icon: '💎', value: 3000, weight: 1, colorA: '#f9d976', colorB: '#3b0764' },
  { id: 13, type: 'money', label: '25', icon: '💰', value: 25, weight: 13, colorA: '#7f1d1d', colorB: '#d4af37' },
  { id: 14, type: 'lose', label: 'LOSE', icon: '☠️', value: 0, weight: 16, colorA: '#101010', colorB: '#4c0519' }
]

function pickWeightedSegment() {
  const total = segments.reduce((sum, s) => sum + s.weight, 0)
  let r = Math.random() * total
  for (const segment of segments) {
    r -= segment.weight
    if (r <= 0) return segment
  }
  return segments[0]
}

function validateBet() {
  const value = Number(bet.value)
  if (!Number.isFinite(value) || value <= 0) return t('invalidBet')
  if (value < 10) return t('minBet')
  if (value > balance.value) return t('maxBet')
  return ''
}

async function spin() {
  error.value = validateBet()
  if (error.value || spinning.value) return
  spinning.value = true
  result.value = null

  const betValue = Number(bet.value)
  emit('update-session', { balance: balance.value - betValue })

  const selected = pickWeightedSegment()
  const landed = await wheelRef.value.spinTo(selected.id)
  const multiplierBonus = selected.type === 'money' ? Math.round(selected.value + betValue * 0.25) : selected.value
  const rewardValue = selected.type === 'lose' ? 0 : multiplierBonus

  emit('update-session', {
    balance: balance.value + rewardValue,
    lastPrize: { ...landed, value: rewardValue, at: Date.now() }
  })
  result.value = { ...landed, value: rewardValue }
  spinning.value = false
}

function resetBalance() {
  emit('update-session', { balance: 1000, lastPrize: null })
}
</script>

<template>
  <main class="game-layout">
    <section class="player-panel glass-card">
      <div>
        <p class="eyebrow">{{ t('welcome') }}</p>
        <h2>{{ session.username }}</h2>
      </div>
      <div class="balance-card">
        <span>{{ t('balance') }}</span>
        <strong>{{ balance.toLocaleString() }}</strong>
      </div>
      <div class="last-prize" v-if="session.lastPrize">
        <span>{{ t('lastPrize') }}</span>
        <strong>{{ session.lastPrize.icon }} {{ session.lastPrize.label }}</strong>
      </div>
      <button class="ghost-button" @click="resetBalance">{{ t('reset') }}</button>
      <button class="ghost-button" @click="emit('logout')">{{ t('logout') }}</button>
    </section>

    <section class="wheel-stage">
      <div class="machine-frame">
        <div class="pointer-wrap" :class="{ bounce: !spinning && result }">
          <div class="pointer"></div>
        </div>
        <WheelCanvas ref="wheelRef" :segments="segments" />
        <div class="machine-lights">
          <span v-for="n in 24" :key="n"></span>
        </div>
      </div>
      <p class="premium-line">{{ t('premiumFeel') }}</p>
    </section>

    <section class="controls-panel glass-card">
      <p class="eyebrow">{{ t('controls') }}</p>
      <h2>{{ t('chooseBet') }}</h2>
      <label>{{ t('bet') }}</label>
      <input v-model.number="bet" type="number" min="10" :max="balance" step="10" :disabled="spinning" />
      <p v-if="error" class="error-text">{{ error }}</p>
      <button class="gold-button spin-button" @click="spin" :disabled="spinning || balance < 10">
        <span v-if="!spinning">{{ t('spin') }}</span>
        <span v-else>{{ t('spinning') }}</span>
      </button>
    </section>

    <ResultModal v-if="result" :result="result" @close="result = null" />
  </main>
</template>
