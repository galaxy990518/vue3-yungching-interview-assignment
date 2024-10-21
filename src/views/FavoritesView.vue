<template>
  <div class="favorites">
    <h1>我的最愛列表</h1>
    <div v-if="favorites.length === 0">還沒有添加任何最愛項目。</div>
    <div v-else>
      <div class="weather-list">
        <table>
          <thead>
            <tr>
              <th>縣市</th>
              <th>預報時間</th>
              <th>天氣狀況</th>
              <th>降雨機率</th>
              <th>最低溫</th>
              <th>最高溫</th>
              <th>舒適度</th>
              <th>操作</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="forecast in paginatedFavorites"
              :key="forecast.startTime + forecast.city"
            >
              <td>{{ forecast.city }}</td>
              <td>
                {{ formatDate(forecast.startTime) }} -
                {{ formatDate(forecast.endTime) }}
              </td>
              <td>{{ forecast.weather }}</td>
              <td>{{ forecast.pop }}%</td>
              <td>{{ forecast.minTemp }}°C</td>
              <td>{{ forecast.maxTemp }}°C</td>
              <td>{{ forecast.comfort }}</td>
              <td>
                <button @click="openEditDialog(forecast)" class="edit-favorite">
                  編輯
                </button>
                <button
                  @click="removeFavorite(forecast)"
                  class="remove-favorite"
                >
                  移除最愛
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="pagination">
        <button @click="prevPage" :disabled="currentPage === 1">上一頁</button>
        <span>第 {{ currentPage }} 頁，共 {{ totalPages }} 頁</span>
        <button @click="nextPage" :disabled="currentPage === totalPages">
          下一頁
        </button>
      </div>

      <!-- 編輯天氣預報對話框 -->
      <EditDialog
        v-model:show="showEditDialog"
        :forecast="editingForecast"
        @save="saveEdit"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import EditDialog from '@/components/EditDialog.vue'

const favorites = ref([])
const currentPage = ref(1)
const itemsPerPage = 10

const paginatedFavorites = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  const end = start + itemsPerPage
  return favorites.value.slice(start, end)
})

const totalPages = computed(() => {
  return Math.ceil(favorites.value.length / itemsPerPage)
})

function prevPage() {
  if (currentPage.value > 1) {
    currentPage.value--
  }
}

function nextPage() {
  if (currentPage.value < totalPages.value) {
    currentPage.value++
  }
}

function formatDate(dateString) {
  const date = new Date(dateString)
  return date.toLocaleString('zh-TW')
}

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

const showEditDialog = ref(false)
const editingForecast = ref({})

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

<style lang="scss" scoped>
.edit-favorite {
  background-color: $primary-color;
  color: white;
  border-radius: $border-radius;
  margin-right: 10px;

  &:hover {
    background-color: darken($primary-color, 10%);
  }
}
</style>
