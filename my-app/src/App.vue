<template>
  <div>
    <h1 style="text-align:center; margin-top:1rem;">US State Budget Entry Map</h1>
    <UsMap
      :budgets="budgets"
      @state-click="onStateClick"
    />
    <BudgetModal
      v-if="modalOpen"
      :state="modalState"
      :current-value="budgets[modalState?.fips]"
      @save="onSaveBudget"
      @cancel="closeModal"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import UsMap from './components/UsMap.vue'
import BudgetModal from './components/BudgetModal.vue'

const budgets = ref({})  // { fips: budget }
const modalOpen = ref(false)
const modalState = ref(null) // { name, fips }

function onStateClick(stateInfo) {
  modalState.value = stateInfo // { name, fips }
  modalOpen.value = true
}

function closeModal() {
  modalOpen.value = false
}

function onSaveBudget(val) {
  if (modalState.value && val !== null) {
    budgets.value = {
      ...budgets.value,
      [modalState.value.fips]: val
    }
  }
  closeModal()
}
</script>
