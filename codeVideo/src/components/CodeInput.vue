<script setup lang="ts">
import { ref, defineEmits } from 'vue'

const emit = defineEmits(['submitCode'])
const cCode = ref('')

const parseCode = () => {
  const variables = []
  const lines = cCode.value.split('\n')

  lines.forEach(line => {
    line = line.trim()

    // Ignore function definitions (e.g., int main())
    if (line.match(/^\s*\w+\s+main\s*\(/)) return

    // Match integer arrays (e.g., int a[8];)
    const arrayMatch = line.match(/^\s*int\s+([a-zA-Z_][a-zA-Z0-9_]*)\s*\[(\d+)\]\s*;/)
    if (arrayMatch) {
      const [, name, size] = arrayMatch
      variables.push({ type: 'int', name, isArray: true, size: parseInt(size) })
      return
    }

    // Match multiple integer declarations (e.g., int a, b, sum = 0;)
    const intMatch = line.match(/^int\s+([^;]+);/)
    if (intMatch) {
      const declarations = intMatch[1].split(',').map(v => v.trim().split('=')[0].trim())
      declarations.forEach(name => {
        if (name) variables.push({ type: 'int', name })
      })
      return
    }

    // Match float declarations (e.g., float hh = 0;)
    const floatMatch = line.match(/^float\s+([^;]+);/)
    if (floatMatch) {
      const declarations = floatMatch[1].split(',').map(v => v.trim().split('=')[0].trim())
      declarations.forEach(name => {
        if (name) variables.push({ type: 'float', name })
      })
      return
    }
  })

  emit('submitCode', variables)
}
</script>

<template>
  <div class="code-input">
    <textarea v-model="cCode" placeholder="Paste your C code here..."></textarea>
    <button @click="parseCode" class="submit-btn">Visualize Code</button>
  </div>
</template>



<style scoped>
.code-input {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

textarea {
  width: 100%;
  height: 500px;
  font-family: monospace;
  padding: 1rem;
}

.submit-btn {
  padding: 0.5rem 1rem;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s;
}

.submit-btn:hover {
  background-color: #45a049;
}
</style>
