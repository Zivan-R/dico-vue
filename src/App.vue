<template>
  <main style="min-height: 100vh; display: flex; flex-direction: column; align-items: center; padding-top: 15vh;">
    <h1 style="font-size: 2.5rem; font-weight: 700; color: #fff;">Dictionary</h1>
    <input
      v-model="word"
      @keyup.enter="lookup"
      placeholder="Enter a word"
      style="padding: 1em; font-size: 1.2rem; border-radius: 8px; border: 1px solid #ccc; margin: 20px 0; width: 300px;"
      autofocus
    />
    <button
      @click="lookup"
      style="padding: 0.75em 2em; font-size: 1.1rem; border-radius: 8px; border: none; background: #2d8cf0; color: #fff; cursor: pointer;"
    >Search</button>
    <div v-if="error" style="color: #d32f2f; margin-top: 20px;">{{ error }}</div>
    <div v-if="entries.length" class="slider-container">
      <button
        class="arrow"
        @click="prevEntry"
        :disabled="currentIndex === 0"
      >←</button>
      <section class="entry-box">
        <div style="text-align: center; margin-bottom: 1.5em;">
          <strong style="font-size: 1.5rem;">Word:</strong> {{ result.word }}
        </div>
        <div style="text-align: center; margin-bottom: 1.5em;">
          <div><strong>Part of Speech:</strong> {{ currentEntry.partOfSpeech }}</div>
        </div>
        <div style="text-align: center; margin-bottom: 1.5em;">
          <div><strong>Definition:</strong> {{ currentEntry.definition }}</div>
        </div>
        <div v-if="currentEntry.example">
          <div class="divider"></div>
          <div style="text-align: center; margin-top: 1em;"><strong>Example:</strong> "{{ currentEntry.example }}"</div>
        </div>
      </section>
      <button
        class="arrow"
        @click="nextEntry"
        :disabled="currentIndex === entries.length - 1"
      >→</button>
    </div>
  </main>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import axios from 'axios'

const word = ref('')
const result = ref(null)
const error = ref('')
const currentIndex = ref(0)
const apiBase = import.meta.env.VITE_API_URL || "http://localhost:8000"
const lookup = async () => {
  result.value = null
  error.value = ''
  if (!word.value) return
  try {
    const response = await axios.post(`${apiBase}/lookup`, { word: word.value })
    result.value = response.data[0]
  } catch {
    error.value = 'Word not found or server error.'
  }
}

const entries = computed(() => {
  if (!result.value || !result.value.meanings) return []
  const list = []
  result.value.meanings.forEach((meaning) => {
    meaning.definitions.forEach((def) => {
      list.push({
        partOfSpeech: meaning.partOfSpeech,
        definition: def.definition,
        example: def.example || ''
      })
    })
  })
  return list
})

const currentEntry = computed(() => entries.value[currentIndex.value] || {})

const nextEntry = () => {
  if (currentIndex.value < entries.value.length - 1) currentIndex.value++
}

const prevEntry = () => {
  if (currentIndex.value > 0) currentIndex.value--
}

watch(result, () => {
  currentIndex.value = 0
})
</script>

<style scoped>
.slider-container {
  display: flex;
  align-items: flex-start;
  margin-top: 40px;
}

.entry-box {
  width: 500px;
  padding: 2em;
  border-radius: 12px;
  background: #1a1a2e;
  color: #fff;
  box-shadow: 0 2px 16px rgba(0, 0, 0, 0.5);
  margin: 0 1em;
  box-sizing: border-box;
}

.divider {
  width: 30%;
  height: 1px;
  background: #fff;
  margin: 1em auto;
}

.arrow {
  background: #2d8cf0;
  color: #fff;
  border: none;
  padding: 0.75em 1em;
  font-size: 1.5rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
}

.arrow:disabled {
  opacity: 0.5;
  cursor: default;
}

.arrow:hover:not(:disabled) {
  background: #fff;
  color: #000;
}
</style>
