<script setup>
import Form from './components/Form.vue'
import Weather from './components/Weather.vue'
import Error from './components/Error.vue'
import WeatherNextHours from './components/WeatherNextHours.vue';
</script>

<template>
  <main class="w-screen h-screen flex flex-col bg-[url('/src/assets/wallpaper.jpg')] bg-cover bg-center">
      <div class="m-auto" style="max-width: 90vw;">
        <img alt="App Weather logo" class="mx-auto mb-4 w-4/5 max-w-xs" src="./assets/logo.png"/>
        <div class="bg-white bg-opacity-80 p-6 rounded-xl shadow-2xl m-auto">
          <Form @handleLocalization="loadWeather" :city="city"/>
          <Error v-if="error" :value="error"/>
          <Weather v-if="weather" :weather="weather"/>
          <WeatherNextHours v-if="weathersNextHours" :weathers="weathersNextHours"/>
        </div>
      </div>
  </main>
</template>

<script>
export default {
  data() {
    return {
      localization: null,
      weather: null,
      weathersNextHours: null,
      apiKey: '1c6a5a86ae525b50937740738eaf5292',
      error: null,
    }
  },
  mounted() {
    this.getUserLocation()
  },
  methods: {
    loadWeather(city) {
      this.getGeographicalCoordinates(city)
          .then((localization) => {
            this.getWeather(localization.lat, localization.lon)
            this.getWeathersNextHours(localization.lat, localization.lon)
          })
          .catch((err) => {
            this.error = err
            this.weather = null
          })
    },
    getWeather(lat, lon) {
      const url = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${this.apiKey}`
      fetch(url)
          .then((response) => response.json())
          .then((data) => {
            this.weather = data
          })
    },
    getWeathersNextHours(lat, lon) {
      const url = `https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&appid=${this.apiKey}`
      fetch(url)
          .then((response) => response.json())
          .then((data) => {
            this.weathersNextHours = data.list
          })
    },
    getGeographicalCoordinates(city) {
      return new Promise((resolve, reject) => {
        const url = `https://api.openweathermap.org/geo/1.0/direct?q=${city}&appid=${this.apiKey}`
        fetch(url)
            .then((response) => response.json())
            .then((data) => {
              if (data.length) {
                resolve(data[0])
              }
              reject('The city you are looking for cannot be found')
            })
            .catch(() => reject('An error has occurred'))
      })
    },
    getUserLocation() {
      navigator.geolocation.getCurrentPosition((location) => {
        if (location) {
          this.getWeather(location.coords.latitude, location.coords.longitude)
          this.getWeathersNextHours(location.coords.latitude, location.coords.longitude)
        }
      })
    }

  },
  computed: {
    city() {
      return this.weather ? this.weather.name : null
    }
  },

  watch: {
    error() {
      setTimeout(() => {
        this.error = null
      }, 3000)
    }
  }
}
</script>
