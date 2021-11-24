<template>
  <div class="weather-day-block" :ref="`weather-${info.id}`">
    <h4>{{ dateFormat(info.applicable_date, "YYYY/MM/DD") }}</h4>
    <div class="weather-day-img">
      <img
        :src="`${baseUrl}static/img/weather/${info.weather_state_abbr}.svg`"
        :alt="info.applicable_date"
      />
    </div>
    <p>{{ info.weather_state_name }}</p>
    <canvas
      height="300"
      class="weather-day-humidity"
      :ref="`humidity-${info.id}`"
    ></canvas>
    <canvas
      :height="temperatureHeight"
      class="weather-day-temperature"
      :ref="`temperature-${info.id}`"
    ></canvas>
    <!-- {{ info }} -->
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
      ctx.fillStyle = "#333";
      ctx.fill();

      // 濕度色
      ctx.beginPath();
      ctx.moveTo(x, y);
      ctx.arc(x, y, r, beginDeg, humidityDeg);
      ctx.fillStyle = "yellow";
      ctx.fill();

      // 文字
      ctx.font = "14px Arial";
      ctx.fillStyle = "#333";
      const text = `${humidity}%`;
      const title = `Humidity`;
      ctx.fillText(text, x + 20, y + 20);
      ctx.fillText(title, x - 30, 30);
    },
    drawTemperatureChart() {
      const x = 150;
      const { id, min_temp, max_temp } = this.info;
      const minTempH = (min_temp / this.temperatureHeight) * 300;
      const maxTempH = (max_temp / this.temperatureHeight) * 300;
      const canvas = this.$refs[`temperature-${id}`];
      const ctx = canvas.getContext("2d");

      // 最高溫度
      ctx.beginPath();
      ctx.fillStyle = "blue";
      ctx.fillRect(x - 50, this.temperatureHeight - minTempH, 40, minTempH);

      // 最低溫度
      ctx.beginPath();
      ctx.fillStyle = "red";
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
};
</script>
