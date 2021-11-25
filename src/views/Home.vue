<template>
  <div class="home">
    <header>Weather forcast</header>
    <main class="weather-block">
      <div v-show="loading">loading...</div>
      <div class="my-20" v-show="!loading">
        <div class="row justify-content-center">
          <!-- today -->
          <div class="text-center" v-if="todayWeather.id">
            <div class="today-weather-day-img">
              <img
                :src="`${baseUrl}static/img/weather/${todayWeather.weather_state_abbr}.svg`"
                :alt="todayWeather.applicable_date"
              />
            </div>
            <p>{{ todayWeather.weather_state_name }}</p>
          </div>
        </div>
        <div class="row">
          <weather-day
            v-for="weather in showWeather"
            :key="weather.id"
            :info="weather"
            @setShowModal="setShowModal"
          />
        </div>
      </div>
    </main>
    <footer>
      <input
        placeholder="e.g. Taipei, Tokyo, London"
        name="location"
        type="text"
        class="search-input"
        v-model="location"
      />
      <button class="search-button" @click="searchLocation" :disabled="loading">
        {{ "search" }}
      </button>
    </footer>
  </div>
</template>

<script>
import WeatherDay from "@/components/WeatherDay.vue";
import axios from "axios";
export default {
  name: "Home",
  components: {
    WeatherDay,
  },
  data() {
    return {
      location: "Taipei",
      weatherInfo: {},
      loading: false,
      showModal: false,
    };
  },
  computed: {
    baseUrl() {
      return "https://www.metaweather.com/";
    },
    todayWeather() {
      return this.showWeather[0] || {};
    },
    showWeather() {
      return this.weatherInfo.consolidated_weather
        ? this.weatherInfo.consolidated_weather.filter(
            (weather, index) => index < 5
          )
        : [];
    },
  },
  created() {
    this.searchLocation();
  },
  methods: {
    setShowModal(val) {
      this.showModal = val;
      if (val) document.body.style.overflow = "hidden";
      else document.body.style.overflow = "inherit";
    },
    searchLocation() {
      this.loading = true;
      axios
        .get(
          `https://www.metaweather.com/api/location/search/?query=${this.location.toLowerCase()}`
        )
        .then((response) => {
          if (response.data.length) {
            this.getWeather(response.data[0]);
            return;
          }
          this.loading = false;
        })
        .catch(() => {
          this.loading = false;
        });
    },
    getWeather({ woeid }) {
      axios
        .get(`https://www.metaweather.com/api/location/${woeid}`)
        .then((response) => {
          this.weatherInfo = response.data;
          this.loading = false;
        })
        .catch(() => {
          this.loading = false;
        });
    },
  },
};
</script>
<style lang="scss"></style>
