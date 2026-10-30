<script setup>
import ProjectCard from "~/components/ProjectCard.vue"
import { projects } from "~/utils/projects"
useHead({
  title: 'All Projects | Sourabh Kahar',
  meta: [
    { name: 'description', content: 'Work across finance, healthcare, media and wellness, built with Vue, Nuxt, Go and Laravel.' }
  ]
})


const groups = ['All', 'Full-stack', 'Frontend', 'CMS']
const activeFilter = ref('All')

const filteredProjects = computed(() => {
  if (activeFilter.value === 'All') return projects
  return projects.filter(p => p.group === activeFilter.value)
})

const getGroupCount = (group) => {
  if (group === 'All') return projects.length
  return projects.filter(p => p.group === group).length
}

// ── Glitch / Text Scramble helper ───────────────────────────────────────────
const GLYPHS = '!<>-_/[]{}=+*^?#01aeoxz'
function scrambleNode(node, dur = 700) {
  const final = node.textContent; let start = null
  const tick = t => {
    if (start === null) start = t
    const p = Math.min(1, (t - start) / dur)
    const done = Math.floor(p * final.length)
    node.textContent = final.split('').map((ch, i) =>
      i < done || ch === ' ' ? ch : GLYPHS[(Math.random() * GLYPHS.length) | 0]
    ).join('')
    if (p < 1) requestAnimationFrame(tick); else node.textContent = final
  }
  requestAnimationFrame(tick)
}

function scramble(el, dur = 380) {
  if (!el || el.dataset.scr === '1') return
  el.dataset.scr = '1'
  const walker = document.createTreeWalker(el, NodeFilter.SHOW_TEXT); const nodes = []
  while (walker.nextNode()) if (walker.currentNode.textContent.trim()) nodes.push(walker.currentNode)
  nodes.forEach(n => scrambleNode(n, dur))
  setTimeout(() => { el.dataset.scr = '' }, (dur || 380) + 60)
}

const burst = el => {
  if (!el) return
  el.classList.remove('burst'); void el.offsetWidth; el.classList.add('burst')
  setTimeout(() => el.classList.remove('burst'), 450)
}

const filterChange = (g) => {
  activeFilter.value = g
  nextTick(() => {
    document.querySelectorAll('.proj-grid .card').forEach(c => {
      c.classList.remove('pop')
      void c.offsetWidth
      c.classList.add('pop', 'in')
    })
  })
}

onMounted(() => {
  if (!import.meta.client) return

  const pvHero = document.querySelector('.pv-hero')
  if (pvHero) {
    pvHero.classList.add('ready')
  }

  // Cover reveals
  const io = 'IntersectionObserver' in window
    ? new IntersectionObserver(es => es.forEach(e => {
        if (e.isIntersecting) { e.target.classList.add('in'); io.unobserve(e.target) }
      }), { threshold: 0.15 })
    : null

  document.querySelectorAll('.rv').forEach(el => io ? io.observe(el) : el.classList.add('in'))

  // Scan & Glitch for headings
  const headIO = 'IntersectionObserver' in window
    ? new IntersectionObserver(es => es.forEach(e => {
        if (!e.isIntersecting) return
        const h = e.target; headIO.unobserve(h)
        h.classList.remove('scanning'); void h.offsetWidth; h.classList.add('scanning')
        scramble(h, 650)
        h.querySelectorAll('.gl').forEach(g => setTimeout(() => burst(g), 500))
      }), { threshold: 0.5 })
    : null
  document.querySelectorAll('.scan').forEach(h => headIO?.observe(h))

  // Hover scramble on buttons and links
  document.querySelectorAll('.back, .filters button, .btn').forEach(el => {
    el.addEventListener('mouseenter', () => scramble(el, 380))
  })
})
</script>

<template>
  <div class="view" id="v-projects">
    <!-- Projects Hero -->
    <section class="pv-hero" id="projects">
      <div class="wrap">
        <NuxtLink to="/" class="back">← Back to home</NuxtLink>
        <h1>
          <span class="line"><span>All projects</span></span>
        </h1>
        <p class="lead" style="margin-top:20px">
          Work across finance, healthcare, media and wellness, built with Vue, Nuxt, Go and Laravel.
        </p>

        <!-- Filters -->
        <div class="filters" id="filters">
          <button
            v-for="g in groups"
            :key="g"
            :aria-pressed="activeFilter === g"
            :data-f="g"
            @click="filterChange(g)"
          >
            {{ g }}<sup>{{ getGroupCount(g) }}</sup>
          </button>
        </div>

        <!-- Grid with Real UI Screenshots -->
        <div class="proj-grid" id="grid">
          <ProjectCard
            v-for="p in filteredProjects"
            :key="p.name"
            :project="p"
            class="in pop"
            :data-group="p.group"
          />
        </div>
      </div>
    </section>

    <!-- Contact CTA Section on Projects Page -->
    <section class="dark contact sec">
      <div class="wrap">
        <h2 class="scan">
          Like what you see?
          <span class="o gl" data-text="Let's talk.">Let's talk.</span>
        </h2>

        <!-- Interactive Collaboration Banner -->
        <div class="contact-banner">
          <div>
            <p class="lead" style="color: #E2D9CC; font-size: 1.22rem; max-width: 44ch;">
              Whether you need a new web application from scratch or want to scale an existing platform, let's talk.
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
            <img src="/images/illustrations/cta-talk.svg" alt="Calendar with a booked day and a call in progress" class="contact-illu-img" loading="lazy" />
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
