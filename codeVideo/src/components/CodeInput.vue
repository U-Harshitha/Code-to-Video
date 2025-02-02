<script setup lang="ts">
import { ref, defineEmits } from 'vue'

const emit = defineEmits(['updateCode'])
const cCode = ref('')

// Helper function to evaluate arithmetic expressions
const evaluateExpression = (expr: string, currentVariables: Map<string, any>): number => {
  // Replace array accesses first
  let evaluatedExpr = expr.replace(/([a-zA-Z_][a-zA-Z0-9_]*)\[(\d+)\]/g, (match, name, index) => {
    const varObj = currentVariables.get(name)
    if (varObj && varObj.isArray) {
      return varObj.values[parseInt(index)].toString()
    }
    return '0'
  })

  // Then replace variable names with their values
  evaluatedExpr = evaluatedExpr.replace(/[a-zA-Z_][a-zA-Z0-9_]*/g, (match) => {
    const varObj = currentVariables.get(match)
    if (varObj) {
      return varObj.isArray ? '0' : varObj.value.toString()
    }
    return match // Keep the original text if it's not a variable (might be a function name)
  })

  // Handle increment/decrement operators
  evaluatedExpr = evaluatedExpr.replace(/(\d+)\+\+/g, '($1+1)')
  evaluatedExpr = evaluatedExpr.replace(/(\d+)--/g, '($1-1)')

  // Safely evaluate the expression
  try {
    // Remove any remaining invalid characters
    evaluatedExpr = evaluatedExpr.replace(/[^0-9+\-*/()%. ]/g, '')
    return Function(`"use strict"; return (${evaluatedExpr})`)()
  } catch (e) {
    console.error('Error evaluating expression:', e)
    return 0
  }
}

// Helper function to handle variable declarations
const handleDeclaration = (
  type: string, 
  declaration: string, 
  currentVariables: Map<string, any>
): { line: string; variables: any[] } | null => {
  declaration = declaration.trim()
  
  // Handle arrays
  const arrayMatch = declaration.match(/([a-zA-Z_][a-zA-Z0-9_]*)\s*\[(\d+)\]/)
  if (arrayMatch) {
    const [, name, size] = arrayMatch
    const varObj = { 
      type, 
      name, 
      isArray: true, 
      size: parseInt(size),
      values: new Array(parseInt(size)).fill(0)
    }
    currentVariables.set(name, varObj)
    return {
      line: `${type} ${declaration}`,
      variables: structuredClone(Array.from(currentVariables.values()))
    }
  }
  
  // Handle regular variables
  const varMatch = declaration.match(/([a-zA-Z_][a-zA-Z0-9_]*)\s*(?:=\s*(.+))?/)
  if (varMatch) {
    const [, name, value] = varMatch
    const evaluatedValue = value ? evaluateExpression(value, currentVariables) : 0
    const varObj = { 
      type, 
      name, 
      value: evaluatedValue
    }
    currentVariables.set(name, varObj)
    return {
      line: `${type} ${declaration}`,
      variables: structuredClone(Array.from(currentVariables.values()))
    }
  }
  
  return null
}

// Helper function to handle assignments
const handleAssignment = (
  line: string,
  currentVariables: Map<string, any>
): { line: string; variables: any[] } | null => {
  const assignmentMatch = line.match(/^\s*([a-zA-Z_][a-zA-Z0-9_]*(?:\[\d+\])?)\s*([+\-*/%]?=|\+\+|--)\s*(.+)?;/)
  if (!assignmentMatch) return null

  const [, left, operator, right] = assignmentMatch
  
  // Handle array assignment
  const arrayAssignMatch = left.match(/([a-zA-Z_][a-zA-Z0-9_]*)\[(\d+)\]/)
  if (arrayAssignMatch) {
    const [, name, index] = arrayAssignMatch
    const varObj = currentVariables.get(name)
    if (varObj && varObj.isArray) {
      const idx = parseInt(index)
      const currentValue = varObj.values[idx]
      
      switch(operator) {
        case '+=': varObj.values[idx] = currentValue + evaluateExpression(right, currentVariables); break;
        case '-=': varObj.values[idx] = currentValue - evaluateExpression(right, currentVariables); break;
        case '*=': varObj.values[idx] = currentValue * evaluateExpression(right, currentVariables); break;
        case '/=': varObj.values[idx] = currentValue / evaluateExpression(right, currentVariables); break;
        case '%=': varObj.values[idx] = currentValue % evaluateExpression(right, currentVariables); break;
        case '=': varObj.values[idx] = evaluateExpression(right, currentVariables); break;
      }
      
      return {
        line,
        variables: structuredClone(Array.from(currentVariables.values()))
      }
    }
  } else {
    // Handle regular variable assignment
    const varObj = currentVariables.get(left)
    if (varObj) {
      const currentValue = varObj.value
      
      switch(operator) {
        case '+=': varObj.value = currentValue + evaluateExpression(right, currentVariables); break;
        case '-=': varObj.value = currentValue - evaluateExpression(right, currentVariables); break;
        case '*=': varObj.value = currentValue * evaluateExpression(right, currentVariables); break;
        case '/=': varObj.value = currentValue / evaluateExpression(right, currentVariables); break;
        case '%=': varObj.value = currentValue % evaluateExpression(right, currentVariables); break;
        case '++': varObj.value = currentValue + 1; break;
        case '--': varObj.value = currentValue - 1; break;
        case '=': varObj.value = evaluateExpression(right, currentVariables); break;
      }
      
      return {
        line,
        variables: structuredClone(Array.from(currentVariables.values()))
      }
    }
  }
  
  return null
}

// Store variables globally
const globalVariables = new Map()

const parseCode = () => {
  const steps = []
  const lines = cCode.value.split('\n')
  let inMainFunction = false

  // Reset or use existing global variables
  const currentVariables = globalVariables

  lines.forEach(line => {
    line = line.trim()
    if (line.startsWith('#include') || !line) return
    
    if (line.match(/^\s*int\s+main\s*\(/)) {
      inMainFunction = true
      return
    }
    
    if (!inMainFunction) return
    
    if (line.includes('return') || line === '}') {
      inMainFunction = false
      return
    }

    // Handle declarations
    const combinedMatch = line.match(/^\s*(\w+)\s+([^;]+);/)
    if (combinedMatch) {
      const [, type, declarations] = combinedMatch
      declarations.split(',').forEach(declaration => {
        const step = handleDeclaration(type, declaration, currentVariables)
        if (step) steps.push(step)
      })
      return
    }
    
    // Handle assignments and operations
    const step = handleAssignment(line, currentVariables)
    if (step) steps.push(step)
  })

  emit('updateCode', steps)
}
</script>

<template>
  <div class="code-input">
    <textarea 
      v-model="cCode" 
      placeholder="#include <stdio.h>

int main() {
    int a[8], b = 5, sum = 10;
    sum = 8 * 7;        // Direct arithmetic
    sum = a[0] + b;     // Array and variable arithmetic
    sum += b;           // Compound assignment
    b++;               // Increment
    a[2] = sum * b;    // Array assignment with arithmetic
    sum *= 2;          // Compound multiplication
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
