<template>
  <div class="favorites">
    <h1>我的最愛列表</h1>
    <div v-if="favorites.length === 0">還沒有添加任何最愛項目。</div>
    <div v-else>
      <WeatherTable
        :forecasts="favorites"
        :itemsPerPage="10"
        @selection-change="handleSelectionChange"
      >
        <template #actions="{ forecast }">
          <button @click="openEditDialog(forecast)" class="edit-favorite">
            <span class="button-text">編輯</span>
            <i class="icon fas fa-edit"></i>
          </button>
        </template>
      </WeatherTable>

      <button
        v-show="selectedForecasts.length > 0"
        @click="batchRemoveFavorites"
        class="floating-button remove-favorite"
      >
        <i class="fas fa-trash"></i>
        <span>移除已選項目 ({{ selectedForecasts.length }})</span>
      </button>
    </div>
    <EditDialog
      v-model:show="showEditDialog"
      :forecast="editingForecast"
      @save="saveEdit"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import EditDialog from '@/components/EditDialog.vue'
import WeatherTable from '@/components/WeatherTable.vue'

const favorites = ref([])
const showEditDialog = ref(false)
const editingForecast = ref({})
const selectedForecasts = ref([])

function loadFavorites() {
  const storedFavorites = localStorage.getItem('weatherFavorites')
  if (storedFavorites) {
    favorites.value = JSON.parse(storedFavorites)
  }
}

function saveFavorites() {
  localStorage.setItem('weatherFavorites', JSON.stringify(favorites.value))
}

function handleSelectionChange(selected) {
  selectedForecasts.value = selected
}

function batchRemoveFavorites() {
  selectedForecasts.value.forEach(forecast => {
    const index = favorites.value.findIndex(
      fav => fav.city === forecast.city && fav.startTime === forecast.startTime,
    )
    if (index !== -1) {
      favorites.value.splice(index, 1)
    }
  })
  saveFavorites()
  selectedForecasts.value = []
}

function openEditDialog(forecast) {
  editingForecast.value = { ...forecast }
  showEditDialog.value = true
}

function saveEdit(updatedForecast) {
  const index = favorites.value.findIndex(
    fav =>
      fav.city === updatedForecast.city &&
      fav.startTime === updatedForecast.startTime,
  )
  if (index !== -1) {
    favorites.value[index] = updatedForecast
    saveFavorites()
  }
  showEditDialog.value = false
}

onMounted(() => {
  loadFavorites()
})
</script>
