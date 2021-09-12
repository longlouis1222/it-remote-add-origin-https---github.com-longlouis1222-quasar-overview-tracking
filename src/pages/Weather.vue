<template>
  <q-page class="flex column weather-page" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn
            @keyup.enter="getWeatherBySearch"
            round
            dense
            flat
            icon="search"
          />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree"> &deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
          alt="icon"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar <br />
          Weather
        </div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location"> </q-icon>
          <div>Find my Location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline" :class="underDetails"></div>
  </q-page>
</template>

<script>
import axios from "axios";

export default {
  name: "WeatherPage",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: "77865ad22faf322b82f3f4970d463390",
    };
  },

  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      }
    },
    underDetails() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "underDetails-night";
        } else {
          return "underDetails-day";
        }
      } else {
        return "skyline";
      }
    },
  },

  methods: {
    getLocation() {
      this.$q.loading.show();

      if (this.$q.platform.is.electron) {
        axios.get("https://freegeoip.app/json/").then((response) => {
          this.lat = response.data.latitude;
          this.lon = response.data.longitude;
          this.getWeatherByCoords();
        });
      } else {
        navigator.geolocation.getCurrentPosition((position) => {
          console.log("position: ", position);
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;
          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      axios(
        `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`
      ).then((response) => {
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
      this.search = "";
    },
    getWeatherBySearch() {
      this.$q.loading.show();
      axios(`${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`)
        .then((response) => {
          this.weatherData = response.data;
          this.$q.loading.hide();
        })
        .catch((err) => {
          console.log(err);
          this.$q.notify({
            message: "Please enter a City !",
            type: "negative",
          });
          this.$q.loading.hide();
        });
      this.search = "";
    },
  },
};
</script>

<style lang="sass">
.q-page.weather-page
  background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #00b4db, #0083b0)
.degree
  top: -44px
.skyline
  flex: 0 0 100px
  background: url(../statics/dubai.png)
  background-size: contain
  background-position: center bottom
  transition: all 0.3s linear
  &.underDetails-night
    background: url(../statics/night1.png)
    background-size: contain
    background-position: center bottom
  &.underDetails-day
    background: url(../statics/skyline.png)
    background-size: contain
    background-position: center bottom
</style>
