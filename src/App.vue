<template>
  <div class="container">
    <div class="row justify-content-center my-4">
      <div class="col col-12 col-lg-8">
        <div class="d-flex align-items-center">
          <h1 class="mb-0">curl2ffmpeg</h1>
          <button
            type="button"
            class="btn btn-sm btn-outline-secondary ms-auto"
            @click="toggleTheme"
          >
            {{ theme === 'dark' ? '☀️ Light' : '🌙 Dark' }}
          </button>
        </div>
        <a href="https://github.com/windyakin/curl2ffmpeg">GitHub Repository</a>
        <CommandCoverter />
      </div>
    </div>
    <div class="row justify-content-center my-4">
      <div class="col col-12 col-lg-8 text-end">
        &copy; 2021 windyakin
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch } from 'vue'
import CommandCoverter from '~/components/CommandConverter.vue'

const STORAGE_KEY = 'curl2ffmpeg-theme'

export default {
  components: {
    CommandCoverter
  },
  setup() {
    const preferredTheme = () => {
      const stored = localStorage.getItem(STORAGE_KEY)
      if (stored === 'light' || stored === 'dark') {
        return stored
      }
      return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
    }

    const theme = ref(preferredTheme())

    const applyTheme = (value) => {
      document.documentElement.setAttribute('data-bs-theme', value)
    }

    const toggleTheme = () => {
      theme.value = theme.value === 'dark' ? 'light' : 'dark'
    }

    watch(theme, (value) => {
      applyTheme(value)
      localStorage.setItem(STORAGE_KEY, value)
    })

    onMounted(() => {
      applyTheme(theme.value)
    })

    return {
      theme,
      toggleTheme
    }
  }
}
</script>
