<script setup lang="ts">
import { ref, defineEmits } from 'vue'

const emit = defineEmits(['submitCode'])
const cCode = ref('')

const parseCode = () => {
  const variables = []
  const lines = cCode.value.split('\n')
  
  let inMainFunction = false

  lines.forEach(line => {
    line = line.trim()
    
    // Skip includes and empty lines
    if (line.startsWith('#include') || !line) return
    
    // Check if we're entering main function
    if (line.match(/^\s*int\s+main\s*\(/)) {
      inMainFunction = true
      return
    }
    
    // Only parse declarations inside main
    if (!inMainFunction) return
    
    // Check for end of main
    if (line.includes('return') || line === '}') {
      inMainFunction = false
      return
    }

    // Match combined declarations (e.g., int a[8], b, sum = 0;)
    const combinedMatch = line.match(/^\s*(\w+)\s+([^;]+);/)
    if (combinedMatch) {
      const type = combinedMatch[1]
      const declarations = combinedMatch[2].split(',')
      
      declarations.forEach(declaration => {
        declaration = declaration.trim()
        
        // Handle arrays (e.g., a[8])
        const arrayMatch = declaration.match(/([a-zA-Z_][a-zA-Z0-9_]*)\s*\[(\d+)\]/)
        if (arrayMatch) {
          const [, name, size] = arrayMatch
          variables.push({ 
            type, 
            name, 
            isArray: true, 
            size: parseInt(size),
            values: new Array(parseInt(size)).fill(0)
          })
          return
        }
        
        // Handle regular variables with or without initialization
        const varMatch = declaration.match(/([a-zA-Z_][a-zA-Z0-9_]*)\s*(?:=\s*([^,\s]+))?/)
        if (varMatch) {
          const [, name, value] = varMatch
          variables.push({ 
            type, 
            name, 
            value: value ? parseFloat(value) : 0
          })
        }
      })
    }
  })

  emit('submitCode', variables)
}
</script>

<template>
  <div class="code-input">
    <textarea 
      v-model="cCode" 
      placeholder="#include <stdio.h>

int main() {
    // Your C code here
    return 0;
}"
    ></textarea>
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
