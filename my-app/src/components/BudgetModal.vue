<template>
    <div class="modal-backdrop" @keydown.esc.prevent="onCancel">
      <div class="modal" role="dialog" aria-modal="true">
        <h2>Enter Budget for {{ state.name }}</h2>
        <form @submit.prevent="onSave">
          <div style="margin:1.5em 0;">
            <label :for="inputId">Budget (0-1000):</label>
            <input
              :id="inputId"
              ref="inputRef"
              type="number"
              min="0"
              max="1000"
              v-model="val"
              required
              @keydown.enter.prevent="onSave"
              autocomplete="off"
            />
          </div>
          <div v-if="error" style="color:#c00; margin-bottom:1em;">{{ error }}</div>
          <div style="text-align:right;">
            <button type="button" class="secondary" @click="onCancel">Cancel</button>
            <button type="submit" class="primary">Save</button>
          </div>
        </form>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, computed } from 'vue'
  const props = defineProps({
    state: { type: Object, required: true },
    currentValue: { type: Number, default: null },
  })
  const emit = defineEmits(['save', 'cancel'])
  const val = ref(props.currentValue ?? '')
  const error = ref('')
  const inputRef = ref(null)
  const inputId = computed(() => `budget-input-${props.state.fips}`)
  
  onMounted(() => {
    setTimeout(() => {
      if (inputRef.value) inputRef.value.focus()
    }, 5)
  })
  
  function onSave() {
    const num = Number(val.value)
    if (isNaN(num) || num < 0 || num > 1000) {
      error.value = 'Please enter a number from 0 to 1000.'
      return
    }
    error.value = ''
    emit('save', num)
  }
  function onCancel() {
    emit('cancel')
  }
  </script>
  
  <style scoped>
  /* Uses global .modal/.modal-backdrop, see main.css */
  </style>
  