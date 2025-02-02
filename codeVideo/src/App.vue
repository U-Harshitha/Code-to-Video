<script setup lang="ts">
import { ref } from 'vue'
import CodeInput from './components/CodeInput.vue'
import VariableVisualizer from './components/VariableVisualizer.vue'

const cCode = ref('')
const variables = ref([])

const handleCodeSubmit = (parsedVariables: any) => {
  variables.value = parsedVariables
}
</script>

<template>
  <div class="container">
    <CodeInput @submitCode="handleCodeSubmit" />
    <VariableVisualizer :variables="variables" />
  </div>
</template>

<style scoped>
.container {
  display: flex;
  gap: 2rem;
  padding: 2rem;
}
</style>
