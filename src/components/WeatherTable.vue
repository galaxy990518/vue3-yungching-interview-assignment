<template>
  <div>
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
            v-for="forecast in paginatedForecasts"
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
              <slot name="actions" :forecast="forecast"></slot>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="weather-cards">
      <div
        v-for="forecast in paginatedForecasts"
        :key="forecast.startTime + forecast.city"
        class="weather-card"
      >
        <div class="card-row">縣市：{{ forecast.city }}</div>
        <div class="card-row">
          預報時間：{{ formatDate(forecast.startTime, true) }} -
          {{ formatDate(forecast.endTime, true) }}
        </div>
        <div class="card-row">天氣狀況：{{ forecast.weather }}</div>
        <div class="card-row">降雨機率：{{ forecast.pop }}%</div>
        <div class="card-row">
          溫度：{{ forecast.minTemp }}°C - {{ forecast.maxTemp }}°C
        </div>
        <div class="card-row">舒適度：{{ forecast.comfort }}</div>
        <div class="card-actions">
          <slot name="actions" :forecast="forecast"></slot>
        </div>
      </div>
    </div>
    <div class="pagination">
      <button @click="prevPage" :disabled="currentPage === 1">上一頁</button>
      <span>第 {{ currentPage }} 頁，共 {{ totalPages }} 頁</span>
      <button @click="nextPage" :disabled="currentPage === totalPages">
        下一頁
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  forecasts: {
    type: Array,
    required: true,
  },
  itemsPerPage: {
    type: Number,
    default: 10,
  },
})

const currentPage = ref(1)

const paginatedForecasts = computed(() => {
  const start = (currentPage.value - 1) * props.itemsPerPage
  const end = start + props.itemsPerPage
  return props.forecasts.slice(start, end)
})

const totalPages = computed(() => {
  return Math.ceil(props.forecasts.length / props.itemsPerPage)
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

function formatDate(dateString, isMobile = false) {
  const date = new Date(dateString)
  const options = {
    hour: '2-digit',
    minute: '2-digit',
    hour12: false,
  }

  if (!isMobile) {
    options.year = 'numeric'
    options.month = '2-digit'
    options.day = '2-digit'
  } else {
    options.month = '2-digit'
    options.day = '2-digit'
  }

  return date
    .toLocaleString('zh-TW', options)
    .replace(/\//g, '/')
    .replace(',', '')
}
</script>

<style scoped lang="scss">
@use '@/styles/main.scss' as *;

.weather-list {
  table {
    width: 100%;
    border-collapse: collapse;

    @media (max-width: $mobile-breakpoint) {
      display: none;
    }

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
}

.weather-cards {
  display: none;

  @media (max-width: $mobile-breakpoint) {
    display: block;
  }

  .weather-card {
    background-color: #f9f9f9;
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 15px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 15px;

    .card-row {
      margin-bottom: 8px;
      line-height: 1.5;
    }

    .card-actions {
      display: flex;
      justify-content: flex-end;
      margin-top: 10px;
    }
  }
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;

  button {
    margin: 0 10px;
    padding: 5px 10px;
    background-color: $primary-color;
    color: white;
    border: none;
    cursor: pointer;

    &:disabled {
      background-color: #cccccc;
      cursor: not-allowed;
    }
  }

  span {
    margin: 0 10px;
  }
}
</style>
