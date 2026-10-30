<script setup>
import { useWindowScroll } from '@vueuse/core'

const { y } = useWindowScroll()
const scrolled = computed(() => y.value > 30)

const isMenuOpen = ref(false)
const toggleMenu = () => { isMenuOpen.value = !isMenuOpen.value }
const closeMenu  = () => { isMenuOpen.value = false }

// Lock body scroll when mobile menu is open
watch(isMenuOpen, (val) => {
  if (import.meta.client) document.body.style.overflow = val ? 'hidden' : ''
})

// ── Soft glitch page transition ───────────────────────────────────────────
// A light, low-contrast grain in the site's ivory tone dissolves in, the page
// swaps underneath, then the grain clears block by block with a few thin
// orange scan lines. Short and gentle on the eyes.
const router = useRouter()
const gCanvas = ref(null)
const gOn = ref(false)
let busy = false
const PX = 4          // size of one grain "pixel" on screen
const CELL = 10       // reveal block size, in grain pixels

function runGlitch (mode, dur) {
  return new Promise(resolve => {
    const cv = gCanvas.value
    if (!cv) return resolve()
    const W = Math.ceil(window.innerWidth / PX), H = Math.ceil(window.innerHeight / PX)
    cv.width = W; cv.height = H
    const ctx = cv.getContext('2d')
    const img = ctx.createImageData(W, H)
    const d = img.data
    const cols = Math.ceil(W / CELL), rows = Math.ceil(H / CELL)
    const thr = new Float32Array(cols * rows)
    for (let i = 0; i < thr.length; i++) thr[i] = Math.random()
    const t0 = performance.now()
    let last = 0
    const frame = (now) => {
      const p = Math.min(1, (now - t0) / dur)
      // redraw the grain ~30fps so it shimmers instead of strobing
      if (now - last > 33 || p === 1) {
        last = now
        const e = mode === 'in' ? 1 - Math.pow(1 - p, 3) : p * p * (3 - 2 * p)
        // a couple of thin orange lines and small row offsets per frame
        const lineA = (Math.random() * H) | 0, lineB = Math.random() < 0.5 ? (Math.random() * H) | 0 : -1
        const shiftRow0 = (Math.random() * H) | 0, shiftLen = 2 + ((Math.random() * 4) | 0)
        const shift = Math.random() < 0.6 ? ((Math.random() - 0.5) * 8) | 0 : 0
        for (let y = 0; y < H; y++) {
          const row = (y / CELL) | 0
          const sh = (y >= shiftRow0 && y < shiftRow0 + shiftLen) ? shift : 0
          const orange = y === lineA || y === lineB
          for (let x = 0; x < W; x++) {
            const sx = Math.min(W - 1, Math.max(0, x + sh))
            const th = thr[row * cols + ((sx / CELL) | 0)]
            const covered = mode === 'in' ? th < e * 1.02 : th >= e
            const i = (y * W + x) * 4
            if (!covered) { d[i + 3] = 0; continue }
            if (orange) { d[i] = 217; d[i + 1] = 106 + Math.random() * 20; d[i + 2] = 43; d[i + 3] = 200; continue }
            const n = Math.random() * 16 + ((y & 1) ? 0 : 5)   // soft grain + faint scanline
            d[i] = 245 - n; d[i + 1] = 239 - n; d[i + 2] = 228 - n; d[i + 3] = 255
          }
        }
        ctx.putImageData(img, 0, 0)
      }
      if (p < 1) requestAnimationFrame(frame)
      else resolve()
    }
    requestAnimationFrame(frame)
  })
}

// The home page's intro animation should only play when the site is first
// opened on the home page, never when navigating back to it.
onMounted(() => { window.__pfBooted = true })

router.beforeEach(async (to, from) => {
  if (!import.meta.client || busy || to.path === from.path) return
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return
  busy = true
  gOn.value = true
  await nextTick()
  await runGlitch('in', 260)
})

router.afterEach(async () => {
  if (!busy) return
  await nextTick()
  await new Promise(r => setTimeout(r, 80))
  document.documentElement.classList.add('g-shake')
  await runGlitch('out', 520)
  document.documentElement.classList.remove('g-shake')
  gOn.value = false
  busy = false
})
</script>

<template>
  <div class="relative min-h-screen">

    <!-- ── Header ── -->
    <header
      id="hdr"
      class="hdr"
      :class="{ solid: scrolled, open: isMenuOpen }"
    >
      <div class="wrap hdr-in">
        <!-- Logo -->
        <NuxtLink to="/" class="logo" @click="closeMenu">
          <span class="logo-dot" />
          <span class="logo-txt">GLITCHHEAD</span>
        </NuxtLink>

        <!-- Desktop Nav -->
        <nav class="nav" aria-label="Main">
          <NuxtLink to="/#work"     class="nav-link">Work</NuxtLink>
          <NuxtLink to="/#about"    class="nav-link">About</NuxtLink>
          <NuxtLink to="/#services" class="nav-link">Services</NuxtLink>
          <NuxtLink to="/projects"  class="nav-link">All projects</NuxtLink>
        </nav>

        <NuxtLink to="/#contact" class="btn btn-solid hdr-cta">Let's work</NuxtLink>

        <!-- Burger -->
        <button
          class="burger"
          aria-label="Menu"
          :aria-expanded="isMenuOpen"
          @click="toggleMenu"
        >
          <span />
          <span />
        </button>
      </div>

      <!-- Mobile Menu -->
      <div class="menu">
        <nav class="wrap" aria-label="Mobile">
          <NuxtLink to="/"          @click="closeMenu" style="--i:0">Home</NuxtLink>
          <NuxtLink to="/#work"     @click="closeMenu" style="--i:1">Work</NuxtLink>
          <NuxtLink to="/#about"    @click="closeMenu" style="--i:2">About</NuxtLink>
          <NuxtLink to="/#services" @click="closeMenu" style="--i:3">Services</NuxtLink>
          <NuxtLink to="/projects"  @click="closeMenu" style="--i:4">All projects</NuxtLink>
          <NuxtLink to="/#contact"  @click="closeMenu" style="--i:5">Contact</NuxtLink>
        </nav>
        <div class="wrap menu-foot">
          <a href="https://www.linkedin.com/in/sourabh-kahar-6558a7193/" target="_blank" rel="noopener">LinkedIn</a>
          <a href="https://github.com/sourabhkahar" target="_blank" rel="noopener">GitHub</a>
        </div>
      </div>
    </header>

    <!-- ── Page Content ── -->
    <main>
      <NuxtPage />
    </main>

    <!-- ── Footer ── -->
    <footer class="dark">
      <div class="wrap">
        <span>© 2026 Glitch Head. All rights reserved.</span>
        <span>
          <a href="https://www.linkedin.com/in/sourabh-kahar-6558a7193/" target="_blank" rel="noopener">LinkedIn</a>
          &nbsp;/&nbsp;
          <a href="https://github.com/sourabhkahar" target="_blank" rel="noopener">GitHub</a>
        </span>
      </div>
    </footer>

    <!-- ── Soft Glitch Overlay ── -->
    <div class="gx" :class="{ on: gOn }" aria-hidden="true">
      <canvas ref="gCanvas" class="gx-canvas" />
    </div>

  </div>
</template>

<style>
/* Page transitions */
.page-enter-active,
.page-leave-active {
  transition: opacity .3s ease;
}
.page-enter-from,
.page-leave-to {
  opacity: 0;
}
</style>
