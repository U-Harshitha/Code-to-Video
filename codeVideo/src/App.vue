<script setup lang="ts">
import { ref } from 'vue'
import CodeInput from './components/CodeInput.vue'
import VariableVisualizer from './components/VariableVisualizer.vue'

const steps = ref([])

const handleCodeUpdate = (newSteps: any) => {
  steps.value = newSteps
}
</script>

<template>
  <div class="container">
    <CodeInput @updateCode="handleCodeUpdate" />
    <VariableVisualizer :steps="steps" />
  </div>
</template>

<style scoped>
.container {
  display: flex;
  gap: 2rem;
  padding: 2rem;
}
</style>
