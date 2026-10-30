<script setup>
import * as THREE from 'three'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

if (import.meta.client) {
  gsap.registerPlugin(ScrollTrigger)
}

const container = ref(null)
const canvas = ref(null)

const icons = [
  'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/vuejs/vuejs-original.svg',
  'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/javascript/javascript-original.svg',
  'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/typescript/typescript-original.svg',
  'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/nodejs/nodejs-original.svg',
  'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/tailwindcss/tailwindcss-original.svg',
  'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/postgresql/postgresql-original.svg',
  'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/mongodb/mongodb-original.svg',
  'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/laravel/laravel-original.svg',
  'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/nuxtjs/nuxtjs-original.svg'
]

onMounted(() => {
  if (!import.meta.client) return

  // Scene setup
  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  const renderer = new THREE.WebGLRenderer({ 
    canvas: canvas.value, 
    alpha: true, 
    antialias: true 
  })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

  // Groups
  const mainGroup = new THREE.Group()
  const particlesGroup = new THREE.Group()
  scene.add(mainGroup)
  scene.add(particlesGroup)

  // Lights
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.4)
  scene.add(ambientLight)

  const pointLight1 = new THREE.PointLight(0xF2613F, 3, 50)
  pointLight1.position.set(5, 5, 5)
  scene.add(pointLight1)

  const pointLight2 = new THREE.PointLight(0x9B3922, 3, 50)
  pointLight2.position.set(-5, -5, 5)
  scene.add(pointLight2)

  // Particle System
  const particlesGeometry = new THREE.BufferGeometry()
  const particlesCount = 2000
  const posArray = new Float32Array(particlesCount * 3)
  for (let i = 0; i < particlesCount * 3; i++) {
    posArray[i] = (Math.random() - 0.5) * 60
  }
  particlesGeometry.setAttribute('position', new THREE.BufferAttribute(posArray, 3))
  const particlesMaterial = new THREE.PointsMaterial({
    size: 0.04,
    color: 0xF2613F,
    transparent: true,
    opacity: 0.4,
    blending: THREE.AdditiveBlending
  })
  const particlesMesh = new THREE.Points(particlesGeometry, particlesMaterial)
  particlesGroup.add(particlesMesh)

  // Skills Text
  const createTextTexture = () => {
    const canvas = document.createElement('canvas')
    const ctx = canvas.getContext('2d')
    canvas.width = 1024
    canvas.height = 256
    
    const gradient = ctx.createLinearGradient(0, 0, 1024, 0)
    gradient.addColorStop(0, '#481E14')
    gradient.addColorStop(0.5, '#F2613F')
    gradient.addColorStop(1, '#481E14')
    
    ctx.fillStyle = gradient
    ctx.font = '900 180px Outfit'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    ctx.shadowColor = 'rgba(242, 97, 63, 0.4)'
    ctx.shadowBlur = 30
    ctx.fillText('SKILLS', 512, 128)
    return new THREE.CanvasTexture(canvas)
  }

  const textMaterial = new THREE.MeshBasicMaterial({ 
    map: createTextTexture(),
    transparent: true,
    side: THREE.DoubleSide
  })
  const textGeometry = new THREE.PlaneGeometry(10, 2.5)
  const textMesh = new THREE.Mesh(textGeometry, textMaterial)
  textMesh.position.z = 1
  mainGroup.add(textMesh)

  /**
   * Load an SVG URL by drawing it onto an offscreen canvas and returning a
   * callback-driven setup. We do NOT mutate a texture's .image after it has
   * already been uploaded to the GPU (that causes
   * "GL_INVALID_VALUE: glTexSubImage2DRobustANGLE: Offset overflows texture dimensions").
   * Instead we:
   *   1. Start with a 1×1 placeholder CanvasTexture (valid, zero-cost upload).
   *   2. Once the SVG is rasterized onto a 256×256 canvas, dispose the old
   *      texture and assign a brand-new CanvasTexture to the material's .map.
   */
  const loadIconIntoMaterial = (url, material) => {
    const img = new Image()
    img.crossOrigin = 'anonymous'
    img.onload = () => {
      const offscreen = document.createElement('canvas')
      offscreen.width = 256
      offscreen.height = 256
      const ctx2d = offscreen.getContext('2d')
      ctx2d.drawImage(img, 0, 0, 256, 256)

      // Dispose old placeholder texture to free GPU memory
      if (material.map) material.map.dispose()

      // Assign a fresh texture — this triggers a clean full upload, not a sub-update
      material.map = new THREE.CanvasTexture(offscreen)
      material.needsUpdate = true
    }
    // onerror: placeholder stays, no GPU error
    img.src = url
  }

  // Floating Icons in a sphere
  const iconMeshes = []
  icons.forEach((url, i) => {
    // Placeholder: a 1×1 transparent canvas — always a valid WebGL texture size
    const placeholderCanvas = document.createElement('canvas')
    placeholderCanvas.width = 1
    placeholderCanvas.height = 1
    const cardGroup = new THREE.Group()
    
    const iconMaterial = new THREE.MeshBasicMaterial({ 
      map: new THREE.CanvasTexture(placeholderCanvas), 
      transparent: true, 
      side: THREE.DoubleSide 
    })
    // Async: once SVG is rasterized, replace .map with the real 256×256 texture
    loadIconIntoMaterial(url, iconMaterial)

    const iconGeometry = new THREE.PlaneGeometry(0.8, 0.8)
    const iconMesh = new THREE.Mesh(iconGeometry, iconMaterial)
    iconMesh.position.z = 0.01
    cardGroup.add(iconMesh)
    
    const cardGeometry = new THREE.PlaneGeometry(1.2, 1.2)
    const cardMaterial = new THREE.MeshStandardMaterial({ 
      color: 0x9B3922,
      transparent: true,
      opacity: 0.6,
      metalness: 0.9,
      roughness: 0.1,
      side: THREE.DoubleSide,
      emissive: 0x9B3922,
      emissiveIntensity: 0.2
    })
    const cardMesh = new THREE.Mesh(cardGeometry, cardMaterial)
    cardGroup.add(cardMesh)
    
    // Add a bezel for the card
    const bezelGeo = new THREE.EdgesGeometry(cardGeometry)
    const bezelMat = new THREE.LineBasicMaterial({ color: 0xF2613F, transparent: true, opacity: 0.4 })
    const bezel = new THREE.LineSegments(bezelGeo, bezelMat)
    cardGroup.add(bezel)
    
    const phi = Math.acos(-1 + (2 * i) / icons.length)
    const theta = Math.sqrt(icons.length * Math.PI) * phi
    const radius = 6
    
    cardGroup.position.set(
      radius * Math.cos(theta) * Math.sin(phi),
      radius * Math.sin(theta) * Math.sin(phi),
      radius * Math.cos(phi)
    )
    
    mainGroup.add(cardGroup)
    iconMeshes.push({ group: cardGroup, initialPos: cardGroup.position.clone(), i })
  })

  camera.position.z = 12

  // Mouse interactivity
  let mouseX = 0, mouseY = 0
  const onMouseMove = (e) => {
    mouseX = (e.clientX / window.innerWidth) - 0.5
    mouseY = (e.clientY / window.innerHeight) - 0.5
  }
  window.addEventListener('mousemove', onMouseMove)

  // Animation Loop
  const clock = new THREE.Clock()
  const animate = () => {
    const elapsedTime = clock.getElapsedTime()
    mainGroup.rotation.y += (mouseX * 0.5 - mainGroup.rotation.y) * 0.05
    mainGroup.rotation.x += (mouseY * 0.5 - mainGroup.rotation.x) * 0.05
    particlesGroup.rotation.y = elapsedTime * 0.05

    iconMeshes.forEach(({ group, initialPos, i }) => {
      group.position.y = initialPos.y + Math.sin(elapsedTime * 0.5 + i) * 0.2
      group.lookAt(camera.position)
    })

    renderer.render(scene, camera)
    requestAnimationFrame(animate)
  }
  animate()

  // GSAP ScrollTrigger
  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: container.value,
      start: 'top top',
      end: 'bottom top',
      scrub: 1.5,
    }
  })

  tl.to(mainGroup.position, { z: 50, ease: 'power2.inOut' })
    .to(mainGroup.scale, { x: 5, y: 5, z: 5, ease: 'power2.inOut' }, 0)
    .to(canvas.value, { opacity: 0, filter: 'blur(40px)', duration: 0.3 }, 0.7)

  const handleResize = () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  }
  window.addEventListener('resize', handleResize)

  onUnmounted(() => {
    window.removeEventListener('resize', handleResize)
    window.removeEventListener('mousemove', onMouseMove)
    renderer.dispose()
  })
})
</script>

<template>
  <div ref="container" class="relative h-[120vh] w-full bg-[#0C0C0C]">
    <div class="sticky top-0 h-screen w-full overflow-hidden flex items-center justify-center">
      <canvas ref="canvas" class="w-full h-full" />
      <div class="absolute bottom-12 left-0 right-0 flex flex-col items-center gap-4 animate-bounce z-20">
        <span class="text-[10px] font-black tracking-[0.5em] text-[#F2613F] uppercase text-center w-full">Glitch the Skills</span>
        <div class="w-px h-12 bg-gradient-to-b from-[#F2613F] to-transparent mx-auto" />
      </div>
    </div>
  </div>
</template>

<style scoped>
canvas {
  display: block;
}
</style>
