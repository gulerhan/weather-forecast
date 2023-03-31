<template>
  <div>
    <div class="mx-auto max-w-4xl py-12 sm:py-48 lg:py-12 text-center">
      <h1 class="text-4xl tracking-tight text-gray-900 sm:text-5xl">
        Hava Durumu
      </h1>
    </div>
    <div class="flex flex-row items-center justify-center">
      <v-col xs="6" sm="3" lg="3">
        <div class="relative">
          <v-text-field
            v-model="search"
            label="Şehir ara"
            hide-details="auto"
          ></v-text-field>
          <v-card class="city-list">
            <v-list v-if="showList">
              <v-list-item
                v-for="(item, i) in cities"
                :key="i"
                :value="item"
                @click="handleCity(item)"
              >
                {{ item.name }}
                {{ item.state ? "/ " + item.state : "/ " + item.country }}
              </v-list-item>
            </v-list>
          </v-card>
        </div>
      </v-col>
    </div>
    <div>
      <h1
        class="text-4xl tracking-tight text-gray-900 sm:text-3xl text-center pt-6"
        v-if="cityWeather.name"
      >
        Bugün
      </h1>
      <h1
        class="text-4xl font-bold tracking-tight text-gray-900 sm:text-5xl text-center pt-4 pb-4"
        v-if="cityWeather.name"
      >
        {{
          cityWeather.name + "," + Math.round(cityWeather.main.temp) + "&deg;C"
        }}
      </h1>
      <p
        class="tracking-tight text-gray-900 text-4xl flex justify-center pb-8 capitalize"
        v-if="cityWeather.weather"
      >
        {{ cityWeather.weather[0].description }}
      </p>
    </div>

    <div class="sm:flex justify-around px-12" id="cards">
      <template v-for="(weather, i) in anotherDays.list">
        <v-card v-if="i % 8 == 0" class="mt-4">
          <v-card-item>
            <h1 class="flex flex-row justify-center font-bold text-2xl">
              {{ weather.day }}
            </h1>
          </v-card-item>

          <v-card-text>
            <v-row align="center" no-gutters class="justify-around mb-4">
              <v-col class="text-h3 md:text-h2" cols="4">
                {{ Math.round(weather.main.temp) }}&deg;C
              </v-col>
              <div class="flex flex-col align-center">
                <img
                  :src="`${
                    'https://openweathermap.org/img/wn/' +
                    weather.weather[0].icon +
                    '@2x.png'
                  }`"
                  :alt="weather.weather[0].description"
                  class="flex"
                />
                <p class="capitalize">{{ weather.weather[0].description }}</p>
              </div>
            </v-row>
            <div class="flex justify-around pt-4">
              <p class="font-medium">
                Hissedilen:{{ Math.round(weather.main.feels_like) }}&deg;C
              </p>
              <p class="font-medium">Nem:{{ weather.main.humidity }}%</p>
            </div>
          </v-card-text>
        </v-card>
      </template>
    </div>
  </div>
</template>
<style lang="css">
.city-list {
  position: absolute !important;
  width: 100%;
  z-index: 1;
}
@media screen and (max-width: 950px) {
  body {
    background-repeat: repeat;
  }
  #cards {
    display: block;
  }
}
</style>

<script>
import axios from "axios";
export default {
  name: "CreatePage",

  data: () => ({
    geoApiUrl: "http://api.openweathermap.org/geo/1.0/direct",
    weatherApiUrl: "https://api.openweathermap.org/data/2.5/weather",
    forecastApiUrl: "https://api.openweathermap.org/data/2.5/forecast",
    apiKey: "b7543fa6fabc1f81d487a9a1a98e64bb",
    weatherApiKey: "9bb9b4143c23bf61bd6842dba3e4b125",
    search: "",
    cities: [],
    cityWeather: {},
    showList: false,
    anotherDays: {},
    days: [
      "Pazar",
      "Pazartesi",
      "Salı",
      "Çarşamba",
      "Perşembe",
      "Cuma",
      "Cumartesi",
    ],
  }),
  watch: {
    search(location) {
      if (location.length < 3) return;
      axios
        .get(`${this.geoApiUrl}?q=${location}&limit=5&appid=${this.apiKey}`)
        .then((res) => {
          this.cities = res.data;
          this.showList = true;
        })
        .finally(() => (this.isLoading = false));
    },
    model(val) {
      this.name = val.Title;
      this.image = val.Poster;
    },
  },
  methods: {
    handleCity(city) {
      const lat = city.lat;
      const lon = city.lon;
      axios
        .get(
          `${this.weatherApiUrl}?lat=${lat}&lon=${lon}&appid=${this.weatherApiKey}&units=metric&lang=tr`
        )
        .then((res) => {
          this.cityWeather = res.data;
          this.showList = false;
        });
      axios
        .get(
          `${this.forecastApiUrl}?lat=${lat}&lon=${lon}&appid=${this.weatherApiKey}&units=metric&lang=tr`
        )
        .then((res) => {
          this.anotherDays = res.data;
          let today = new Date();
          today.setDate(today.getDate() + 1);
          today = Math.floor(today / 1000);
          let afterDays = [];
          this.anotherDays.list.forEach((element) => {
            if (element.dt > today) {
              const day = new Date(element.dt_txt).getDay();
              element.day = this.days[day];
              afterDays.push(element);
            }
          });

          this.anotherDays.list = afterDays;
        });
    },
  },
};
</script>
