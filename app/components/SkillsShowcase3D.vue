<script setup>
import * as THREE from 'three'

const canvasRef = ref(null)
const containerRef = ref(null)

const skills = [
  // ── Frontend Architecture & UI ──
  {
    id: 'javascript',
    name: 'JavaScript (ES6+)',
    short: 'JS',
    category: 'Core Language',
    categoryGroup: 'Frontend',
    color: '#F7DF1E',
    experience: '5+ Years',
    description: 'Deep mastery of asynchronous patterns, Event Loop, Closures, modern ESNext syntax, browser APIs, and clean modular architecture.',
    projects: ['All Projects', 'Interactive WebGL', 'Custom SPAs']
  },
  {
    id: 'vue',
    name: 'Vue.js 3',
    short: 'Vue',
    category: 'Frontend Framework',
    categoryGroup: 'Frontend',
    color: '#42b883',
    experience: '4+ Years',
    description: 'Reactive component systems, Pinia state management, Composition API, custom directives, and sub-millisecond virtual DOM rendering.',
    projects: ['Golfo Radio', 'Hair Creation', 'Capmagic', 'Ashray Infrastructure']
  },
  {
    id: 'nuxt',
    name: 'Nuxt 4',
    short: 'Nuxt',
    category: 'Full-Stack Framework',
    categoryGroup: 'Frontend',
    color: '#00DC82',
    experience: '3+ Years',
    description: 'Universal SSR, static site generation, Nitro server engine, automatic route code-splitting, SEO metadata, and edge caching.',
    projects: ['QRGEN', 'Chit Fund Manager', 'This Portfolio']
  },
  {
    id: 'tailwind',
    name: 'Tailwind CSS',
    short: 'Tailwind',
    category: 'UI & Design Systems',
    categoryGroup: 'Frontend',
    color: '#38BDF8',
    experience: '4+ Years',
    description: 'Scalable utility-first design systems, fluid responsive typography, dark/light theme tokens, and custom micro-animations.',
    projects: ['Golfo Radio', 'Hair Creation', 'QRGEN', 'Chit Fund Manager']
  },
  {
    id: 'bootstrap',
    name: 'Bootstrap',
    short: 'Bootstrap',
    category: 'Responsive Framework',
    categoryGroup: 'Frontend',
    color: '#7952B3',
    experience: '4+ Years',
    description: 'Rapid responsive UI prototyping, 12-column flexible grid architectures, cross-browser compatibility, and modular layout components.',
    projects: ['XS Pilates', 'Admin Portals', 'Enterprise Dashboards']
  },

  // ── Backend & Server Engineering ──
  {
    id: 'php',
    name: 'PHP',
    short: 'PHP',
    category: 'Backend Engineering',
    categoryGroup: 'Backend',
    color: '#777BB4',
    experience: '5+ Years',
    description: 'Object-oriented programming, modern PHP 8.x attributes, robust MVC architectures, RESTful API design, and MySQL/Postgres drivers.',
    projects: ['XS Pilates', 'Capmagic', 'Sarvesukhantu Swaha']
  },
  {
    id: 'laravel',
    name: 'Laravel 11',
    short: 'Laravel',
    category: 'Backend Framework',
    categoryGroup: 'Backend',
    color: '#FF2D20',
    experience: '4+ Years',
    description: 'Enterprise backend development, Eloquent ORM relations, Livewire dynamic interfaces, queued jobs, authentication, and secure REST APIs.',
    projects: ['Capmagic', 'Sarvesukhantu Swaha']
  },
  {
    id: 'node',
    name: 'Node.js',
    short: 'Node',
    category: 'Runtime & APIs',
    categoryGroup: 'Backend',
    color: '#68A063',
    experience: '3+ Years',
    description: 'Event-driven backend services, Express and Nitro APIs, asynchronous I/O, middleware pipelines, and npm package orchestration.',
    projects: ['Full-stack Microservices', 'Serverless APIs']
  },
  {
    id: 'go',
    name: 'Go (Golang)',
    short: 'Go',
    category: 'High-Concurrency Systems',
    categoryGroup: 'Backend',
    color: '#00ACD7',
    experience: '2+ Years',
    description: 'High-throughput microservices, concurrent goroutine channels, high-performance REST APIs with Gin/Fiber, and minimal CPU footprint.',
    projects: ['Ashray Infrastructure']
  },

  // ── Database & Cloud ──
  {
    id: 'postgres',
    name: 'PostgreSQL',
    short: 'Postgres',
    category: 'Relational Database',
    categoryGroup: 'Database',
    color: '#4169E1',
    experience: '4+ Years',
    description: 'Normalized database schema design, index tuning, complex analytical aggregations, ACID transactions, and hybrid JSONB document stores.',
    projects: ['Capmagic', 'Ashray Infrastructure']
  },
  {
    id: 'firebase',
    name: 'Firebase',
    short: 'Firebase',
    category: 'Serverless Cloud',
    categoryGroup: 'Database',
    color: '#FFCA28',
    experience: '3+ Years',
    description: 'Production Firebase Hosting, Firestore real-time NoSQL database streams, Firebase Auth rules, and scalable cloud storage.',
    projects: ['Hair Creation', 'Chit Fund Manager', 'This Portfolio']
  },

  // ── DevOps & Version Control ──
  {
    id: 'github',
    name: 'GitHub',
    short: 'GitHub',
    category: 'DevOps & CI/CD',
    categoryGroup: 'DevOps',
    color: '#E6EDF3',
    experience: '5+ Years',
    description: 'Git source control, GitHub Actions CI/CD automated deployment pipelines, code review workflows, releases, and issue tracking.',
    projects: ['Open Source Repos', 'All Production Workflows']
  },
  {
    id: 'gitlab',
    name: 'GitLab',
    short: 'GitLab',
    category: 'DevOps & Pipelines',
    categoryGroup: 'DevOps',
    color: '#FC6D26',
    experience: '3+ Years',
    description: 'GitLab CI/CD automated test and deployment pipelines, container registry integration, branch protection rules, and enterprise collaboration.',
    projects: ['Enterprise CI/CD Pipelines']
  },

  // ── Creative Motion & 3D ──
  {
    id: 'gsap',
    name: 'GSAP',
    short: 'GSAP',
    category: 'Motion & Physics',
    categoryGroup: 'Creative',
    color: '#88CE02',
    experience: '3+ Years',
    description: 'Silky smooth ScrollTrigger timelines, kinetic typography, coordinate tweening, SVG morphing, and hardware-accelerated motion choreography.',
    projects: ['Ashray Infrastructure', 'This Portfolio']
  },
  {
    id: 'threejs',
    name: 'Three.js / WebGL',
    short: 'Three.js',
    category: 'Creative 3D & Shaders',
    categoryGroup: 'Creative',
    color: '#C2571A',
    experience: '2+ Years',
    description: 'Interactive 3D WebGL perspective viewports, procedural particle systems, dynamic lighting, camera choreography, and custom shaders.',
    projects: ['Interactive Skills Showcase', 'Portfolio 3D Viewport']
  }
]

// Category tabs summary
const categories = [
  { name: 'All', count: skills.length },
  { name: 'Frontend', count: skills.filter(s => s.categoryGroup === 'Frontend').length },
  { name: 'Backend', count: skills.filter(s => s.categoryGroup === 'Backend').length },
  { name: 'Database', count: skills.filter(s => s.categoryGroup === 'Database').length },
  { name: 'DevOps', count: skills.filter(s => s.categoryGroup === 'DevOps').length },
  { name: 'Creative', count: skills.filter(s => s.categoryGroup === 'Creative').length }
]

const activeIndex = ref(0)
const activeSkill = computed(() => skills[activeIndex.value])
const currentCategory = computed(() => activeSkill.value?.categoryGroup || 'Frontend')

// Filter jump by category
const filterByCategory = (catName) => {
  if (catName === 'All') {
    selectSkill(0)
    return
  }
  const firstMatchIndex = skills.findIndex(s => s.categoryGroup === catName)
  if (firstMatchIndex !== -1) {
    selectSkill(firstMatchIndex)
  }
}

// Interactive programmatic jump
const selectSkill = (idx) => {
  activeIndex.value = idx
}

const nextSkill = () => {
  activeIndex.value = (activeIndex.value + 1) % skills.length
}

const prevSkill = () => {
  activeIndex.value = (activeIndex.value - 1 + skills.length) % skills.length
}

onMounted(() => {
  if (!import.meta.client) return

  const canvas = canvasRef.value
  const container = containerRef.value
  if (!canvas || !container) return

  const width = container.clientWidth || 900
  const height = container.clientHeight || 420

  // ── Scene Setup ──
  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(40, width / height, 0.1, 1000)
  camera.position.set(0, 0, 7.8)

  const renderer = new THREE.WebGLRenderer({
    canvas,
    alpha: true,
    antialias: true,
    powerPreference: 'high-performance'
  })
  renderer.setSize(width, height)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

  // ── Lighting ──
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.8)
  scene.add(ambientLight)

  const pointLight = new THREE.PointLight(0xC2571A, 2.5, 30)
  pointLight.position.set(0, 4, 6)
  scene.add(pointLight)

  // ── Canvas Textures for 3D Cards ──
  function createCardTexture(item) {
    const c = document.createElement('canvas')
    c.width = 600
    c.height = 760
    const ctx = c.getContext('2d')

    // Card background
    ctx.fillStyle = '#1A1815'
    ctx.strokeStyle = 'rgba(245, 239, 228, 0.18)'
    ctx.lineWidth = 4

    const r = 40
    const x = 12, y = 12, w = 576, h = 736
    ctx.beginPath()
    ctx.moveTo(x + r, y)
    ctx.lineTo(x + w - r, y)
    ctx.quadraticCurveTo(x + w, y, x + w, y + r)
    ctx.lineTo(x + w, y + h - r)
    ctx.quadraticCurveTo(x + w, y + h, x + w - r, y + h)
    ctx.lineTo(x + r, y + h)
    ctx.quadraticCurveTo(x, y + h, x, y + h - r)
    ctx.lineTo(x, y + r)
    ctx.quadraticCurveTo(x, y, x + r, y)
    ctx.closePath()
    ctx.fill()
    ctx.stroke()

    // Top Accent line in skill color
    ctx.fillStyle = item.color
    ctx.beginPath()
    ctx.roundRect(x + 28, y + 28, 80, 8, 4)
    ctx.fill()

    // Glowing Circle indicator
    ctx.fillStyle = item.color
    ctx.beginPath()
    ctx.arc(x + 70, y + 150, 36, 0, Math.PI * 2)
    ctx.fill()

    ctx.fillStyle = '#1A1815'
    ctx.font = '900 28px Syne, Arial, sans-serif'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    const badgeText = item.short.length <= 3 ? item.short : item.short.slice(0, 2)
    ctx.fillText(badgeText.toUpperCase(), x + 70, y + 152)

    // Category
    ctx.fillStyle = '#A3998C'
    ctx.font = '700 24px Manrope, sans-serif'
    ctx.textAlign = 'left'
    ctx.fillText(item.category.toUpperCase(), x + 40, y + 250)

    // Skill Name
    ctx.fillStyle = '#F5EFE4'
    ctx.font = '800 50px Syne, Arial, sans-serif'
    ctx.fillText(item.name, x + 40, y + 330)

    // Experience badge
    ctx.fillStyle = 'rgba(245, 239, 228, 0.08)'
    ctx.beginPath()
    ctx.roundRect(x + 40, y + 390, 230, 52, 26)
    ctx.fill()

    ctx.fillStyle = item.color
    ctx.font = '700 24px Manrope, sans-serif'
    ctx.fillText(`⚡ ${item.experience}`, x + 64, y + 424)

    // Divider line
    ctx.strokeStyle = 'rgba(245, 239, 228, 0.1)'
    ctx.lineWidth = 2
    ctx.beginPath()
    ctx.moveTo(x + 40, y + 490)
    ctx.lineTo(x + w - 40, y + 490)
    ctx.stroke()

    // Short summary
    ctx.fillStyle = '#D6CCBE'
    ctx.font = '500 25px Manrope, sans-serif'
    
    // Simple line wrap for summary
    const words = item.description.split(' ')
    let line = ''
    let curY = y + 545
    for (let n = 0; n < words.length && curY < y + 700; n++) {
      const testLine = line + words[n] + ' '
      const metrics = ctx.measureText(testLine)
      if (metrics.width > w - 80 && n > 0) {
        ctx.fillText(line, x + 40, curY)
        line = words[n] + ' '
        curY += 38
      } else {
        line = testLine
      }
    }
    if (curY < y + 700) {
      ctx.fillText(line, x + 40, curY)
    }

    const tex = new THREE.CanvasTexture(c)
    tex.minFilter = THREE.LinearFilter
    return tex
  }

  // ── Create 3D Cards along a Curved Arc Stage ──
  const cardGroup = new THREE.Group()
  scene.add(cardGroup)

  const cardWidth = 2.4
  const cardHeight = 3.04
  const cardGeometry = new THREE.PlaneGeometry(cardWidth, cardHeight)
  const cardMeshes = []

  skills.forEach((skill) => {
    const mat = new THREE.MeshStandardMaterial({
      map: createCardTexture(skill),
      transparent: true,
      side: THREE.DoubleSide,
      roughness: 0.2,
      metalness: 0.1
    })

    const mesh = new THREE.Mesh(cardGeometry, mat)
    cardGroup.add(mesh)
    cardMeshes.push(mesh)
  })

  // ── Drag & Scroll Kinetic Physics ──
  let targetOffset = 0
  let currentOffset = 0
  let isPointerDown = false
  let startX = 0
  let lastX = 0
  let pointerVelocity = 0

  const onPointerDown = (e) => {
    isPointerDown = true
    startX = e.clientX || (e.touches && e.touches[0].clientX)
    lastX = startX
    pointerVelocity = 0
  }

  const onPointerMove = (e) => {
    if (!isPointerDown) return
    const clientX = e.clientX || (e.touches && e.touches[0].clientX)
    const delta = clientX - lastX
    pointerVelocity = delta
    targetOffset -= delta * 0.006
    lastX = clientX
  }

  const onPointerUp = () => {
    if (!isPointerDown) return
    isPointerDown = false
    // Snap to nearest index
    targetOffset += -pointerVelocity * 0.02
    targetOffset = Math.round(targetOffset)
    if (targetOffset < 0) targetOffset = 0
    if (targetOffset > skills.length - 1) targetOffset = skills.length - 1
  }

  // Scroll link
  let lastScrollY = window.scrollY
  const onWindowScroll = () => {
    const rect = container.getBoundingClientRect()
    // When section is in view
    if (rect.top < window.innerHeight && rect.bottom > 0) {
      const scrollDelta = window.scrollY - lastScrollY
      targetOffset += scrollDelta * 0.003
      // Clamp bounds
      if (targetOffset < 0) targetOffset = 0
      if (targetOffset > skills.length - 1) targetOffset = skills.length - 1
    }
    lastScrollY = window.scrollY
  }

  window.addEventListener('scroll', onWindowScroll, { passive: true })
  container.addEventListener('mousedown', onPointerDown)
  window.addEventListener('mousemove', onPointerMove)
  window.addEventListener('mouseup', onPointerUp)
  container.addEventListener('touchstart', onPointerDown, { passive: true })
  window.addEventListener('touchmove', onPointerMove, { passive: true })
  window.addEventListener('touchend', onPointerUp)

  // Watch programmatic changes from buttons/dots
  watch(activeIndex, (newVal) => {
    targetOffset = newVal
  })

  // ── Animation Loop ──
  const clock = new THREE.Clock()
  let animId = null

  const animate = () => {
    const elapsed = clock.getElapsedTime()

    // Smooth inertia interpolation
    currentOffset += (targetOffset - currentOffset) * 0.08

    // Update active index in Vue state
    const nearestIndex = Math.max(0, Math.min(skills.length - 1, Math.round(currentOffset)))
    if (activeIndex.value !== nearestIndex) {
      activeIndex.value = nearestIndex
    }

    // Position cards on a 3D perspective arc
    const radius = 6.2
    const spacing = 0.52

    cardMeshes.forEach((mesh, i) => {
      const relPos = i - currentOffset
      const angle = relPos * spacing

      // Circular arc coordinates
      mesh.position.x = radius * Math.sin(angle)
      mesh.position.z = radius * (1 - Math.cos(angle)) * -1.2 + Math.abs(relPos) * -0.6
      mesh.position.y = Math.sin(elapsed * 1.8 + i) * 0.06 - Math.abs(relPos) * 0.12

      // Y-axis rotation to face focus point
      mesh.rotation.y = angle * 0.85
      // Dynamic tilt based on distance
      mesh.rotation.z = -relPos * 0.04

      // Scale & opacity based on proximity to center
      const dist = Math.abs(relPos)
      const scale = Math.max(0.68, 1 - dist * 0.16)
      mesh.scale.set(scale, scale, scale)

      // Dim non-focused cards
      if (mesh.material) {
        mesh.material.opacity = Math.max(0.2, 1 - dist * 0.35)
      }
    })

    renderer.render(scene, camera)
    animId = requestAnimationFrame(animate)
  }

  animate()

  // Resize handler
  const onResize = () => {
    if (!container || !canvas) return
    const w = container.clientWidth || 900
    const h = container.clientHeight || 420
    camera.aspect = w / h
    camera.updateProjectionMatrix()
    renderer.setSize(w, h)
  }
  window.addEventListener('resize', onResize)

  // Cleanup
  onUnmounted(() => {
    cancelAnimationFrame(animId)
    window.removeEventListener('scroll', onWindowScroll)
    container.removeEventListener('mousedown', onPointerDown)
    window.removeEventListener('mousemove', onPointerMove)
    window.removeEventListener('mouseup', onPointerUp)
    container.removeEventListener('touchstart', onPointerDown)
    window.removeEventListener('touchmove', onPointerMove)
    window.removeEventListener('touchend', onPointerUp)
    window.removeEventListener('resize', onResize)
    renderer.dispose()
  })
})
</script>

<template>
  <div class="skills-showcase-container">
    <!-- Category Filter Tabs -->
    <div class="category-tabs-bar">
      <button
        v-for="cat in categories"
        :key="cat.name"
        class="cat-tab-btn"
        :class="{ active: (cat.name === 'All' && false) || currentCategory === cat.name }"
        @click="filterByCategory(cat.name)"
      >
        <span>{{ cat.name }}</span>
        <span class="cat-count">{{ cat.count }}</span>
      </button>
    </div>

    <!-- Top HUD Navigation / Individual skill pills -->
    <div class="skills-pills-bar">
      <button
        v-for="(s, idx) in skills"
        :key="s.id"
        class="skill-tab-pill"
        :class="{ active: activeIndex === idx }"
        @click="selectSkill(idx)"
      >
        <span class="pill-dot" :style="{ backgroundColor: s.color }" />
        <span>{{ s.short }}</span>
        <span class="pill-cat-mini">{{ s.categoryGroup }}</span>
      </button>
    </div>

    <!-- 3D Perspective Stage -->
    <div
      ref="containerRef"
      class="skills-3d-stage"
      title="Drag horizontally to scrub through skills in 3D"
    >
      <canvas ref="canvasRef" class="skills-3d-canvas" />

      <!-- Stage Controls Overlay -->
      <div class="stage-controls">
        <button
          class="stage-arrow-btn"
          aria-label="Previous skill"
          :disabled="activeIndex === 0"
          @click="prevSkill"
        >
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
            <path d="M15 18l-6-6 6-6" />
          </svg>
        </button>
        <div class="stage-hint-pill">
          <span>{{ activeIndex + 1 }} / {{ skills.length }}</span>
          <span class="hint-sep">·</span>
          <span>Scroll or drag in 3D</span>
        </div>
        <button
          class="stage-arrow-btn"
          aria-label="Next skill"
          :disabled="activeIndex === skills.length - 1"
          @click="nextSkill"
        >
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
            <path d="M9 18l6-6-6-6" />
          </svg>
        </button>
      </div>
    </div>

    <!-- Active Skill Deep-Dive Card (HUD) -->
    <div v-if="activeSkill" class="skill-detail-card">
      <div class="detail-header">
        <div class="detail-badge">
          <span class="detail-dot" :style="{ backgroundColor: activeSkill.color }" />
          <span class="detail-cat">{{ activeSkill.category }}</span>
          <span class="detail-group-pill">{{ activeSkill.categoryGroup }}</span>
        </div>
        <span class="detail-exp">{{ activeSkill.experience }} Real-world Dev</span>
      </div>

      <div class="detail-body">
        <h3 class="detail-title">{{ activeSkill.name }}</h3>
        <p class="detail-desc">{{ activeSkill.description }}</p>
      </div>

      <div class="detail-footer">
        <span class="footer-label">Shipped with in:</span>
        <div class="detail-tags">
          <span v-for="proj in activeSkill.projects" :key="proj" class="proj-tag">
            {{ proj }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.skills-showcase-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 100%;
}

/* Category Filter Tabs */
.category-tabs-bar {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  padding-bottom: 4px;
}

.cat-tab-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 6px 14px;
  border-radius: 999px;
  background: transparent;
  border: 1.5px solid var(--line);
  font-family: var(--display);
  font-size: 0.82rem;
  font-weight: 700;
  color: var(--mute);
  cursor: pointer;
  transition: all 0.25s ease;
}

.cat-tab-btn:hover {
  background: var(--ivory-2);
  color: var(--ink);
  border-color: var(--ink);
}

.cat-tab-btn.active {
  background: var(--ink);
  color: var(--ivory);
  border-color: var(--ink);
}

.cat-count {
  font-size: 0.72rem;
  opacity: 0.7;
}

/* Individual Skill Pills Row */
.skills-pills-bar {
  display: flex;
  gap: 8px;
  overflow-x: auto;
  padding: 4px 4px 10px;
  scrollbar-width: none;
  -webkit-overflow-scrolling: touch;
}
.skills-pills-bar::-webkit-scrollbar {
  display: none;
}

.skill-tab-pill {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 7px 16px;
  border-radius: 999px;
  background: var(--ivory-2);
  border: 1.5px solid var(--line);
  font-family: var(--body);
  font-size: 0.86rem;
  font-weight: 700;
  color: var(--ink);
  white-space: nowrap;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  cursor: pointer;
}

.skill-tab-pill:hover {
  background: rgba(30, 27, 24, 0.08);
  transform: translateY(-2px);
}

.skill-tab-pill.active {
  background: var(--ink);
  color: var(--ivory);
  border-color: var(--ink);
  box-shadow: 0 4px 16px rgba(30, 27, 24, 0.2);
}

.pill-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  flex-shrink: 0;
}

.pill-cat-mini {
  font-size: 0.68rem;
  opacity: 0.55;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  margin-left: 2px;
}

.skill-tab-pill.active .pill-cat-mini {
  opacity: 0.8;
  color: var(--orange-hi);
}

/* 3D Perspective Stage */
.skills-3d-stage {
  position: relative;
  width: 100%;
  height: 440px;
  background: radial-gradient(circle at center, rgba(194, 87, 26, 0.08) 0%, transparent 68%), #181512;
  border: 1px solid rgba(245, 239, 228, 0.12);
  border-radius: 28px;
  overflow: hidden;
  cursor: grab;
  user-select: none;
  touch-action: pan-y;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.25);
}

.skills-3d-stage:active {
  cursor: grabbing;
}

.skills-3d-canvas {
  width: 100% !important;
  height: 100% !important;
  display: block;
}

/* Stage Overlay Controls */
.stage-controls {
  position: absolute;
  bottom: 20px;
  left: 0;
  right: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 16px;
  pointer-events: none;
  z-index: 10;
}

.stage-arrow-btn {
  pointer-events: auto;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: rgba(30, 27, 24, 0.85);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  border: 1.5px solid rgba(245, 239, 228, 0.2);
  color: #F5EFE4;
  display: grid;
  place-items: center;
  transition: all 0.25s ease;
  cursor: pointer;
}

.stage-arrow-btn svg {
  width: 18px;
  height: 18px;
}

.stage-arrow-btn:hover:not(:disabled) {
  background: var(--orange);
  border-color: var(--orange);
  transform: scale(1.08);
}

.stage-arrow-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.stage-hint-pill {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 6px 16px;
  border-radius: 999px;
  background: rgba(30, 27, 24, 0.85);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  border: 1px solid rgba(245, 239, 228, 0.15);
  color: #E2D9CC;
  font-size: 0.82rem;
  font-weight: 700;
  letter-spacing: -0.01em;
}

.hint-sep {
  opacity: 0.4;
}

/* Detail Card HUD */
.skill-detail-card {
  background: var(--ivory-2);
  border: 1.5px solid var(--line);
  border-radius: 24px;
  padding: clamp(24px, 3.5vw, 36px);
  display: flex;
  flex-direction: column;
  gap: 16px;
  transition: all 0.35s ease;
  box-shadow: 0 10px 30px rgba(30, 27, 24, 0.05);
}

.detail-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
}

.detail-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.84rem;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--mute);
}

.detail-dot {
  width: 9px;
  height: 9px;
  border-radius: 50%;
}

.detail-group-pill {
  font-size: 0.74rem;
  font-weight: 700;
  padding: 2px 10px;
  border-radius: 999px;
  background: var(--ink);
  color: var(--ivory);
  letter-spacing: 0.02em;
}

.detail-exp {
  font-size: 0.86rem;
  font-weight: 700;
  color: var(--orange);
  background: rgba(194, 87, 26, 0.1);
  padding: 4px 12px;
  border-radius: 999px;
}

.detail-title {
  font-family: var(--display);
  font-size: clamp(1.8rem, 3.5vw, 2.5rem);
  font-weight: 800;
  margin: 0;
  line-height: 1.1;
  color: var(--ink);
}

.detail-desc {
  font-size: 1.05rem;
  color: #4D453D;
  margin: 8px 0 0;
  line-height: 1.6;
  max-width: 80ch;
}

.detail-footer {
  display: flex;
  align-items: center;
  gap: 14px;
  flex-wrap: wrap;
  margin-top: 8px;
  padding-top: 16px;
  border-top: 1px solid var(--line);
}

.footer-label {
  font-size: 0.86rem;
  font-weight: 700;
  color: var(--mute);
}

.detail-tags {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.proj-tag {
  font-size: 0.82rem;
  font-weight: 700;
  padding: 4px 12px;
  border-radius: 999px;
  background: rgba(30, 27, 24, 0.06);
  border: 1px solid var(--line);
  color: var(--ink);
}

@media (max-width: 768px) {
  .skills-3d-stage {
    height: 360px;
  }
}
</style>
