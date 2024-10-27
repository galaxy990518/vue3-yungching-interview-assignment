<template>
  <div class="home">
    <h1>天氣預報</h1>
    <div class="select-wrapper">
      <label for="city-select">選擇縣市：</label>
      <div class="select-container">
        <select id="city-select" v-model="selectedCity" @change="fetchWeather">
          <option value="">全部</option>
          <option v-for="city in cities" :key="city" :value="city">
            {{ city }}
          </option>
        </select>
      </div>
    </div>

    <div v-if="isLoading">加載中...</div>
    <div v-else-if="error">錯誤：{{ error }}</div>
    <div v-else>
      <WeatherTable
        ref="weatherTable"
        :forecasts="weatherForecasts"
        :itemsPerPage="10"
        @selection-change="handleSelectionChange"
      />

      <button
        v-show="selectedForecasts.length > 0"
        @click="batchToggleFavorite"
        class="floating-button"
        :class="shouldAddToFavorites ? 'add-favorite' : 'remove-favorite'"
      >
        <i :class="shouldAddToFavorites ? 'fas fa-heart' : 'fas fa-trash'"></i>
        <span>
          {{ shouldAddToFavorites ? '加入最愛' : '移除最愛' }}
          ({{ selectedForecasts.length }})
        </span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import WeatherTable from '@/components/WeatherTable.vue'

const weatherTable = ref(null)
const weatherForecasts = ref([])
const isLoading = ref(false)
const error = ref(null)

const selectedCity = ref('')
const cities = [
  '基隆市',
  '臺北市',
  '新北市',
  '桃園市',
  '新竹市',
  '新竹縣',
  '宜蘭縣',
  '苗栗縣',
  '臺中市',
  '彰化縣',
  '南投縣',
  '雲林縣',
  '嘉義市',
  '嘉義縣',
  '臺南市',
  '高雄市',
  '屏東縣',
  '花蓮縣',
  '臺東縣',
  '澎湖縣',
  '金門縣',
  '連江縣',
]

const favorites = ref([])
const selectedForecasts = ref([])

const shouldAddToFavorites = computed(() => {
  return selectedForecasts.value.some(forecast => !isFavorite(forecast))
})

async function fetchWeather() {
  isLoading.value = true
  error.value = null
  weatherForecasts.value = []

  try {
    const apiKey = import.meta.env.VITE_API_KEY
    let url = `https://opendata.cwa.gov.tw/api/v1/rest/datastore/F-C0032-001?Authorization=${apiKey}`

    if (selectedCity.value) {
      url += `&locationName=${encodeURIComponent(selectedCity.value)}`
    }

    const response = await fetch(url)
    const data = await response.json()

    if (data.success === 'true') {
      const forecasts = data.records.location.flatMap(location => {
        return location.weatherElement[0].time.map((time, index) => ({
          city: location.locationName,
          startTime: time.startTime,
          endTime: time.endTime,
          weather:
            location.weatherElement[0].time[index].parameter.parameterName,
          pop: location.weatherElement[1].time[index].parameter.parameterName,
          minTemp:
            location.weatherElement[2].time[index].parameter.parameterName,
          maxTemp:
            location.weatherElement[4].time[index].parameter.parameterName,
          comfort:
            location.weatherElement[3].time[index].parameter.parameterName,
        }))
      })

      // 按照 cities 的順序排序
      weatherForecasts.value = forecasts.sort((a, b) => {
        return cities.indexOf(a.city) - cities.indexOf(b.city)
      })
    } else {
      throw new Error(data.message || '獲取天氣數據失敗')
    }
  } catch (err) {
    console.error('Error fetching weather:', err)
    error.value = err.message
  } finally {
    isLoading.value = false
  }
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

function isFavorite(forecast) {
  return favorites.value.some(
    fav => fav.city === forecast.city && fav.startTime === forecast.startTime,
  )
}

function handleSelectionChange(selected) {
  selectedForecasts.value = selected
}

function batchToggleFavorite() {
  if (shouldAddToFavorites.value) {
    const newFavorites = selectedForecasts.value.filter(
      forecast => !isFavorite(forecast),
    )
    favorites.value.push(...newFavorites)
  } else {
    favorites.value = favorites.value.filter(
      fav =>
        !selectedForecasts.value.some(
          selected =>
            selected.city === fav.city && selected.startTime === fav.startTime,
        ),
    )
  }

  saveFavorites()
  selectedForecasts.value = []
  weatherTable.value.resetSelection()
}

onMounted(() => {
  fetchWeather()
  loadFavorites()
})
</script>

<style lang="scss" scoped>
.select-wrapper {
  margin-bottom: 10px;
  display: flex;
  align-items: center;
  gap: 10px;

  label {
    white-space: nowrap;
    font-size: 16px;
    color: #333;
  }

  .select-container {
    width: 100%;
    max-width: 300px;

    #city-select {
      width: 100%;
      padding: 10px 15px;
      font-size: 16px;
      color: #333;
      background-color: white;
      border: 2px solid #e0e0e0;
      border-radius: 8px;
      cursor: pointer;

      // 移除預設樣式
      appearance: none;
      -webkit-appearance: none;
      -moz-appearance: none;

      // 自定義下拉箭頭的位置
      background: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath d='M2 4L6 8L10 4' stroke='%23666' stroke-width='2' fill='none'/%3E%3C/svg%3E")
        no-repeat;
      background-position: right 10px center;
      background-color: white;
      padding-right: 30px;

      &:hover {
        border-color: #4a90e2;
      }

      &:focus {
        outline: none;
        border-color: #4a90e2;
        box-shadow: 0 0 5px rgba(74, 144, 226, 0.3);
      }

      option {
        padding: 10px;

        &:checked {
          background-color: #f5f5f5;
        }
      }
    }
  }
}
</style>
