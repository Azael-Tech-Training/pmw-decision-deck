<script setup>
import { ref } from 'vue'

const props = defineProps({
  question: String,
  options: Array,
  explanation: String,
  questionNumber: Number
})

const selectedIndex = ref(null)
const submitted = ref(false)
const isCorrect = ref(false)

function selectOption(index) {
  if (submitted.value) return
  selectedIndex.value = index
}

function submit() {
  if (selectedIndex.value === null) return
  submitted.value = true
  isCorrect.value = props.options[selectedIndex.value].correct
}

function reset() {
  selectedIndex.value = null
  submitted.value = false
  isCorrect.value = false
}
</script>

<template>
  <div class="quiz-widget">
    <div class="quiz-title">Pregunta {{ questionNumber }}</div>
    <p class="quiz-question">{{ question }}</p>
    
    <div class="quiz-options">
      <button 
        v-for="(option, index) in options" 
        :key="index"
        class="quiz-option"
        :class="{
          selected: selectedIndex === index && !submitted,
          correct: submitted && index === options.findIndex(o => o.correct),
          incorrect: submitted && selectedIndex === index && !option.correct
        }"
        @click="selectOption(index)"
      >
        {{ option.text }}
      </button>
    </div>

    <div v-if="submitted" class="quiz-feedback" :class="isCorrect ? 'success' : 'error'">
      {{ isCorrect ? '¡Correcto! ' : 'Incorrecto. ' }}{{ explanation }}
    </div>

    <button 
      v-if="!submitted" 
      class="quiz-submit" 
      :disabled="selectedIndex === null"
      @click="submit"
    >
      Verificar respuesta
    </button>
    <button v-else class="quiz-reset" @click="reset">
      Intentar de nuevo
    </button>
  </div>
</template>

<style scoped>
.quiz-widget {
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 1.5rem;
  margin: 1rem 0;
}

.quiz-title {
  color: var(--accent);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  font-size: 0.875rem;
  margin-bottom: 0.75rem;
}

.quiz-question {
  color: var(--text-primary);
  font-size: 1.1rem;
  margin-bottom: 1rem;
  line-height: 1.5;
}

.quiz-options {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.quiz-option {
  background: var(--bg-tertiary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 0.75rem 1rem;
  color: var(--text-primary);
  text-align: left;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.95rem;
}

.quiz-option:hover:not(:disabled) {
  border-color: var(--accent);
  background: rgba(59, 130, 246, 0.1);
}

.quiz-option.selected {
  border-color: var(--accent);
  background: rgba(59, 130, 246, 0.2);
}

.quiz-option.correct {
  border-color: var(--accent-success);
  background: rgba(16, 185, 129, 0.2);
}

.quiz-option.incorrect {
  border-color: #ef4444;
  background: rgba(239, 68, 68, 0.2);
}

.quiz-feedback {
  padding: 0.75rem 1rem;
  border-radius: 8px;
  margin-bottom: 1rem;
  font-size: 0.95rem;
  line-height: 1.5;
}

.quiz-feedback.success {
  background: rgba(16, 185, 129, 0.15);
  border: 1px solid rgba(16, 185, 129, 0.3);
  color: #6ee7b7;
}

.quiz-feedback.error {
  background: rgba(239, 68, 68, 0.15);
  border: 1px solid rgba(239, 68, 68, 0.3);
  color: #fca5a5;
}

.quiz-submit, .quiz-reset {
  background: var(--accent);
  color: white;
  border: none;
  border-radius: 8px;
  padding: 0.6rem 1.5rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.quiz-submit:hover:not(:disabled) {
  background: var(--accent-hover);
}

.quiz-submit:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.quiz-reset {
  background: var(--bg-tertiary);
  border: 1px solid var(--border-color);
}

.quiz-reset:hover {
  background: var(--border-color);
}
</style>
