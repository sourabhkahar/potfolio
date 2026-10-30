<script setup>
import ProjectCard from "~/components/ProjectCard.vue"
import { projects } from "~/utils/projects"
// ── Data ──────────────────────────────────────────────────────────────────

const steps = [
  { img: 'discover', title: 'Discover', alt: 'Chat bubbles and a light bulb', text: 'We talk through your goals, users and constraints, and agree on what success looks like.' },
  { img: 'design',   title: 'Design',   alt: 'Wireframe being arranged on an artboard', text: 'Wireframes and UI in Figma, so you can see and click through it before any code.' },
  { img: 'build',    title: 'Build',    alt: 'Code editor next to a server stack', text: 'Clean front end and a solid backend, shipped in small pieces you can review as we go.' },
  { img: 'ship',     title: 'Ship',     alt: 'Rocket launching around a globe', text: 'Deployed, monitored and handed over, with support after launch when you need it.' },
]

const featuredProjects = computed(() => projects.filter(p => p.featured))


// ── Rail ─────────────────────────────────────────────────────────────────
const rail    = ref(null)
const prog    = ref(null)
const prevBtn = ref(null)
const nextBtn = ref(null)

const step    = () => rail.value?.firstElementChild?.getBoundingClientRect().width + 24 || 440
const updRail = () => {
  if (!rail.value || !prog.value) return
  const max = rail.value.scrollWidth - rail.value.clientWidth
  const p   = max > 0 ? rail.value.scrollLeft / max : 1
  prog.value.style.setProperty('--p', Math.max(.08, p))
  if (prevBtn.value) prevBtn.value.disabled = rail.value.scrollLeft < 4
  if (nextBtn.value) nextBtn.value.disabled = rail.value.scrollLeft > max - 4
}

// drag-to-scroll
let down = false, sx = 0, sl = 0, moved = false
const onPointerDown = e => {
  if (e.pointerType !== 'mouse' || window.innerWidth <= 720) return
  down = true; moved = false; sx = e.clientX; sl = rail.value.scrollLeft
}
const onPointerMove = e => {
  if (!down) return
  const dx = e.clientX - sx
  if (Math.abs(dx) > 5) { moved = true; rail.value.classList.add('drag') }
  rail.value.scrollLeft = sl - dx
}
const onPointerUp = () => { down = false; rail.value?.classList.remove('drag') }
const onRailClick = e => { if (moved) { e.preventDefault(); e.stopPropagation(); moved = false } }

// ── IntersectionObserver for cover reveal ─────────────────────────────────
let io = null
const watchCovers = () => {
  document.querySelectorAll('.rv:not(.in)').forEach(el =>
    io ? io.observe(el) : el.classList.add('in')
  )
}

// ── Glitch helpers ────────────────────────────────────────────────────────
const GLYPHS = '!<>-_/[]{}=+*^?#01aeoxz'
function scrambleNode (node, dur = 700) {
  const final = node.textContent; let start = null
  const tick = t => {
    if (start === null) start = t
    const p    = Math.min(1, (t - start) / dur)
    const done = Math.floor(p * final.length)
    node.textContent = final.split('').map((ch, i) =>
      i < done || ch === ' ' ? ch : GLYPHS[(Math.random() * GLYPHS.length) | 0]
    ).join('')
    if (p < 1) requestAnimationFrame(tick); else node.textContent = final
  }
  requestAnimationFrame(tick)
}
function scramble (el, dur = 700) {
  if (!el || el.dataset.scr === '1') return
  el.dataset.scr = '1'
  const r = el.getBoundingClientRect()
  if (el.matches('.btn, .nav a')) el.style.minWidth = r.width + 'px'
  else { el.style.minHeight = r.height + 'px'; el.style.maxHeight = r.height + 'px'; el.style.overflow = 'visible' }
  const walker = document.createTreeWalker(el, NodeFilter.SHOW_TEXT); const nodes = []
  while (walker.nextNode()) if (walker.currentNode.textContent.trim()) nodes.push(walker.currentNode)
  nodes.forEach(n => scrambleNode(n, dur))
  setTimeout(() => { el.dataset.scr = ''; el.style.maxHeight = ''; el.style.minHeight = '' }, (dur || 700) + 60)
}
const burst = el => {
  if (!el) return
  el.classList.remove('burst'); void el.offsetWidth; el.classList.add('burst')
  setTimeout(() => el.classList.remove('burst'), 450)
}

const heroGl = ref(null)

onMounted(() => {
  if (!import.meta.client) return

  // Boot animation (first load only, not on every return to home)
  const firstLoad = !window.__pfBooted
  window.__pfBooted = true
  const boot = document.createElement('div')
  boot.className = 'boot'
  boot.setAttribute('aria-hidden', 'true')
  boot.innerHTML = Array.from({ length: 14 }, () =>
    `<i style="--d:${(Math.random() * .25).toFixed(2)}s;--x:${(Math.random() > .5 ? 1 : -1) * (6 + Math.random() * 14)}px"></i>`
  ).join('')
  if (firstLoad) {
    document.body.appendChild(boot)
    setTimeout(() => { boot.classList.add('go'); burst(document.querySelector('.logo-txt')) }, 120)
    setTimeout(() => { boot.remove(); burst(heroGl.value) }, 1100)
  }

  // Hero ready state
  document.querySelector('.hero')?.classList.add('ready')

  // Rail setup
  rail.value?.addEventListener('scroll', updRail, { passive: true })
  window.addEventListener('resize', updRail)
  window.addEventListener('pointermove', onPointerMove)
  window.addEventListener('pointerup', onPointerUp)
  updRail()

  // IntersectionObserver
  io = 'IntersectionObserver' in window
    ? new IntersectionObserver(es => es.forEach(e => {
        if (e.isIntersecting) { e.target.classList.add('in'); io.unobserve(e.target) }
      }), { threshold: .2 })
    : null
  watchCovers()

  // Heading scan + scramble observer
  const headIO = 'IntersectionObserver' in window
    ? new IntersectionObserver(es => es.forEach(e => {
        if (!e.isIntersecting) return
        const h = e.target; headIO.unobserve(h)
        h.classList.remove('scanning'); void h.offsetWidth; h.classList.add('scanning')
        scramble(h, 650)
        h.querySelectorAll('.gl').forEach(g => setTimeout(() => burst(g), 500))
      }), { threshold: .5 })
    : null
  document.querySelectorAll('.scan').forEach(h => headIO?.observe(h))

  // Hover scramble on nav + buttons
  document.querySelectorAll('.nav a, .btn, .visit, .back, .filters button').forEach(el =>
    el.addEventListener('mouseenter', () => scramble(el, 380))
  )

  // Random glitch bursts on hero word
  ;(function loop () {
    setTimeout(() => {
      if (window.scrollY < window.innerHeight) burst(heroGl.value)
      loop()
    }, 3500 + Math.random() * 4000)
  })()

  // Scroll-speed colour split
  let lastY = window.scrollY, split = 0, splitRaf = 0
  const root = document.documentElement
  const decay = () => {
    split *= .86
    root.style.setProperty('--split', split.toFixed(2))
    splitRaf = split > .1 ? requestAnimationFrame(decay) : (root.style.setProperty('--split', 0), 0)
  }
  window.addEventListener('scroll', () => {
    const v = Math.abs(window.scrollY - lastY); lastY = window.scrollY
    split = Math.min(9, Math.max(split, v * .12))
    if (!splitRaf) splitRaf = requestAnimationFrame(decay)
  }, { passive: true })
})

onUnmounted(() => {
  if (!import.meta.client) return
  window.removeEventListener('resize', updRail)
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)
})
</script>

<template>
  <div>
    <!-- ── Hero ─────────────────────────────────────────────── -->
    <section class="hero" id="home">
      <div class="hero-ring" aria-hidden="true" />
      <div class="wrap">
        <!-- Live Status Pill with Avatar -->
        <div class="hero-status-pill fade" style="--d:.1s">
          <div class="hero-avatar-wrap">
            <img src="/images/about-me.png" alt="Sourabh Kahar" class="hero-avatar-img" />
            <span class="hero-online-dot" />
          </div>
          <span class="hero-status-txt">Available for new projects</span>
        </div>

        <h1>
          <span class="line"><span style="--d:.15s">Sourabh Kahar,</span></span>
          <span class="line"><span style="--d:.27s">full-stack</span></span>
          <span class="line"><span style="--d:.39s" class="o">
            <span ref="heroGl" class="gl" data-text="developer.">developer.</span>
          </span></span>
        </h1>
        <div class="hero-foot">
          <div>
            <p class="lead fade" style="--d:.6s">
              I build fast, good-looking web apps from the database to the last pixel:
              dashboards, finance tools, CMS sites and everything between.
            </p>
            <div class="hero-btns fade" style="--d:.72s">
              <a href="#work"    class="btn btn-solid">See my work</a>
              <a href="#contact" class="btn">Let's chat</a>
            </div>
          </div>
          <!-- <div class="stack fade" style="--d:.85s" aria-label="Main stack">
            <span class="chip">Vue</span>
            <span class="chip">Nuxt</span>
            <span class="chip">Go</span>
            <span class="chip">Laravel</span>
            <span class="chip">PostgreSQL</span>
            <span class="chip">Tailwind</span>
            <span class="chip">GSAP</span>
            <span class="chip">Three.js</span>
          </div> -->
        </div>
      </div>
    </section>

    <!-- ── Work Rail ─────────────────────────────────────────── -->
    <section class="sec dark" id="work">
      <div class="wrap sec-head">
        <div>
          <h2 class="h2 scan">Selected work</h2>
          <p class="lead" style="margin-top:16px">Live products I designed, built and shipped.</p>
        </div>
        <div class="rail-ctrl">
          <button
            ref="prevBtn"
            class="rbtn"
            aria-label="Previous projects"
            @click="rail.scrollBy({ left: -step(), behavior: 'smooth' })"
          >
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M15 6l-6 6 6 6"/>
            </svg>
          </button>
          <button
            ref="nextBtn"
            class="rbtn"
            aria-label="Next projects"
            @click="rail.scrollBy({ left: step(), behavior: 'smooth' })"
          >
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M9 6l6 6-6 6"/>
            </svg>
          </button>
        </div>
      </div>

      <!-- Rail -->
      <div
        ref="rail"
        class="rail"
        @pointerdown="onPointerDown"
        @click.capture="onRailClick"
        @dragstart.prevent
      >
        <ProjectCard v-for="p in featuredProjects" :key="p.name" :project="p" />

        <!-- View All card -->
        <NuxtLink to="/projects" class="all-card">
          View all {{ projects.length }} projects →
        </NuxtLink>
      </div>

      <!-- Progress bar -->
      <div class="wrap">
        <div class="progress"><i ref="prog" /></div>
      </div>
    </section>

    <!-- ── Scroll-driven 3D Skills ─────────────────────────── -->
    <client-only>
      <SkillsScroll3D />
      <template #fallback>
        <section id="skills" class="sec dark"><div class="wrap"><h2 class="h2">Core tech stack</h2></div></section>
      </template>
    </client-only>

    <!-- ── About ──────────────────────────────────────────────── -->
    <section class="sec" id="about">
      <div class="wrap about">
        <div>
          <h2 class="h2 scan">I care about what happens after launch.</h2>
          <p class="lead" style="margin-top:20px">
            Clean interfaces are only half the job. The other half is a backend that stays fast,
            secure and easy to change when your business grows.
          </p>
          <p style="margin-top:18px">
            I work across the stack with Vue and Nuxt on the front, Go and Laravel on the back,
            and PostgreSQL or Firebase for data. Clients get one developer who understands the
            whole product, from the first Figma frame to the production server.
          </p>
          <div class="facts">
            <div><b>Frontend</b><span>Vue, Nuxt, Tailwind, Vuetify</span></div>
            <div><b>Backend</b><span>Go, Laravel, Livewire, PHP</span></div>
            <div><b>Data</b><span>PostgreSQL, MySQL, Firebase</span></div>
            <div><b>Motion</b><span>GSAP, Three.js</span></div>
          </div>
        </div>

        <!-- Editorial Portrait Card -->
        <div class="about-card-wrap">
          <div class="about-badge-float about-badge-top">
            <span class="about-badge-dot" />
            <span>Full-Stack Engineer</span>
          </div>
          <div class="about-card-frame">
            <img src="/images/about-me.png" alt="Sourabh Kahar" class="about-portrait" loading="lazy" />
          </div>
          <div class="about-badge-float about-badge-bottom">
            <span>⚡ 8+ Products Shipped</span>
          </div>
        </div>
      </div>
    </section>

    <!-- ── Process ────────────────────────────────────────────── -->
    <section class="sec" id="process" style="padding-top:0">
      <div class="wrap">
        <div class="sec-head">
          <div>
            <h2 class="h2 scan">How a project comes together</h2>
            <p class="lead" style="margin-top:16px">Four clear steps from first call to a live product, with you in the loop the whole way.</p>
          </div>
        </div>
        <ol class="proc">
          <li v-for="(st, i) in steps" :key="st.title">
            <div class="proc-illu"><img :src="`/images/illustrations/${st.img}.svg`" :alt="st.alt" loading="lazy" /></div>
            <span class="proc-n">{{ String(i + 1).padStart(2, '0') }}</span>
            <h3>{{ st.title }}</h3>
            <p>{{ st.text }}</p>
          </li>
        </ol>
      </div>
    </section>

    <!-- ── Services ───────────────────────────────────────────── -->
    <section class="sec" id="services" style="padding-top:0">
      <div class="wrap">
        <div class="sec-head">
          <h2 class="h2 scan">What I can build for you</h2>
        </div>
        <div class="svc">
          <article>
            <div class="svc-illu"><img src="/images/illustrations/fast-apps.svg" alt="Browser loading instantly with a lightning bolt" loading="lazy" /></div>
            <h3>Fast web apps</h3>
            <p>Speed-first builds with optimised rendering and caching, so pages load in under a second.</p>
          </article>
          <article>
            <div class="svc-illu"><img src="/images/illustrations/dashboards.svg" alt="Admin dashboard with charts" loading="lazy" /></div>
            <h3>Dashboards &amp; admin tools</h3>
            <p>Finance apps, HRMS and data tools with solid backends in Go or Laravel.</p>
          </article>
          <article>
            <div class="svc-illu"><img src="/images/illustrations/motion.svg" alt="Animated layered cards" loading="lazy" /></div>
            <h3>Websites with motion</h3>
            <p>Brand sites and landing pages with GSAP and Three.js animation that supports the story.</p>
          </article>
          <article>
            <div class="svc-illu"><img src="/images/illustrations/responsive.svg" alt="Laptop, tablet and phone showing the same site" loading="lazy" /></div>
            <h3>Works on every screen</h3>
            <p>Mobile-first layouts that hold up from small phones to ultra-wide monitors.</p>
          </article>
        </div>
      </div>
    </section>

    <!-- ── Contact ────────────────────────────────────────────── -->
    <section class="sec dark contact" id="contact">
      <div class="wrap">
        <h2 class="scan">
          Have a big idea?
          <span class="o gl" data-text="Let's build it.">Let's build it.</span>
        </h2>

        <!-- Interactive Collaboration Banner -->
        <div class="contact-banner">
          <div>
            <p class="lead" style="color: #E2D9CC; font-size: 1.22rem; max-width: 44ch;">
              I'm open to freelance projects, MVP builds, and long-term engineering collaborations.
              Send a message and I'll get back within a day.
            </p>
            <div class="contact-row" style="margin-top: 32px;">
              <div class="socials">
                <a class="btn btn-solid" href="https://www.linkedin.com/in/sourabh-kahar-6558a7193/" target="_blank" rel="noopener">
                  Message on LinkedIn
                </a>
                <a class="btn" href="https://github.com/sourabhkahar" target="_blank" rel="noopener">
                  GitHub
                </a>
              </div>
            </div>
          </div>

          <div class="contact-illu-wrap">
            <img src="/images/illustrations/cta-build.svg" alt="A message being sent and approved" class="contact-illu-img" loading="lazy" />
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
