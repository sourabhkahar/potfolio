<script setup>
import * as THREE from 'three'

// ── Skill data ─────────────────────────────────────────────────────────────
// Logos from devicon (MIT), bundled locally in /public/images/skills
const ICON = '/images/skills'
// Grouped by category; the order here is the scroll order.
// `experience` and `projects` are optional — leave them out to hide those chips.
const skills = [
  // ── Frontend
  { id: 'javascript', group: 'Frontend', name: 'JavaScript', short: 'JS', category: 'Core Language', color: '#F7DF1E', icon: null, shape: 'js',
    description: 'Modern ES features, async patterns and clean, framework-free DOM work underneath everything I ship.',
    projects: ['All live projects'] },
  { id: 'vue', group: 'Frontend', name: 'Vue.js 3', short: 'Vue', category: 'Frontend Framework', color: '#42b883', experience: '4+ Years', icon: `${ICON}/vue.svg`,
    description: 'Reactive component systems, Pinia state management and the Composition API for fast, high-frequency dashboards.',
    projects: ['Golfo Radio', 'Hair Creation', 'Capmagic'] },
  { id: 'nuxt', group: 'Frontend', name: 'Nuxt 4', short: 'Nuxt', category: 'Full-Stack Framework', color: '#00DC82', experience: '3+ Years', icon: `${ICON}/nuxt.svg`,
    description: 'Universal SSR, static pre-rendering, Nitro server routes and top Lighthouse scores out of the box.',
    projects: ['QRGEN', 'Chit Fund Manager', 'This Portfolio'] },
  { id: 'tailwind', group: 'Frontend', name: 'Tailwind CSS', short: 'Tailwind', category: 'Styling & Design Systems', color: '#38BDF8', experience: '4+ Years', icon: `${ICON}/tailwind.svg`,
    description: 'Custom design systems, fluid type scales, responsive tokens and bespoke animation utilities.',
    projects: ['All live projects'] },
  { id: 'bootstrap', group: 'Frontend', name: 'Bootstrap', short: 'Bootstrap', category: 'Styling & UI Kits', color: '#8F5BF0', icon: `${ICON}/bootstrap.svg`,
    description: 'Responsive grids and ready-made UI components for quick, consistent layouts and admin panels.' },
  { id: 'gsap', group: 'Frontend', name: 'GSAP', short: 'GSAP', category: 'Motion & Animation', color: '#88CE02', experience: '3+ Years', icon: null,
    description: 'ScrollTrigger timelines, kinetic typography and smooth, hardware-accelerated motion choreography.',
    projects: ['Ashray Infrastructure', 'This Portfolio'] },
  { id: 'threejs', group: 'Frontend', name: 'Three.js / WebGL', short: 'Three.js', category: 'Creative 3D', color: '#D96A2B', experience: '2+ Years', icon: `${ICON}/threejs.svg`,
    description: 'Custom WebGL scenes, particle systems, shaders and interactive 3D like the one you are scrolling through.',
    projects: ['This Portfolio'] },
  // ── Backend
  { id: 'node', group: 'Backend', name: 'Node.js', short: 'Node', category: 'Server Runtime', color: '#6CC24A', icon: `${ICON}/nodejs.svg`,
    description: 'JavaScript on the server: APIs, build tooling, scripts and real-time services.' },
  { id: 'go', group: 'Backend', name: 'Go', short: 'Go', category: 'Backend & Systems', color: '#00ACD7', experience: '2+ Years', icon: `${ICON}/go.svg`,
    description: 'High-throughput services, goroutine concurrency, clean REST APIs and low-latency data processing.',
    projects: ['Ashray Infrastructure'] },
  { id: 'php', group: 'Backend', name: 'PHP', short: 'PHP', category: 'Server Language', color: '#8892BF', icon: `${ICON}/php.svg`,
    description: 'Server-rendered sites, custom CMS builds and integrations, from plain PHP to modern frameworks.',
    projects: ['XS Pilates', 'Sarvesukhantu Swaha'] },
  { id: 'laravel', group: 'Backend', name: 'Laravel', short: 'Laravel', category: 'Backend Framework', color: '#FF2D20', experience: '4+ Years', icon: `${ICON}/laravel.svg`,
    description: 'Eloquent ORM, Livewire interfaces, secure auth, queued workers and well-structured RESTful APIs.',
    projects: ['Capmagic', 'Sarvesukhantu Swaha'] },
  // ── Data & Cloud
  { id: 'postgres', group: 'Data & Cloud', name: 'PostgreSQL', short: 'Postgres', category: 'Database', color: '#5B8CFF', experience: '4+ Years', icon: `${ICON}/postgres.svg`,
    description: 'Relational modelling, index tuning, analytical SQL, ACID transactions and JSONB document storage.',
    projects: ['Capmagic', 'Ashray Infrastructure'] },
  { id: 'mysql', group: 'Data & Cloud', name: 'MySQL', short: 'MySQL', category: 'Database', color: '#4AA8D8', icon: `${ICON}/mysql.svg`,
    description: 'Schema design, joins and indexing for the CMS and Laravel apps that run on classic LAMP stacks.',
    projects: ['Sarvesukhantu Swaha'] },
  { id: 'firebase', group: 'Data & Cloud', name: 'Firebase', short: 'Firebase', category: 'Serverless Cloud', color: '#FFCA28', experience: '3+ Years', icon: `${ICON}/firebase.svg`,
    description: 'Hosting, real-time Firestore streams, auth rules and cloud functions for apps that ship fast.',
    projects: ['Hair Creation', 'Chit Fund Manager'] },
  // ── Tools & DevOps
  { id: 'github', group: 'Tools & DevOps', name: 'GitHub', short: 'GitHub', category: 'Version Control', color: '#E6EDF3', icon: `${ICON}/github.svg`, mode: 'hole',
    description: 'Git workflows, pull-request reviews, Actions for CI and tidy, well-documented repositories.' },
  { id: 'gitlab', group: 'Tools & DevOps', name: 'GitLab', short: 'GitLab', category: 'Version Control & CI', color: '#FC6D26', icon: `${ICON}/gitlab.svg`,
    description: 'Merge-request workflows and GitLab CI/CD pipelines for automated testing and deployment.' },
]
// Sidebar: skills grouped under their category headings
const groups = computed(() => {
  const out = []
  skills.forEach((s, i) => {
    let g = out.find(x => x.name === s.group)
    if (!g) { g = { name: s.group, items: [] }; out.push(g) }
    g.items.push({ ...s, i })
  })
  return out
})
const N = skills.length
const STEP_VH = 55 // scroll distance per skill

const section = ref(null)
const canvas  = ref(null)
const idx      = ref(0)
const progress = ref(0)
const skill    = computed(() => skills[idx.value])
const pad = n => String(n).padStart(2, '0')

const jumpTo = (i) => {
  const el = section.value; if (!el) return
  const total = el.offsetHeight - window.innerHeight
  const top = el.getBoundingClientRect().top + window.scrollY
  window.scrollTo({ top: top + ((i + 0.5) / N) * total, behavior: 'smooth' })
}

// ── Shape sampling (logo → point cloud) ────────────────────────────────────
const SAMPLE = 220
function textShape (label) {
  const c = document.createElement('canvas'); c.width = c.height = SAMPLE
  const x = c.getContext('2d')
  x.fillStyle = '#fff'; x.textAlign = 'center'; x.textBaseline = 'middle'
  let fs = 120
  do { x.font = `800 ${fs}px Syne, Arial Black, sans-serif`; fs -= 4 } while (x.measureText(label).width > SAMPLE * 0.9 && fs > 20)
  x.fillText(label, SAMPLE / 2, SAMPLE / 2)
  return x.getImageData(0, 0, SAMPLE, SAMPLE)
}
function imageShape (url) {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.crossOrigin = 'anonymous'
    img.onload = () => {
      try {
        const c = document.createElement('canvas'); c.width = c.height = SAMPLE
        const x = c.getContext('2d')
        const iw = img.naturalWidth || 128, ih = img.naturalHeight || 128
        const s = (SAMPLE * 0.86) / Math.max(iw, ih)
        const w = iw * s, h = ih * s
        x.drawImage(img, (SAMPLE - w) / 2, (SAMPLE - h) / 2, w, h)
        resolve(x.getImageData(0, 0, SAMPLE, SAMPLE))
      } catch (e) { reject(e) }
    }
    img.onerror = reject
    img.src = url
  })
}
// Turn a rasterised logo into `count` particle positions.
//  mode 'alpha' — the drawn shape itself (default)
//  mode 'hole'  — the transparent cut-out inside a round badge (e.g. GitHub's Octocat)
// About half the particles go on outlines and colour boundaries, so shapes and
// inner details (letters, facets) read clearly instead of as a solid blob.
function pointsFrom (data, count, mode = 'alpha', edgeRatio = 0.5) {
  const d = data.data, S = SAMPLE
  const A = (x, y) => d[(y * S + x) * 4 + 3]
  const R = S / 2
  const on = new Uint8Array(S * S)
  for (let y = 0; y < S; y++) for (let x = 0; x < S; x++) {
    if (mode === 'hole') {
      const dx = x - R + 0.5, dy = y - R + 0.5
      on[y * S + x] = A(x, y) <= 110 && dx * dx + dy * dy < (R * 0.84) ** 2 ? 1 : 0
    } else on[y * S + x] = A(x, y) > 110 ? 1 : 0
  }
  const fill = [], edge = []
  const colDiff = (i, j) => {
    const a = i * 4, b = j * 4
    return Math.abs(d[a] - d[b]) + Math.abs(d[a + 1] - d[b + 1]) + Math.abs(d[a + 2] - d[b + 2])
  }
  for (let y = 1; y < S - 1; y++) for (let x = 1; x < S - 1; x++) {
    const i = y * S + x
    if (!on[i]) continue
    fill.push(x, y)
    const nb = [i - 1, i + 1, i - S, i + S]
    let isEdge = false
    for (const j of nb) {
      if (!on[j] || (mode === 'alpha' && colDiff(i, j) > 60)) { isEdge = true; break }
    }
    if (isEdge) edge.push(x, y)
  }
  const nf = fill.length / 2, ne = edge.length / 2
  if (nf < 50) return null
  const out = new Float32Array(count * 3)
  for (let i = 0; i < count; i++) {
    const useEdge = ne > 20 && Math.random() < edgeRatio
    const src = useEdge ? edge : fill
    const k = ((Math.random() * (useEdge ? ne : nf)) | 0) * 2
    out[i * 3]     = ((src[k] + Math.random()) / S - 0.5)
    out[i * 3 + 1] = -((src[k + 1] + Math.random()) / S - 0.5)
    out[i * 3 + 2] = (Math.random() - 0.5) * 0.12
  }
  return out
}

// Custom drawn shapes for logos that don't translate well from their SVG
const customShapes = {
  // JavaScript: rounded square outline with a bold "JS" in the corner, as in the badge
  js (x, S) {
    const m = S * 0.1, w = S - m * 2, r = S * 0.06
    x.strokeStyle = '#fff'; x.lineWidth = S * 0.05
    x.beginPath(); x.roundRect(m, m, w, w, r); x.stroke()
    x.fillStyle = '#fff'; x.textAlign = 'right'; x.textBaseline = 'alphabetic'
    x.font = `800 ${S * 0.42}px Syne, Arial Black, sans-serif`
    x.fillText('JS', S - m * 1.6, S - m * 1.55)
  },
}
function drawShape (fn) {
  const c = document.createElement('canvas'); c.width = c.height = SAMPLE
  const x = c.getContext('2d'); fn(x, SAMPLE)
  return x.getImageData(0, 0, SAMPLE, SAMPLE)
}

// ── Three.js scene ─────────────────────────────────────────────────────────
let cleanup = () => {}

onMounted(() => {
  if (!import.meta.client) return
  const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  const mobile = window.innerWidth < 760
  const COUNT = mobile ? 2600 : 5200

  const renderer = new THREE.WebGLRenderer({ canvas: canvas.value, alpha: true, antialias: false, powerPreference: 'high-performance' })
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  const scene  = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(45, 1, 0.1, 100)
  camera.position.z = 6

  const group = new THREE.Group(); scene.add(group)

  // Particle cloud
  const pos = new Float32Array(COUNT * 3)
  const vel = new Float32Array(COUNT * 3)
  const col = new Float32Array(COUNT * 3)
  const tint = new Float32Array(COUNT)
  const speed = new Float32Array(COUNT)
  const size = new Float32Array(COUNT)
  const phase = new Float32Array(COUNT)
  for (let i = 0; i < COUNT; i++) {
    const r = 0.6 + Math.random() * 1.4, a = Math.random() * Math.PI * 2, b = Math.acos(Math.random() * 2 - 1)
    pos[i * 3] = r * Math.sin(b) * Math.cos(a); pos[i * 3 + 1] = r * Math.sin(b) * Math.sin(a); pos[i * 3 + 2] = r * Math.cos(b)
    tint[i] = 0.55 + Math.random() * 0.7
    speed[i] = 0.6 + Math.random() * 0.8
    size[i] = 0.55 + Math.random() * 0.7
    phase[i] = Math.random() * Math.PI * 2
    col[i * 3] = col[i * 3 + 1] = col[i * 3 + 2] = 0.4
  }
  const geo = new THREE.BufferGeometry()
  geo.setAttribute('position', new THREE.BufferAttribute(pos, 3).setUsage(THREE.DynamicDrawUsage))
  geo.setAttribute('color',    new THREE.BufferAttribute(col, 3).setUsage(THREE.DynamicDrawUsage))
  geo.setAttribute('aSize',    new THREE.BufferAttribute(size, 1))
  geo.setAttribute('aPhase',   new THREE.BufferAttribute(phase, 1))
  const mat = new THREE.ShaderMaterial({
    uniforms: { uTime: { value: 0 }, uPR: { value: renderer.getPixelRatio() }, uScale: { value: 1 } },
    vertexShader: `
      attribute float aSize; attribute float aPhase;
      uniform float uTime; uniform float uPR; uniform float uScale;
      varying vec3 vColor;
      void main() {
        vec3 p = position;
        p += vec3(sin(uTime * 1.3 + aPhase), cos(uTime * 1.1 + aPhase * 1.7), sin(uTime * .9 + aPhase * 2.3)) * .006;
        vec4 mv = modelViewMatrix * vec4(p, 1.0);
        gl_PointSize = aSize * uPR * uScale * 26.0 / -mv.z;
        gl_Position = projectionMatrix * mv;
        vColor = color;
      }`,
    fragmentShader: `
      varying vec3 vColor;
      void main() {
        float d = length(gl_PointCoord - .5);
        float a = smoothstep(.5, .0, d);
        gl_FragColor = vec4(vColor, a * .95);
      }`,
    vertexColors: true, transparent: true, depthWrite: false, blending: THREE.AdditiveBlending,
  })
  const cloud = new THREE.Points(geo, mat); group.add(cloud)

  // Orbit rings
  const ringMat = new THREE.LineBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.22 })
  const mkRing = (r, tilt) => {
    const pts = []; for (let i = 0; i <= 128; i++) { const t = i / 128 * Math.PI * 2; pts.push(new THREE.Vector3(Math.cos(t) * r, Math.sin(t) * r, 0)) }
    const l = new THREE.Line(new THREE.BufferGeometry().setFromPoints(pts), ringMat); l.rotation.x = tilt; group.add(l); return l
  }
  const ring1 = mkRing(0.62, 1.15), ring2 = mkRing(0.7, -0.95)

  // Background dust
  const DUST = 700
  const dpos = new Float32Array(DUST * 3)
  for (let i = 0; i < DUST; i++) { dpos[i * 3] = (Math.random() - .5) * 24; dpos[i * 3 + 1] = (Math.random() - .5) * 14; dpos[i * 3 + 2] = -2 - Math.random() * 10 }
  const dgeo = new THREE.BufferGeometry(); dgeo.setAttribute('position', new THREE.BufferAttribute(dpos, 3))
  const dust = new THREE.Points(dgeo, new THREE.PointsMaterial({ color: 0x8a7f72, size: 0.03, transparent: true, opacity: 0.7, depthWrite: false }))
  scene.add(dust)

  // Target shapes (start with text, upgrade to logos when they load)
  const targets = skills.map(s =>
    (s.shape && pointsFrom(drawShape(customShapes[s.shape]), COUNT)) || pointsFrom(textShape(s.short), COUNT, 'alpha', 0.35))
  skills.forEach((s, i) => {
    if (!s.icon || s.shape) return
    imageShape(s.icon).then(d => {
      const p = pointsFrom(d, COUNT, s.mode || 'alpha')
      if (p) { targets[i] = p; if (i === idx.value) burst(0.025) }
    }).catch(() => {})
  })
  const colors = skills.map(s => new THREE.Color(s.color))
  const white = new THREE.Color('#F5EFE4')
  const curRing = new THREE.Color('#ffffff')

  // Layout: logo to the right on desktop, above text on mobile
  const view = { w: 1, h: 1 }
  const resize = () => {
    const w = canvas.value.clientWidth, h = canvas.value.clientHeight
    renderer.setSize(w, h, false)
    camera.aspect = w / h; camera.updateProjectionMatrix()
    const vh = 2 * Math.tan(THREE.MathUtils.degToRad(camera.fov / 2)) * camera.position.z
    const vw = vh * camera.aspect
    view.w = vw; view.h = vh
    if (w > 900) {
      const s = Math.min(vh * 0.58, vw * 0.3)
      group.scale.setScalar(s); group.position.set(vw * 0.13, 0, 0)
    } else {
      const s = Math.min(vw * 0.62, vh * 0.36)
      group.scale.setScalar(s); group.position.set(0, vh * 0.2, 0)
    }
    mat.uniforms.uScale.value = Math.max(0.8, Math.min(1.25, h / 800))
  }
  resize()

  // Interaction
  const mouse = { x: 0, y: 0, lx: 99, ly: 99, active: false }
  const onMove = (e) => {
    const r = canvas.value.getBoundingClientRect()
    mouse.x = ((e.clientX - r.left) / r.width) * 2 - 1
    mouse.y = -((e.clientY - r.top) / r.height) * 2 + 1
    // pointer in group-local space (z = 0 plane)
    const s = group.scale.x
    mouse.lx = (mouse.x * view.w / 2 - group.position.x) / s
    mouse.ly = (mouse.y * view.h / 2 - group.position.y) / s
    mouse.active = true
  }
  const onLeave = () => { mouse.active = false; mouse.lx = mouse.ly = 99 }
  section.value.addEventListener('pointermove', onMove)
  section.value.addEventListener('pointerleave', onLeave)

  function burst (amt = 0.06) {
    for (let i = 0; i < COUNT; i++) {
      vel[i * 3]     += (Math.random() - .5) * amt
      vel[i * 3 + 1] += (Math.random() - .5) * amt
      vel[i * 3 + 2] += (Math.random() - .5) * amt * 2.5
    }
  }

  // Scroll → active skill
  const onScroll = () => {
    const el = section.value; if (!el) return
    const r = el.getBoundingClientRect()
    const total = el.offsetHeight - window.innerHeight
    const p = Math.min(0.9999, Math.max(0, -r.top / total))
    progress.value = p
    const next = Math.min(N - 1, Math.floor(p * N))
    if (next !== idx.value) { idx.value = next; burst() }
  }
  window.addEventListener('scroll', onScroll, { passive: true })
  window.addEventListener('resize', resize)
  onScroll()

  // Only render while visible
  let visible = false
  const io = new IntersectionObserver(([e]) => { visible = e.isIntersecting; if (visible && !raf) raf = requestAnimationFrame(tick) }, { rootMargin: '100px' })
  io.observe(section.value)

  const clock = new THREE.Clock()
  let raf = 0
  const repelR = 0.22
  function tick () {
    if (!visible) { raf = 0; return }
    raf = requestAnimationFrame(tick)
    const t = clock.getElapsedTime()
    const tgt = targets[idx.value]
    const c = colors[idx.value]
    const spring = reduce ? 0.2 : 0.045
    for (let i = 0; i < COUNT; i++) {
      const j = i * 3
      let vx = vel[j], vy = vel[j + 1], vz = vel[j + 2]
      const k = spring * speed[i]
      vx = vx * 0.86 + (tgt[j] - pos[j]) * k
      vy = vy * 0.86 + (tgt[j + 1] - pos[j + 1]) * k
      vz = vz * 0.86 + (tgt[j + 2] - pos[j + 2]) * k
      if (mouse.active) {
        const dx = pos[j] - mouse.lx, dy = pos[j + 1] - mouse.ly
        const d2 = dx * dx + dy * dy
        if (d2 < repelR * repelR && d2 > 1e-6) {
          const d = Math.sqrt(d2), f = (1 - d / repelR) * 0.018
          vx += dx / d * f; vy += dy / d * f; vz += f * 0.6
        }
      }
      pos[j] += vx; pos[j + 1] += vy; pos[j + 2] += vz
      vel[j] = vx; vel[j + 1] = vy; vel[j + 2] = vz
      // colour: brand colour, some particles lifted toward ivory for sparkle
      const tn = tint[i], m = tn > 1.1 ? 0.45 : 0
      const r = (c.r * (1 - m) + white.r * m) * tn, g = (c.g * (1 - m) + white.g * m) * tn, b = (c.b * (1 - m) + white.b * m) * tn
      col[j] += (r - col[j]) * 0.06; col[j + 1] += (g - col[j + 1]) * 0.06; col[j + 2] += (b - col[j + 2]) * 0.06
    }
    geo.attributes.position.needsUpdate = true
    geo.attributes.color.needsUpdate = true
    mat.uniforms.uTime.value = t

    // gentle sway + follow pointer
    const ry = Math.sin(t * 0.35) * 0.18 + mouse.x * 0.35
    const rx = Math.cos(t * 0.3) * 0.06 - mouse.y * 0.22
    cloud.rotation.y += (ry - cloud.rotation.y) * 0.05
    cloud.rotation.x += (rx - cloud.rotation.x) * 0.05
    ring1.rotation.z = t * 0.25; ring2.rotation.z = -t * 0.18
    ring1.rotation.y = cloud.rotation.y; ring2.rotation.y = cloud.rotation.y
    curRing.lerp(c, 0.05); ringMat.color.copy(curRing)
    dust.rotation.y = t * 0.01; dust.position.x = -mouse.x * 0.3; dust.position.y = -mouse.y * 0.2

    renderer.render(scene, camera)
  }

  cleanup = () => {
    cancelAnimationFrame(raf); io.disconnect()
    window.removeEventListener('scroll', onScroll)
    window.removeEventListener('resize', resize)
    section.value?.removeEventListener('pointermove', onMove)
    section.value?.removeEventListener('pointerleave', onLeave)
    geo.dispose(); mat.dispose(); dgeo.dispose(); ringMat.dispose(); renderer.dispose()
  }
})
onBeforeUnmount(() => cleanup())
</script>

<template>
  <section id="skills" ref="section" class="sk" :style="{ height: `calc(${N * STEP_VH}vh + 100vh)` }">
    <div class="sk-stage">
      <canvas ref="canvas" class="sk-canvas" aria-hidden="true" />

      <div class="wrap sk-ui">
        <div class="sk-text">
          <p class="sk-eyebrow">
            <span class="sk-dot" :style="{ background: skill.color }" />
            {{ skill.group }}
            <span class="sk-count"><b>{{ pad(idx + 1) }}</b> / {{ pad(N) }}</span>
          </p>

          <Transition name="sk-swap" mode="out-in">
            <div :key="idx" class="sk-info">
              <p class="sk-cat" :style="{ color: skill.color }">{{ skill.category }}</p>
              <h2 class="sk-name">{{ skill.name }}</h2>
              <p class="sk-desc">{{ skill.description }}</p>
              <div class="sk-meta">
                <span v-if="skill.experience" class="sk-exp">{{ skill.experience }}</span>
                <span v-for="p in skill.projects" :key="p" class="sk-proj">{{ p }}</span>
              </div>
            </div>
          </Transition>
        </div>

        <nav class="sk-list" aria-label="Skills">
          <div v-for="g in groups" :key="g.name" class="sk-group">
            <p class="sk-gname" :class="{ on: g.name === skill.group }">{{ g.name }}</p>
            <ul>
              <li v-for="s in g.items" :key="s.id">
                <button :class="{ on: s.i === idx, done: s.i < idx }" :style="{ '--c': s.color }" @click="jumpTo(s.i)">
                  <i /> <span>{{ s.short }}</span>
                </button>
              </li>
            </ul>
          </div>
        </nav>
      </div>

      <div class="sk-bar" aria-hidden="true"><i :style="{ transform: `scaleX(${(idx + 1) / N})`, background: skill.color }" /></div>
      <p class="sk-hint" :class="{ hide: progress > 0.03 }" aria-hidden="true">
        <span>Scroll to explore</span>
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 5v14M6 13l6 6 6-6" /></svg>
      </p>
    </div>
  </section>
</template>

<style scoped>
.sk { position: relative; background: var(--ink); color: var(--ivory); }
.sk-stage {
  position: sticky; top: 0; height: 100vh; height: 100svh; overflow: hidden;
  background:
    radial-gradient(60% 70% at 70% 50%, rgba(194, 87, 26, .12), transparent 70%),
    repeating-linear-gradient(0deg, rgba(245, 239, 228, .028) 0 1px, transparent 1px 3px);
}
.sk-canvas { position: absolute; inset: 0; width: 100%; height: 100%; display: block; }
.sk-ui {
  position: relative; height: 100%; pointer-events: none;
  display: flex; align-items: center; justify-content: space-between; gap: 32px;
}
.sk-text { max-width: 470px; pointer-events: auto; }
.sk-eyebrow {
  display: flex; align-items: center; gap: 10px; margin: 0 0 28px;
  font-size: .8rem; letter-spacing: .14em; text-transform: uppercase; color: #B9AE9F; font-weight: 700;
}
.sk-dot { width: 9px; height: 9px; border-radius: 50%; transition: background .5s; box-shadow: 0 0 12px currentColor; }
.sk-count { margin-left: 8px; font-variant-numeric: tabular-nums; color: #8a7f72; }
.sk-count b { color: var(--ivory); }
.sk-cat { margin: 0 0 10px; font-weight: 700; font-size: .95rem; letter-spacing: .02em; }
.sk-name { font-size: clamp(2.6rem, 6vw, 5rem); line-height: .95; font-weight: 800; margin: 0 0 20px; }
.sk-desc { margin: 0 0 24px; color: #CFC5B7; font-size: clamp(1rem, 1.3vw, 1.12rem); max-width: 42ch; }
.sk-meta { display: flex; flex-wrap: wrap; gap: 8px; }
.sk-meta span { font-size: .8rem; font-weight: 600; padding: 6px 12px; border-radius: 999px; border: 1px solid rgba(245, 239, 228, .2); }
.sk-meta .sk-exp { background: var(--ivory); color: var(--ink); border-color: var(--ivory); }

/* glitchy swap between skills */
.sk-swap-enter-active { animation: skIn .55s cubic-bezier(.16, 1, .3, 1); }
.sk-swap-leave-active { animation: skOut .18s steps(2) forwards; }
@keyframes skIn {
  0%   { opacity: 0; transform: translateY(26px); clip-path: inset(0 0 100% 0); filter: blur(4px); }
  30%  { opacity: 1; clip-path: inset(0 0 40% 0); transform: translate(-6px, 10px); text-shadow: 4px 0 var(--orange-hi); }
  50%  { transform: translate(4px, 4px); text-shadow: -3px 0 #4fd1ff; }
  100% { opacity: 1; transform: none; clip-path: inset(0); filter: none; text-shadow: none; }
}
@keyframes skOut {
  0%   { opacity: 1; transform: none; }
  50%  { opacity: .6; transform: translateX(8px) skewX(-6deg); clip-path: inset(30% 0 20% 0); }
  100% { opacity: 0; transform: translateX(-10px); }
}

.sk-list { width: 170px; flex-shrink: 0; display: flex; flex-direction: column; gap: 14px; pointer-events: auto; }
.sk-list ul { list-style: none; margin: 0; padding: 0; display: flex; flex-direction: column; }
.sk-gname { margin: 0 0 2px; font-size: .68rem; letter-spacing: .16em; text-transform: uppercase; color: #5c544b; font-weight: 700; transition: color .3s; }
.sk-gname.on { color: #B9AE9F; }
.sk-list button {
  display: flex; align-items: center; gap: 12px; background: none; border: 0; padding: 3px 0;
  color: #6E655B; font-weight: 700; font-size: .8rem; letter-spacing: .04em; text-transform: uppercase;
  transition: color .3s;
}
.sk-list button i { width: 18px; height: 2px; background: currentColor; transition: width .4s cubic-bezier(.16, 1, .3, 1), background .3s; }
.sk-list button:hover { color: var(--ivory); }
.sk-list button.done { color: #9a8f82; }
.sk-list button.on { color: var(--ivory); }
.sk-list button.on i { width: 42px; background: var(--c); box-shadow: 0 0 10px var(--c); }

.sk-bar { position: absolute; left: 0; right: 0; bottom: 0; height: 3px; background: rgba(245, 239, 228, .08); }
.sk-bar i { display: block; height: 100%; transform-origin: left; transition: transform .5s cubic-bezier(.16, 1, .3, 1), background .5s; }

.sk-hint {
  position: absolute; left: 50%; bottom: 28px; transform: translateX(-50%); margin: 0;
  display: flex; align-items: center; gap: 8px; font-size: .78rem; letter-spacing: .14em; text-transform: uppercase;
  color: #B9AE9F; transition: opacity .4s; pointer-events: none;
}
.sk-hint svg { width: 16px; height: 16px; animation: bob 1.4s ease-in-out infinite; }
.sk-hint.hide { opacity: 0; }
@keyframes bob { 50% { transform: translateY(5px); } }

@media (max-width: 900px) {
  .sk-ui { align-items: flex-end; padding-bottom: 56px; }
  .sk-list { display: none; }
  .sk-text { max-width: none; }
  .sk-eyebrow { margin-bottom: 16px; }
  .sk-name { margin-bottom: 12px; }
  .sk-desc { margin-bottom: 16px; }
  .sk-hint { bottom: auto; top: 90px; }
}
@media (max-height: 760px) and (min-width: 901px) {
  .sk-list { gap: 8px; }
  .sk-list button { padding: 1px 0; font-size: .74rem; }
}
@media (max-height: 640px) and (max-width: 900px) {
  .sk-desc { display: none; }
}
</style>
