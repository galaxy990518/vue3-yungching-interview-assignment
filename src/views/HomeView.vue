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
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="forecast in weatherForecasts"
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
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

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

function formatDate(dateString) {
  const date = new Date(dateString)
  return date.toLocaleString('zh-TW')
}

onMounted(() => {
  fetchWeather()
})
</script>

<style lang="scss" scoped>
table {
  width: 100%;
  border-collapse: collapse;

  th,
  td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }

  th {
    background-color: #f2f2f2;
  }
}
</style>
