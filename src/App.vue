<template>
  <div :class="['weather-app', weatherBackground]">
    <!-- Search Bar -->
    <div class="search-bar">
      <input
        type="text"
        v-model="searchCity"
        placeholder="Enter a city"
        @keyup.enter="fetchWeather"
      />
      <button @click="fetchWeather">Search</button>
    </div>

    <!-- Current Weather Data -->
    <div v-if="weather" class="weather-info animated fadeIn">
      <div class="weather-card">
        <div class="weather-header">
          <h2>{{ weather.location.name }}, {{ weather.location.country }}</h2>
          <p>{{ formatDate(weather.location.localtime) }}</p>
        </div>
        <div class="weather-body">
          <!-- Weather Icon -->
          <div :class="['weather-icon', weatherIcon]" class="bounce"></div>
          <!-- Weather Details -->
          <div class="weather-details">
            <p class="temperature">{{ weather.current.temp_c }}°C</p>
            <p class="condition">{{ weather.current.condition.text }}</p>
            <p>Feels like: {{ weather.current.feelslike_c }}°C</p>
            <p>Humidity: {{ weather.current.humidity }}%</p>
            <p>Wind: {{ weather.current.wind_kph }} km/h</p>
            <p>UV Index: {{ weather.current.uv }}</p>
          </div>
        </div>
      </div>

      <!-- 3-Day Forecast -->
      <div class="forecast animated fadeIn">
        <h3>3-Day Forecast</h3>
        <div class="forecast-container">
          <div v-for="day in weather.forecast.forecastday" :key="day.date" class="forecast-day bounce">
            <p class="forecast-date">{{ formatDay(day.date) }}</p>
            <div :class="['forecast-icon', getWeatherIcon(day.day.condition.text)]"></div>
            <p class="forecast-temp">{{ day.day.maxtemp_c }}°C / {{ day.day.mintemp_c }}°C</p>
            <p class="forecast-condition">{{ day.day.condition.text }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Loading and Error States -->
    <div v-else-if="loading" class="loading animated fadeIn">
      <div class="loader"></div>
      <p>Loading weather data...</p>
    </div>
    <div v-else-if="errorMessage" class="error animated fadeIn">{{ errorMessage }}</div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      searchCity: 'Baghdad', // Default city
      weather: null,
      loading: false,
      errorMessage: '',
      weatherBackground: 'default',
    };
  },
  computed: {
    weatherIcon() {
      const condition = this.weather?.current.condition.text.toLowerCase();
      return this.getWeatherIcon(condition);
    },
  },
  methods: {
    async fetchWeather() {
      this.loading = true;
      this.errorMessage = '';
      const apiKey = 'bcbfe146d14a4dd99aa121316241309'; // Your WeatherAPI key
      const url = `https://api.weatherapi.com/v1/forecast.json?key=${apiKey}&q=${this.searchCity}&days=3`;

      try {
        const response = await axios.get(url);
        this.weather = response.data;
        this.updateBackground(this.weather.current.condition.text);
        this.loading = false;
      } catch (error) {
        this.errorMessage = 'City not found or an error occurred.';
        this.weather = null;
        this.loading = false;
      }
    },
    updateBackground(condition) {
      condition = condition.toLowerCase();
      if (condition.includes('clear') || condition.includes('sunny')) {
        this.weatherBackground = 'sunny';
      } else if (condition.includes('cloud')) {
        this.weatherBackground = 'cloudy';
      } else if (condition.includes('rain')) {
        this.weatherBackground = 'rainy';
      } else if (condition.includes('snow')) {
        this.weatherBackground = 'snowy';
      } else {
        this.weatherBackground = 'default';
      }
    },
    getWeatherIcon(condition) {
      condition = condition.toLowerCase();
      if (condition.includes('clear') || condition.includes('sunny')) {
        return 'sunny-icon';
      } else if (condition.includes('cloud')) {
        return 'cloudy-icon';
      } else if (condition.includes('rain')) {
        return 'rainy-icon';
      } else if (condition.includes('snow')) {
        return 'snowy-icon';
      } else {
        return 'default-icon';
      }
    },
    formatDate(dateString) {
      const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit' };
      return new Date(dateString).toLocaleDateString('en-US', options);
    },
    formatDay(dateString) {
      const options = { weekday: 'short', month: 'short', day: 'numeric' };
      return new Date(dateString).toLocaleDateString('en-US', options);
    },
  },
  mounted() {
    this.fetchWeather(); // Fetch default weather data on load
  },
};
</script>

<style scoped>
/* General Styles */
.weather-app {
  font-family: 'Roboto', sans-serif;
  min-height: 100vh;
  padding: 20px;
  text-align: center;
  background-color: #f0f0f0;
  transition: background-image 0.5s ease;
}

.search-bar {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

input {
  padding: 12px;
  width: 250px;
  border-radius: 25px 0 0 25px;
  border: none;
  outline: none;
  font-size: 16px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

button {
  padding: 12px 20px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 0 25px 25px 0;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s ease;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

button:hover {
  background-color: #45a049;
}

/* Weather Card */
.weather-card {
  background: rgba(255, 255, 255, 0.8);
  border-radius: 15px;
  padding: 30px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  margin-bottom: 30px;
  transition: all 0.3s ease;
}

.weather-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
}

.weather-header h2 {
  font-size: 2.5rem;
  margin-bottom: 10px;
  color: #333;
}

.weather-body {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
}

.weather-details {
  margin-left: 20px;
  text-align: left;
}

.temperature {
  font-size: 3.5rem;
  font-weight: bold;
  margin: 0;
  color: #333;
}

.condition {
  font-size: 1.5rem;
  margin: 5px 0 15px;
  color: #666;
}

/* Forecast Styles */
.forecast {
  background-color: rgba(255, 255, 255, 0.8);
  padding: 30px;
  border-radius: 15px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.forecast h3 {
  font-size: 1.8rem;
  margin-bottom: 20px;
  color: #333;
}

.forecast-container {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
}

.forecast-day {
  text-align: center;
  margin: 10px;
  padding: 15px;
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 10px;
  transition: all 0.3s ease;
}

.forecast-day:hover {
  transform: scale(1.05);
  background-color: rgba(255, 255, 255, 0.8);
}

.forecast-date {
  font-weight: bold;
  margin-bottom: 10px;
}

.forecast-temp {
  font-size: 1.2rem;
  margin: 5px 0;
}

.forecast-condition {
  font-size: 0.9rem;
  color: #666;
}

/* Weather Icons */
.weather-icon, .forecast-icon {
  font-size: 6rem;
  display: inline-block;
  margin-right: 20px;
  width: 100px;
  height: 100px;
}

.sunny-icon {
  background: url('https://cdn-icons-png.flaticon.com/512/869/869869.png') no-repeat center;
  background-size: contain;
}

.cloudy-icon {
  background: url('https://cdn-icons-png.flaticon.com/512/414/414927.png') no-repeat center;
  background-size: contain;
}

.rainy-icon {
  background: url('https://cdn-icons-png.flaticon.com/512/414/414974.png') no-repeat center;
  background-size: contain;
}

.snowy-icon {
  background: url('https://cdn-icons-png.flaticon.com/512/642/642000.png') no-repeat center;
  background-size: contain;
}

.default-icon {
  background: url('https://cdn-icons-png.flaticon.com/512/1163/1163661.png') no-repeat center;
  background-size: contain;
}

/* Animations */
.animated {
  animation-duration: 1s;
  animation-fill-mode: both;
}

.fadeIn {
  animation-name: fadeIn;
}

.bounce {
  animation-name: bounce;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-20px); }
  60% { transform: translateY(-10px); }
}

/* Dynamic Backgrounds */
.sunny {
  background-image: url('https://images.unsplash.com/photo-1601297183305-6df142704ea2?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Mnx8c3Vubnl8ZW58MHx8MHx8fDA%3D&auto=format&fit=crop&w=1600&q=60');
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
}

.cloudy {
  background-image: url('https://images.unsplash.com/photo-1534088568595-a066f410bcda?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTd8fGNsb3VkeXxlbnwwfHwwfHx8MA%3D%3D&auto=format&fit=crop&w=1600&q=60');
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
}

.rainy {
  background-image: url('https://images.unsplash.com/photo-1519692933481-e162a57d6721?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTB8fHJhaW55fGVufDB8fDB8fHww&auto=format&fit=crop&w=1600&q=60');
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
}

.snowy {
  background-image: url('https://images.unsplash.com/photo-1516431883744-f077ead1002b?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MjB8fHNub3d5fGVufDB8fDB8fHww&auto=format&fit=crop&w=1600&q=60');
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
}

.default {
  background-color: #f0f0f0;
}

/* Loading and Error Messages */
.loading, .error {
  font-size: 1.5rem;
  margin-top: 20px;
  color: #333;
}

.loader {
  border: 5px solid #f3f3f3;
  border-top: 5px solid #3498db;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
  margin: 20px auto;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Responsive Design */
@media (max-width: 768px) {
  .weather-body {
    flex-direction: column;
    align-items: center;
  }

  .weather-details {
    margin-left: 0;
    margin-top: 20px;
    text-align: center;
  }

  .forecast-container {
    flex-direction: column;
  }

  .forecast-day {
    width: 100%;
    margin: 10px 0;
  }
}
</style>