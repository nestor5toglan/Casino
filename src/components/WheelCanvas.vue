<script setup>
import { nextTick, onMounted, ref, watch } from 'vue'

const props = defineProps({ segments: { type: Array, required: true } })
const canvas = ref(null)
const currentRotation = ref(0)
const highlightedId = ref(null)
const size = 680
let audioCtx = null
let animationFrame = null
let lastTickIndex = -1

function getCtx() {
  return canvas.value.getContext('2d')
}

function degToRad(deg) {
  return (deg * Math.PI) / 180
}

function normalizeDeg(deg) {
  return ((deg % 360) + 360) % 360
}

function playTone(freq = 700, duration = 0.035, gain = 0.03, type = 'square') {
  try {
    audioCtx ||= new (window.AudioContext || window.webkitAudioContext)()
    const osc = audioCtx.createOscillator()
    const amp = audioCtx.createGain()
    osc.type = type
    osc.frequency.value = freq
    amp.gain.value = gain
    osc.connect(amp)
    amp.connect(audioCtx.destination)
    osc.start()
    osc.stop(audioCtx.currentTime + duration)
  } catch (_) {}
}

function playWin() {
  playTone(523, 0.08, 0.04, 'sine')
  setTimeout(() => playTone(659, 0.08, 0.04, 'sine'), 90)
  setTimeout(() => playTone(784, 0.14, 0.05, 'sine'), 180)
}

function drawTextOnArc(ctx, text, radius, angle, color = '#fff') {
  ctx.save()
  ctx.rotate(angle)
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillStyle = color
  ctx.font = '900 20px Arial'
  ctx.shadowColor = 'rgba(0,0,0,.65)'
  ctx.shadowBlur = 6
  ctx.fillText(text, radius, 0)
  ctx.restore()
}

function drawWheel(rotationDeg = currentRotation.value) {
  const ctx = getCtx()
  const w = size
  const h = size
  const cx = w / 2
  const cy = h / 2
  const radius = 300
  const innerRadius = 72
  const count = props.segments.length
  const arc = (Math.PI * 2) / count

  ctx.clearRect(0, 0, w, h)
  ctx.save()
  ctx.translate(cx, cy)
  ctx.rotate(degToRad(rotationDeg))

  props.segments.forEach((segment, index) => {
    const start = index * arc - Math.PI / 2
    const end = start + arc
    const mid = start + arc / 2

    const grad = ctx.createRadialGradient(0, 0, innerRadius, 0, 0, radius)
    grad.addColorStop(0, '#fff7c2')
    grad.addColorStop(0.16, segment.colorA)
    grad.addColorStop(0.62, segment.colorB)
    grad.addColorStop(1, '#070707')

    ctx.beginPath()
    ctx.moveTo(0, 0)
    ctx.arc(0, 0, radius, start, end)
    ctx.closePath()
    ctx.fillStyle = grad
    ctx.fill()

    ctx.save()
    ctx.globalCompositeOperation = 'overlay'
    ctx.fillStyle = index % 2 ? 'rgba(255,255,255,.10)' : 'rgba(0,0,0,.10)'
    ctx.fill()
    ctx.restore()

    ctx.strokeStyle = 'rgba(255, 220, 130, .75)'
    ctx.lineWidth = 2
    ctx.stroke()

    if (highlightedId.value === segment.id) {
      ctx.save()
      ctx.beginPath()
      ctx.moveTo(0, 0)
      ctx.arc(0, 0, radius, start, end)
      ctx.closePath()
      ctx.fillStyle = 'rgba(255,255,255,.20)'
      ctx.shadowColor = '#ffd76a'
      ctx.shadowBlur = 35
      ctx.fill()
      ctx.restore()
    }

    ctx.save()
    ctx.rotate(mid)
    ctx.font = '42px Arial'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    ctx.shadowColor = 'rgba(0,0,0,.65)'
    ctx.shadowBlur = 8
    ctx.fillText(segment.icon, 205, 0)
    ctx.restore()

    drawTextOnArc(ctx, segment.label, 250, mid, '#fff7d6')
  })

  ctx.beginPath()
  ctx.arc(0, 0, radius, 0, Math.PI * 2)
  ctx.lineWidth = 18
  const rim = ctx.createLinearGradient(-radius, -radius, radius, radius)
  rim.addColorStop(0, '#5b3907')
  rim.addColorStop(.25, '#fff0a7')
  rim.addColorStop(.5, '#b8860b')
  rim.addColorStop(.75, '#fff4bf')
  rim.addColorStop(1, '#4a2d04')
  ctx.strokeStyle = rim
  ctx.shadowColor = 'rgba(255,207,83,.55)'
  ctx.shadowBlur = 20
  ctx.stroke()

  ctx.shadowBlur = 0
  ctx.beginPath()
  ctx.arc(0, 0, innerRadius, 0, Math.PI * 2)
  const hub = ctx.createRadialGradient(-20, -20, 10, 0, 0, innerRadius)
  hub.addColorStop(0, '#fff5c2')
  hub.addColorStop(.45, '#d4af37')
  hub.addColorStop(1, '#3a2608')
  ctx.fillStyle = hub
  ctx.fill()
  ctx.lineWidth = 6
  ctx.strokeStyle = '#fff0a8'
  ctx.stroke()

  ctx.fillStyle = '#120b03'
  ctx.font = '900 18px Arial'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillText('ROYAL', 0, -8)
  ctx.fillText('SPIN', 0, 14)

  ctx.restore()

  const shine = ctx.createLinearGradient(0, 0, w, h)
  shine.addColorStop(0, 'rgba(255,255,255,.18)')
  shine.addColorStop(.35, 'rgba(255,255,255,.02)')
  shine.addColorStop(.65, 'rgba(255,255,255,0)')
  shine.addColorStop(1, 'rgba(255,255,255,.12)')
  ctx.fillStyle = shine
  ctx.beginPath()
  ctx.arc(cx, cy, radius + 9, 0, Math.PI * 2)
  ctx.fill()
}

function easeOutCubic(t) {
  return 1 - Math.pow(1 - t, 3)
}

function getTickIndex(rotation) {
  const segmentSize = 360 / props.segments.length
  return Math.floor(normalizeDeg(-rotation) / segmentSize)
}

function spinTo(segmentId) {
  return new Promise(resolve => {
    const selectedIndex = props.segments.findIndex(s => s.id === segmentId)
    const segmentSize = 360 / props.segments.length
    const segmentCenter = selectedIndex * segmentSize + segmentSize / 2
    const pointerAngle = 0
    const base = normalizeDeg(currentRotation.value)
    const desiredMod = normalizeDeg(pointerAngle - segmentCenter)
    const delta = normalizeDeg(desiredMod - base)
    const fullSpins = 5 + Math.floor(Math.random() * 6)
    const totalRotation = fullSpins * 360 + delta
    const startRotation = currentRotation.value
    const endRotation = startRotation + totalRotation
    const duration = 5200 + Math.random() * 1300
    const startedAt = performance.now()
    highlightedId.value = null
    lastTickIndex = getTickIndex(startRotation)

    function animate(now) {
      const elapsed = now - startedAt
      const p = Math.min(elapsed / duration, 1)
      const eased = easeOutCubic(p)
      currentRotation.value = startRotation + totalRotation * eased
      const tickIndex = getTickIndex(currentRotation.value)
      if (tickIndex !== lastTickIndex) {
        lastTickIndex = tickIndex
        playTone(760 + Math.random() * 120, 0.02, 0.018)
      }
      drawWheel(currentRotation.value)
      if (p < 1) {
        animationFrame = requestAnimationFrame(animate)
      } else {
        currentRotation.value = endRotation
        highlightedId.value = segmentId
        drawWheel(currentRotation.value)
        playWin()
        resolve(props.segments[selectedIndex])
      }
    }
    cancelAnimationFrame(animationFrame)
    animationFrame = requestAnimationFrame(animate)
  })
}

defineExpose({ spinTo })

onMounted(() => nextTick(drawWheel))
watch(() => props.segments, () => drawWheel(), { deep: true })
</script>

<template>
  <canvas ref="canvas" class="wheel-canvas" :width="size" :height="size" aria-label="Casino spin wheel"></canvas>
</template>
