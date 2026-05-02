<script setup>
import { computed, inject } from 'vue'
const props = defineProps({ result: { type: Object, required: true } })
const emit = defineEmits(['close'])
const t = inject('t')
const isLose = computed(() => props.result.type === 'lose')
const title = computed(() => isLose.value ? t('youLost') : `${t('youWon')} ${props.result.value.toLocaleString()}`)
</script>

<template>
  <div class="modal-backdrop" @click.self="emit('close')">
    <section class="result-modal glass-card" :class="{ win: !isLose, lose: isLose }">
      <div class="result-icon">{{ result.icon }}</div>
      <p class="eyebrow">{{ t('result') }}</p>
      <h2>{{ title }}</h2>
      <p v-if="!isLose">{{ t('winMessage') }} <strong>{{ result.label }}</strong>. {{ t('rewardAdded') }}</p>
      <p v-else>{{ t('loseMessage') }}</p>
      <button class="gold-button" @click="emit('close')">{{ t('close') }}</button>
    </section>
  </div>
</template>
