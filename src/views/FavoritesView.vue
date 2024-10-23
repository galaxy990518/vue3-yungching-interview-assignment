<template>
  <div class="favorites">
    <h1>我的最愛列表</h1>
    <div v-if="favorites.length === 0">還沒有添加任何最愛項目。</div>
    <div v-else>
      <WeatherTable :forecasts="favorites" :itemsPerPage="10">
        <template #actions="{ forecast }">
          <button @click="openEditDialog(forecast)" class="edit-favorite">
            <span class="button-text">編輯</span>
            <i class="icon fas fa-edit"></i>
          </button>
          <button @click="removeFavorite(forecast)" class="remove-favorite">
            <span class="button-text">移除最愛</span>
            <i class="icon fas fa-trash-alt"></i>
          </button>
        </template>
      </WeatherTable>
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

function loadFavorites() {
  const storedFavorites = localStorage.getItem('weatherFavorites')
  if (storedFavorites) {
    favorites.value = JSON.parse(storedFavorites)
  }
}

function saveFavorites() {
  localStorage.setItem('weatherFavorites', JSON.stringify(favorites.value))
}

function removeFavorite(forecast) {
  const index = favorites.value.findIndex(
    fav => fav.city === forecast.city && fav.startTime === forecast.startTime,
  )
  if (index !== -1) {
    favorites.value.splice(index, 1)
    saveFavorites()
  }
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
