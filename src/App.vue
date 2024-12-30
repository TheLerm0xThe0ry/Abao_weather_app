<template>
  <v-app :class="weatherBackground">
    <v-container>
      <v-row justify="center">
        <v-col cols="12" md="8">
          <v-card class="pa-4">
            <v-card-title class="text-center">
              <h2 class="text-h5">🌤️ Weather App</h2>
            </v-card-title>
            <v-card-text>
              <v-text-field
                v-model="city"
                label="Enter City"
                dense
                outlined
                @keyup.enter="addCity"
              ></v-text-field>
              <v-btn color="primary" block @click="addCity">Add City</v-btn>
              <v-alert v-if="error" type="error" class="mt-2">{{ error }}</v-alert>
              <v-row class="mt-4" v-if="weatherData.length">
                <v-col
                  v-for="(weather, index) in weatherData"
                  :key="index"
                  cols="12"
                  md="6"
                >
                  <v-card outlined>
                    <v-card-title class="text-h6">
                      <v-icon left color="blue">mdi-city</v-icon>
                      {{ weather.name }}
                    </v-card-title>
                    <v-card-subtitle>
                      <v-icon left>mdi-weather-cloudy</v-icon>
                      {{ weather.weather[0].description }}
                    </v-card-subtitle>
                    <v-card-text>
                      <p>
                        <v-icon left>mdi-thermometer</v-icon>
                        <strong>Temperature:</strong> {{ weather.main.temp }} °C
                      </p>
                      <p>
                        <v-icon left>mdi-water-percent</v-icon>
                        <strong>Humidity:</strong> {{ weather.main.humidity }}%
                      </p>
                      <p>
                        <v-icon left>mdi-weather-windy</v-icon>
                        <strong>Wind Speed:</strong> {{ weather.wind.speed }} m/s
                      </p>
                      <p>
                        <v-icon left>mdi-weather-sunset-up</v-icon>
                        <strong>Sunrise:</strong> {{ formatTime(weather.sys.sunrise) }}
                      </p>
                      <p>
                        <v-icon left>mdi-weather-sunset-down</v-icon>
                        <strong>Sunset:</strong> {{ formatTime(weather.sys.sunset) }}
                      </p>
                      <p>
                        <v-icon left>mdi-gauge</v-icon>
                        <strong>Pressure:</strong> {{ weather.main.pressure }} hPa
                      </p>
                    </v-card-text>
                    <v-btn text color="red" @click="removeCity(index)">
                      Remove
                    </v-btn>
                  </v-card>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      city: '',
      weatherData: [],
      error: '',
    };
  },
  computed: {
    weatherBackground() {
      if (!this.weatherData.length) {
        return "bg-clear"; // Default background
      }

      const currentWeather = this.weatherData[0]?.weather[0]?.main.toLowerCase();
      switch (currentWeather) {
        case "clear":
          return "bg-clear";
        case "clouds":
          return "bg-cloudy";
        case "rain":
          return "bg-rainy";
        case "snow":
          return "bg-snowy";
        case "thunderstorm":
          return "bg-thunderstorm";
        default:
          return "bg-misc";
      }
    },
  },
  methods: {
    async addCity() {
      if (this.city.trim() === "") {
        this.error = "Please enter a city name.";
        return;
      }
      try {
        const apiKey = "84079a722d513c7eb68ab420a1c17618"; // Replace with your OpenWeatherMap API key
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=${apiKey}`
        );
        const data = await response.json();
        if (data.cod === 200) {
          this.weatherData.push(data);
          this.error = "";
          this.city = "";
        } else {
          this.error = "City not found.";
        }
      } catch (error) {
        this.error = "Unable to fetch weather data.";
      }
    },
    removeCity(index) {
      this.weatherData.splice(index, 1);
    },
    formatTime(unixTimestamp) {
      const date = new Date(unixTimestamp * 1000);
      return date.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
    },
  },
};
</script>

<style>
/* Default Background */
.bg-clear {
  background: linear-gradient(to bottom, #87ceeb, #ffffff); /* Clear sky */
}

.bg-cloudy {
  background: linear-gradient(to bottom, #d3d3d3, #ffffff); /* Overcast */
}

.bg-rainy {
  background: linear-gradient(to bottom, #4f6d7a, #c4dfe6); /* Rainy day */
}

.bg-snowy {
  background: linear-gradient(to bottom, #e0f7fa, #ffffff); /* Snowy */
}

.bg-thunderstorm {
  background: linear-gradient(to bottom, #232526, #414345); /* Thunderstorm */
}

.bg-misc {
  background: linear-gradient(to bottom, #b2bec3, #ffffff); /* Miscellaneous */
}

body {
  font-family: "Roboto", sans-serif;
  margin: 0;
  padding: 0;
}

.v-card-title,
.v-card-text p {
  margin-bottom: 0.5rem;
}
</style>
