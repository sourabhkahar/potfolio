<script setup>
const props = defineProps({ project: { type: Object, required: true } })
const host = computed(() => {
  try { return new URL(props.project.url).host.replace(/^www\./, '') } catch { return '' }
})
</script>

<template>
  <a class="card rv" :href="project.url" target="_blank" rel="noopener">
    <div class="cover">
      <div class="cover-bar" aria-hidden="true">
        <i /><i /><i />
        <span class="cover-url">{{ host }}</span>
      </div>
      <div class="cover-shot" :class="{ contain: project.fit === 'contain' }">
        <img :src="project.image" :alt="`${project.name} screenshot`" loading="lazy" />
      </div>
      <span class="go" aria-hidden="true">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2"><path d="M7 17L17 7M8 7h9v9" /></svg>
      </span>
    </div>
    <div class="card-body">
      <span class="kind">{{ project.kind }}</span>
      <div class="card-meta">
        <h3 class="card-name">{{ project.name }}</h3>
        <span class="visit">Visit ↗</span>
      </div>
      <div class="tags">
        <span v-for="t in project.tags" :key="t">{{ t }}</span>
      </div>
    </div>
  </a>
</template>
