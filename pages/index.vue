<template>
  <div
    class="d-flex justify-content-center align-items-center"
    style="height: 100vh; width: 100vw;background-image: url(https://4kwallpapers.com/images/wallpapers/clear-sky-sunset-dusk-blue-sky-starry-sky-horizon-beach-1920x1200-4044.jpg); background-size:cover;"
  >
      <div class="position-absolute">
      <b-form-input
        @click="visible = !visible"
        class="position-relative mb-2"
        v-model="city"
        @keyup.enter="search"
        placeholder="Enter city name"
      ></b-form-input>
      <b-collapse class="position-absolute" id="collapse-4" v-model="visible">
        <b-list-group
          style="min-width: 50vw"
          v-for="country in searchCountries"
          :key="country.id"
        >
          <b-list-group-item
            @focus="visible = !visible"
            @click="search((city = country.name))"
            button
            >{{ country.name }}</b-list-group-item
          >
        </b-list-group>
      </b-collapse>

      <b-card
        v-if="weatherData.weather"
        style="min-width: 50vw"
        class="position-static"
      >
      <div v-if="loading">
        <b-overlay :show="show" rounded="sm">
     
    </b-overlay>
      </div>
        <div v-else
          class="mb-2 d-flex flex-column justify-content-center align-items-center"
        >
          <div
            class="mb-2 d-flex justify-content-around align-items-center w-100"
          >
            <div class="d-flex flex-column align-items-center">
              <b-card-text>
                {{ date }}
              </b-card-text>
              <b-card-text>
                {{ time }}
              </b-card-text>
            </div>
            <div class="d-flex flex-column align-items-center">
              <b-card-text>
                <strong>
                  {{ weatherData.name }}
                </strong>
              </b-card-text>
              <b-card-text>
                <strong>
                  {{ weatherData.sys.country }}
                </strong>
              </b-card-text>
            </div>
            <div class="d-flex flex-column align-items-center">
              <img :src="icon" alt="weather icon" />
              <b-card-text>{{ weatherData.main.temp }}°C</b-card-text>
            </div>
          </div>
          <b-button class="px-4 ml-3" @click="open"  v-b-toggle.collapse-2>{{ isActive ? 'Open' : 'Close' }} </b-button>
          <div>
            <b-collapse id="collapse-2">
              <div
                class="mb-2 position-static d-flex justify-content-between align-items-center"
              >
                <b-list-group class="m-3" style="width: 300px">
                  <b-list-group-item variant="dark"
                    >Temperature</b-list-group-item
                  >
                  <b-list-group-item
                    >Feels Like:
                    {{ weatherData.main.feels_like }}°C</b-list-group-item
                  >
                  <b-list-group-item
                    >Maximum Temperature:
                    {{ weatherData.main.temp_max }}°C</b-list-group-item
                  >
                  <b-list-group-item
                    >Minimum Temperature:
                    {{ weatherData.main.temp_min }}°C</b-list-group-item
                  >
                </b-list-group>
                <b-list-group class="m-3" style="width: 300px">
                  <b-list-group-item variant="dark">Winds</b-list-group-item>
                  <b-list-group-item
                    >Wind Degree: {{ weatherData.wind.deg }}</b-list-group-item
                  >
                  <b-list-group-item
                    >Wind Speed:
                    {{ weatherData.wind.speed }}kmS</b-list-group-item
                  >
                </b-list-group>
                <b-list-group class="m-3" style="width: 300px">
                  <b-list-group-item variant="dark"
                    >Other Info</b-list-group-item
                  >
                  <b-list-group-item
                    >Humidity:
                    {{ weatherData.main.humidity }}°C</b-list-group-item
                  >
                  <b-list-group-item
                    >Pressure:
                    {{ weatherData.main.pressure }}hPa</b-list-group-item
                  >
                  <b-list-group-item
                    >Visibility :{{
                      weatherData.visibility
                    }}m</b-list-group-item
                  >
                </b-list-group>
              </div>
            </b-collapse>
          </div>
        </div>
      </b-card>
    </div>   
  </div>
</template>

<script>
import axios from "axios";
import { ref, computed } from "vue";
import countries from "../city.list.json";

export default {
  data() {
    return {
      loading: false,
      visible: false,
      slide: 0,
      sliding: null,
      date: "",
      time: "",
      weatherData: {},
    };
  },
  setup() {
    let city = ref("");
    let isActive =ref(true)
    const searchCountries = computed(() => {
      if (city.value === "") {
        return [];
      }

      let matches = 0;

      return countries.filter((country) => {
        if (
          country.name.toLowerCase().includes(city.value.toLowerCase()) &&
          matches < 10
        ) {
          matches++;
          return country;
        }
      });
    });

    return {
      searchCountries,
      city,
      countries,
      isActive
    };
  },
  created() {
    setInterval(this.getNow, 1000);
  },
  computed: {
    icon() {
      return this.weatherData.weather
        ? `http://openweathermap.org/img/wn/${this.weatherData.weather[0].icon}.png`
        : "";
    },
  },
  methods: {
    open() {
     this.isActive = !this.isActive;
    },
    getNow: function () {
      const today = new Date();
      const date =
        today.getDate() +
        "/" +
        (today.getMonth() + 1) +
        "/" +
        today.getFullYear()
      const time =
        today.getHours() + ":" + today.getMinutes() + ":" + today.getSeconds();
      this.time = time;
      this.date = date;
    },
    search() {
      this.loading = true;
      this.show = true;
      const api = axios
        .get(
          `https://api.openweathermap.org/data/2.5/weather?units=metric&q=${this.city}&appid=2d89a7a1f1995d23573db47fb068dd88`
        )
        .then((res) => {
          if (res) {
            this.weatherData = res.data;
            this.loading = false;
          }
        })
        .catch((err) => {
          alert("The city you were looking for was not found");
        })
        .then((this.city = ""));
      return { api };
    },
    location() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            const api = axios
              .get(
                `https://api.openweathermap.org/data/2.5/weather?units=metric&lat=${position.coords.latitude}&lon=${position.coords.longitude}&appid=2d89a7a1f1995d23573db47fb068dd88`
              )
              .then((res) => {
                if (res) {
                  this.weatherData = res.data;
                  this.loading = false;
                }
              }).catch((err) => {
                alert("The city you were looking for was not found");
                return { api };
              });
          }
        )}
    },
  },
  mounted() {
    this.location();
  },
};
</script>

