<script setup lang="ts">
import { reactive, ref } from 'vue'

// ! Renseigner la clef d'api (problème avec le process.env)
const apiKey = ''

type Weather = {
  city: {
    name: string
  }
  icon: string
  temp: number
  humidity: number
  title: string
  description: string
  isTemplate: boolean
}

const template: Weather = {
  city: {
    name: ''
  },
  icon: '',
  temp: 0,
  humidity: 0,
  title: '',
  description: '',
  isTemplate: true
}

const weather = reactive<Weather>(template)
const searchInput = ref('')
const status = ref('')

const onSearch = async () => {
  try {
    const weatherRaw: Record<any, any> = await fetch(
      `http://api.openweathermap.org/data/2.5/weather?q=${searchInput.value}&units=metric&APPID=${apiKey}`
    ).then((r) => {
      return r.json()
    })
    status.value = weatherRaw.cod

    searchInput.value = ''

    if (status.value === '404') return null

    weather.city.name = weatherRaw.name
    weather.title = weatherRaw.weather[0].main
    weather.description = weatherRaw.weather[0].description
    weather.humidity = weatherRaw.main.humidity
    weather.temp = Math.ceil(weatherRaw.main.temp)
    weather.icon = `http://openweathermap.org/img/w/${weatherRaw.weather[0].icon}.png`
    weather.isTemplate = false
  } catch (error) {
    console.log('error:', error)
  }
}
</script>

<template>
  <main>
    <div v-if="status === '404'">
      <h2>La ville n'a pas été trouvée !</h2>
    </div>
    <div v-else-if="!weather.isTemplate" class="top">
      <h1>{{ weather.city.name }}</h1>
      <img alt="icon weather" class="logo" :src="weather.icon" width="125" height="125" />
      <h1>{{ weather.temp }}°</h1>
      <h2>{{ weather.title }}</h2>
      <h3>{{ weather.humidity }} % d'humiditée</h3>
    </div>

    <div v-else>
      <h2>Commencer par chercher une ville !</h2>
    </div>

    <div class="bottom">
      <input @keyup.enter="onSearch" v-model="searchInput" placeholder="Chercher une ville" />
      <button @click="onSearch">Rechercher</button>
    </div>
  </main>
</template>

<style scoped>
main {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 2rem;

  height: 100vh;
}
.top {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.bottom {
  display: flex;
  direction: row;
  justify-content: center;
}
</style>
