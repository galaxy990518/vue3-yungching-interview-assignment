<template>
  <div v-if="show" class="dialog-overlay"></div>
  <div v-if="show" class="edit-dialog">
    <h2>編輯天氣預報</h2>
    <div v-if="error" class="error-message">{{ error }}</div>
    <div class="input-group">
      <label>天氣狀況: </label>
      <div class="input-with-unit">
        <input
          v-model="localForecast.weather"
          @input="validateWeather"
          maxlength="20"
        />
      </div>
    </div>
    <div class="input-group">
      <label>降雨機率: </label>
      <div class="input-with-unit">
        <input
          v-model.number="localForecast.pop"
          @input="validatePop"
          type="number"
          min="0"
          max="100"
        />
        <span class="unit">%</span>
      </div>
    </div>
    <div class="input-group">
      <label>最低溫: </label>
      <div class="input-with-unit">
        <input
          v-model.number="localForecast.minTemp"
          @input="validateMinTemp"
          type="number"
        />
        <span class="unit">°C</span>
      </div>
    </div>
    <div class="input-group">
      <label>最高溫: </label>
      <div class="input-with-unit">
        <input
          v-model.number="localForecast.maxTemp"
          @input="validateMaxTemp"
          type="number"
        />
        <span class="unit">°C</span>
      </div>
    </div>
    <div class="input-group">
      <label>舒適度: </label>
      <div class="input-with-unit">
        <input
          v-model="localForecast.comfort"
          @input="validateComfort"
          maxlength="20"
        />
      </div>
    </div>
    <div class="button-group">
      <button
        @click="save"
        class="save"
        :class="{ disabled: hasValidationError }"
        :disabled="hasValidationError"
      >
        保存
      </button>
      <button @click="cancel" class="cancel">取消</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  show: Boolean,
  forecast: Object,
})

const emit = defineEmits(['update:show', 'save'])

const localForecast = ref({})
const error = ref('')

watch(
  () => props.forecast,
  newForecast => {
    localForecast.value = { ...newForecast }
  },
  { deep: true },
)

function validateWeather() {
  const weather = localForecast.value.weather
  if (!/^[\u4e00-\u9fa5]{1,20}$/.test(weather)) {
    error.value = '天氣狀況的描述最多20個中文字'
    return false
  } else {
    error.value = ''
    return true
  }
}

function validatePop() {
  const pop = parseFloat(localForecast.value.pop)
  if (isNaN(pop) || pop < 0 || pop > 100) {
    error.value = '降雨機率區間0 ~ 100之間的數字'
    return false
  } else {
    error.value = ''
    return true
  }
}

function validateMinTemp() {
  const temp = parseFloat(localForecast.value.minTemp)
  if (isNaN(temp) || temp < -200 || temp > 100) {
    error.value = '最低溫度為 -200 ~ 100之間的數字'
    return false
  } else {
    error.value = ''
    return true
  }
}

function validateMaxTemp() {
  const temp = parseFloat(localForecast.value.maxTemp)
  if (isNaN(temp) || temp < 0 || temp > 100) {
    error.value = '最高溫度為 0 ~ 100之間的數字'
    return false
  } else {
    error.value = ''
    return true
  }
}

function validateComfort() {
  const comfort = localForecast.value.comfort
  if (!/^[\u4e00-\u9fa5]{1,20}$/.test(comfort)) {
    error.value = '舒適度的描述最多20個中文字'
    return false
  } else {
    error.value = ''
    return true
  }
}

const hasValidationError = computed(() => {
  return (
    !validateMinTemp() ||
    !validateMaxTemp() ||
    !validatePop() ||
    !validateWeather() ||
    !validateComfort()
  )
})

function save() {
  if (hasValidationError.value) return
  emit('save', {
    ...localForecast.value,
    minTemp: parseFloat(localForecast.value.minTemp),
    maxTemp: parseFloat(localForecast.value.maxTemp),
    pop: parseFloat(localForecast.value.pop),
  })
  emit('update:show', false)
}

function cancel() {
  emit('update:show', false)
}
</script>

<style lang="scss" scoped>
.edit-dialog {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  padding: 24px;
  width: 90%;
  max-width: 500px;
  z-index: 1000;

  h2 {
    font-size: 24px;
    margin-bottom: 20px;
    color: #333;
  }

  .button-group {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;

    button {
      flex: 1;
    }
  }
}

// 對話框和表單相關的通用樣式
.dialog-overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 999;
  pointer-events: none;
}

.error-message {
  background-color: #ffebee;
  color: #d32f2f;
  padding: 10px;
  border-radius: $border-radius;
  margin-bottom: 15px;
}

.input-group {
  margin-bottom: 15px;

  label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
  }

  .input-with-unit {
    position: relative;
    display: flex;
    align-items: center;

    input {
      width: 100%;
      padding: 8px 30px 8px 12px;
      border: 1px solid #ddd;
      border-radius: $border-radius;
      font-size: 16px;

      &:focus {
        outline: none;
        border-color: #4a90e2;
      }

      &[type='number'] {
        &::-webkit-inner-spin-button,
        &::-webkit-outer-spin-button {
          -webkit-appearance: none;
          appearance: none;
          margin: 0;
        }
      }
    }

    .unit {
      position: absolute;
      right: 20px;
      color: #666;
      pointer-events: none;
    }
  }
}
</style>
