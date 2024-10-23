<template>
  <div class="home">
    <h1>天氣預報</h1>
    <div>
      <label for="city-select">選擇縣市：</label>
      <select id="city-select" v-model="selectedCity" @change="fetchWeather">
        <option value="">全部</option>
        <option v-for="city in cities" :key="city" :value="city">
          {{ city }}
        </option>
      </select>
    </div>
    <div v-if="isLoading">加載中...</div>
    <div v-else-if="error">錯誤：{{ error }}</div>
    <div v-else>
      <WeatherTable :forecasts="weatherForecasts" :itemsPerPage="10">
        <template #actions="{ forecast }">
          <button
            @click="toggleFavorite(forecast)"
            :class="isFavorite(forecast) ? 'remove-favorite' : 'add-favorite'"
          >
            <span class="button-text">
              {{ isFavorite(forecast) ? '移除最愛' : '新增最愛' }}
            </span>
            <i
              class="icon"
              :class="
                isFavorite(forecast) ? 'fas fa-heart-broken' : 'fas fa-heart'
              "
            ></i>
          </button>
        </template>
      </WeatherTable>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import WeatherTable from '@/components/WeatherTable.vue'

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

function toggleFavorite(forecast) {
  const index = favorites.value.findIndex(
    fav => fav.city === forecast.city && fav.startTime === forecast.startTime,
  )
  if (index === -1) {
    favorites.value.push(forecast)
  } else {
    favorites.value.splice(index, 1)
  }
  saveFavorites()
}

function isFavorite(forecast) {
  return favorites.value.some(
    fav => fav.city === forecast.city && fav.startTime === forecast.startTime,
  )
}

onMounted(() => {
  fetchWeather()
  loadFavorites()
})
</script>

<style lang="scss" scoped>
#city-select {
  max-width: 100%;
  margin-bottom: 10px;
}
</style>
