<template>
  <div class="weather-day-block">
    <!-- block -->
    <div
      class="weather-day-cell"
      :ref="`weather-${info.id}`"
      @click="showModal = true"
    >
      <h4>{{ dateFormat(info.applicable_date, "YYYY/MM/DD") }}</h4>
      <div class="weather-day-img">
        <img
          :src="`${baseUrl}static/img/weather/${info.weather_state_abbr}.svg`"
          :alt="info.applicable_date"
        />
      </div>
      <p>{{ info.weather_state_name }}</p>

      <!-- {{ info }} -->
    </div>
    <!-- modal -->
    <div
      class="weather-day-modal"
      v-show="showModal"
      @click.capture="showModal = false"
    >
      <span class="modal-cross">×</span>
      <div class="text-center h-100 scroll pb-2">
        <h4>{{ dateFormat(info.applicable_date, "YYYY/MM/DD") }}</h4>
        <div class="weather-day-modal-img">
          <img
            :src="`${baseUrl}static/img/weather/${info.weather_state_abbr}.svg`"
            :alt="info.applicable_date"
          />
        </div>
        <p>{{ info.weather_state_name }}</p>
        <div class="row justify-content-center align-items-center w-100">
          <div class="chart-panel">
            <canvas
              height="300"
              class="weather-day-humidity"
              :ref="`humidity-${info.id}`"
            ></canvas>
          </div>
          <div class="chart-panel">
            <canvas
              :height="temperatureHeight"
              class="weather-day-temperature"
              :ref="`temperature-${info.id}`"
            ></canvas>
          </div>
          <table class="weather-day-table">
            <tbody>
              <tr>
                <td><b>Visibility</b></td>
                <td>{{ info.visibility.toFixed(2) }}miles</td>
              </tr>
              <tr>
                <td><b>Pressure</b></td>
                <td>{{ info.air_pressure }}mb</td>
              </tr>
              <tr>
                <td><b>Speed</b></td>
                <td>
                  {{ info.wind_speed.toFixed(2) }}mph
                  <span v-if="info.wind_direction_compass === 'N'">↑</span>
                  <span v-if="info.wind_direction_compass === 'S'">↓</span>
                  <span v-if="info.wind_direction_compass === 'E'">→</span>
                  <span v-if="info.wind_direction_compass === 'W'">←</span>
                  <span v-if="info.wind_direction_compass.indexOf('NE') > -1"
                    >↗</span
                  >
                  <span v-if="info.wind_direction_compass.indexOf('NW') > -1"
                    >↖</span
                  >
                  <span v-if="info.wind_direction_compass.indexOf('SE') > -1"
                    >↘</span
                  >
                  <span v-if="info.wind_direction_compass.indexOf('SW') > -1"
                    >↙</span
                  >
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import dayjs from "dayjs";
export default {
  name: "WeatherDay",
  props: {
    info: {
      default: function () {
        return {};
      },
      type: Object,
    },
  },
  data() {
    return {
      temperatureHeight: 150,
      temperatureMutiple: 3, // 顯示倍數
      showModal: false,
    };
  },
  mounted() {
    this.drawHumidityChart();
    this.drawTemperatureChart();
  },
  computed: {
    baseUrl() {
      return "https://www.metaweather.com/";
    },
  },
  methods: {
    dateFormat(val, format) {
      return dayjs(val).format(format);
    },
    drawHumidityChart() {
      const { id, humidity } = this.info;
      const canvas = this.$refs[`humidity-${id}`];
      const ctx = canvas.getContext("2d");
      const beginDeg = (-90 * Math.PI) / 180;
      const humidityDeg = (((humidity / 100) * 360) / 360) * Math.PI;

      // 畫出圖案
      const x = 150;
      const y = 150;
      const r = 100;

      // 背景色
      ctx.beginPath();
      ctx.moveTo(x, y);
      ctx.arc(x, y, r, 0, Math.PI * 2);
      ctx.fillStyle = "#bebdbd";
      ctx.fill();

      // 濕度色
      ctx.beginPath();
      ctx.moveTo(x, y);
      ctx.arc(x, y, r, beginDeg, humidityDeg);
      ctx.fillStyle = "#00697d";
      ctx.fill();

      // 文字
      ctx.font = "14px Arial";
      const text = `${humidity}%`;
      const title = `Humidity`;
      ctx.fillStyle = "#fff";
      ctx.fillText(text, x + 20, y + 20);
      ctx.fillStyle = "#333";
      ctx.fillText(title, x - 30, 30);
    },
    drawTemperatureChart() {
      const x = 150;
      const { id, min_temp, max_temp } = this.info;
      const minTempH =
        (min_temp / this.temperatureHeight) * 100 * this.temperatureMutiple;
      const maxTempH =
        (max_temp / this.temperatureHeight) * 100 * this.temperatureMutiple;
      const canvas = this.$refs[`temperature-${id}`];
      const ctx = canvas.getContext("2d");

      // 最高溫度
      ctx.beginPath();
      ctx.fillStyle = "#00697d";
      ctx.fillRect(x - 50, this.temperatureHeight - minTempH, 40, minTempH);

      // 最低溫度
      ctx.beginPath();
      ctx.fillStyle = "#cc375a";
      ctx.fillRect(x + 20, this.temperatureHeight - maxTempH, 40, maxTempH);

      // 文字
      ctx.font = "14px Arial";
      ctx.fillStyle = "#333";
      const MaxText = `${max_temp.toFixed(0)}°C`;
      const MinText = `${min_temp.toFixed(0)}°C`;
      const title = `Temperature`;
      ctx.fillText(MaxText, x + 25, this.temperatureHeight - maxTempH - 10);
      ctx.fillText(MinText, x - 45, this.temperatureHeight - minTempH - 10);
      ctx.fillText(title, x - 30, 30);
    },
  },
  watch: {
    showModal(to) {
      this.$emit("setShowModal", to);
    },
  },
};
</script>
