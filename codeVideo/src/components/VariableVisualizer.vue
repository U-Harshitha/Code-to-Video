<script setup lang="ts">
import { ref, defineProps, watch } from 'vue'

const props = defineProps(['variables'])
const visibleVariables = ref([])

const animateVariables = async () => {
  visibleVariables.value = []
  for (let i = 0; i < props.variables.length; i++) {
    await new Promise(resolve => setTimeout(resolve, 500))
    visibleVariables.value.push(props.variables[i])
  }
}

// Watch for new variables and trigger animation
watch(() => props.variables, animateVariables, { deep: true })
</script>

<template>
  <div class="visualization">
    <div v-for="(variable, index) in visibleVariables" :key="index" class="variable-container">
      <div class="variable-name">{{ variable.name }}</div>
      
      <!-- Single box for int and float -->
      <div v-if="variable.type === 'int' || variable.type === 'float'" class="box">
        {{ variable.name }}
      </div>

      <!-- Array visualization -->
      <div v-if="variable.isArray" class="array-container">
        <div v-for="n in variable.size" :key="n" class="box">
          {{ variable.name }}[{{ n-1 }}]
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

.box {
  border: 2px solid #333;
  padding: 0.5rem;
  min-width: 60px;
  text-align: center;
  background-color: #f0f0f0;
}

.array-container {
  display: flex;
  gap: 0.25rem;
}
</style>
