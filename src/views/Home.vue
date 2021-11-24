<template>
  <div class="home">
    Weather
    <div class="weather-block">
      <input
        placeholder="e.g. Taipei, Tokyo, London"
        name="location"
        type="text"
        v-model="location"
      />
      <button @click="searchLocation" :disabled="loading">
        {{ loading ? "loading..." : "search" }}
      </button>
      <div v-show="loading">loading...</div>
      <div class="row" v-show="!loading">
        <weather-day
          v-for="weather in showWeather"
          :key="weather.id"
          :info="weather"
        />
      </div>
    </div>
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
      location: "taipei",
      weatherInfo: {},
      loading: false,
    };
  },
  computed: {
    showWeather() {
      return this.weatherInfo.consolidated_weather
        ? this.weatherInfo.consolidated_weather.filter(
            (weather, index) => index < 5
          )
        : [];
    },
  },
  methods: {
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
          console.log(this.weatherInfo);
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
