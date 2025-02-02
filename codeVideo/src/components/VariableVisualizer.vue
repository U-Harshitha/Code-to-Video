<script setup lang="ts">
import { ref, defineProps, watch } from 'vue'

const props = defineProps(['steps'])
const currentStep = ref(0)
const visibleVariables = ref([])
const currentLine = ref('')
const isAnimating = ref(false)

const animateSteps = async () => {
  if (isAnimating.value) return
  isAnimating.value = true
  currentStep.value = 0
  visibleVariables.value = []
  
  for (let i = 0; i < props.steps.length; i++) {
    currentStep.value = i
    currentLine.value = props.steps[i].line
    visibleVariables.value = props.steps[i].variables
    await new Promise(resolve => setTimeout(resolve, 1000))
  }
  
  isAnimating.value = false
}

watch(() => props.steps, async (newSteps) => {
  if (newSteps && newSteps.length > 0) {
    await animateSteps()
  }
}, { deep: true })
</script>

<template>
  <div class="visualization">
    <div class="current-line" v-if="currentLine">
      Executing: <code>{{ currentLine }}</code>
    </div>
    <div v-for="(variable, index) in visibleVariables" :key="index" class="variable-container">
      <div class="variable-name">{{ variable.name }}</div>
      
      <!-- Single box for int and float -->
      <div v-if="!variable.isArray" class="box">
        {{ variable.value }}
      </div>

      <!-- Array visualization -->
      <div v-if="variable.isArray" class="array-container">
        <div v-for="(value, i) in variable.values" :key="i" class="array-element">
          <div class="array-index">[{{ i }}]</div>
          <div class="box">{{ value }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.visualization {
  flex: 1;
  padding: 1rem;
  border-left: 2px solid #ddd;
}

.current-line {
  margin-bottom: 1rem;
  padding: 0.5rem;
  background-color: #f8f8f8;
  border-radius: 4px;
  font-family: monospace;
}

.variable-container {
  margin-bottom: 1rem;
  opacity: 0;
  transform: translateY(20px);
  animation: fadeIn 0.5s ease-out forwards;
}

@keyframes fadeIn {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.variable-name {
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.box {
  border: 2px solid #333;
  padding: 0.5rem;
  min-width: 60px;
  text-align: center;
  background-color: #f0f0f0;
  transition: background-color 0.3s;
}

.array-container {
  display: flex;
  gap: 0.25rem;
}

.array-element {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.array-index {
  font-size: 0.8rem;
  color: #666;
  margin-bottom: 0.2rem;
}
</style>
