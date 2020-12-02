<template>
  <div>
    <div class="header">
      <div class="search-container">
        <div class="search-bar">
          <button @click="isOpen = !isOpen" class="search init">
            <i class="fas fa-search"></i>
          </button>
          <form>
            <input v-show="!isOpen" type="search" id="location-search" name="location-search"
              placeholder="Enter a city name..." v-model="userSearch">
            <input type="submit" @click="submit">
          </form>
        </div>
        <button @click="getRandomWeather" class="search random">
          <i class="fas fa-globe-americas"></i>
        </button>
      </div>
    </div>
    <div v-show="isError" class="content">
      <div class="content-box">
        <div class="location">
          {{ name }}
        </div>
        <div class="date">
          {{ today(new Date()) }}
        </div>
        <div class="mainTemp">
          {{ mainTemp }}
          <div class="metric">
            {{ metric }}
          </div>
        </div>
        <label class="switch">
          <input type="checkbox" v-model="checked" @change="convert">
          <span class="slider round"></span>
        </label>
        <div class="info-row">
          {{ maxTemp + String.fromCharCode(176)}} / {{ minTemp + String.fromCharCode(176) }} | Real Feel
          {{ feelsLike + String.fromCharCode(176) }}
        </div>
        <br>
        <div class="weather">
          {{ weatherConditions }}
          <br>
          <img v-bind:src="iconUrl" />
          <br>
          {{ weatherDescription }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import moment from 'moment'

  export default {
    name: 'Content',
    data() {
      return {
        weatherApi: process.env.VUE_APP_WEATHER_API,
        geolocationApi: process.env.VUE_APP_GEO_API,
        longitude: null,
        latitude: null,
        postalCode: '',
        state: '',
        gettingLocation: false,
        city: null,
        name: null,
        mainTemp: '',
        metric: String.fromCharCode(176) + 'F',
        checked: false,
        maxTemp: '',
        minTemp: '',
        feelsLike: '',
        weatherConditions: '',
        iconUrl: '',
        weatherDescription: '',
        isOpen: true,
        userSearch: '',
        isError: true
      }
    },
    created() {
      if (navigator.geolocation) {
        this.gettingLocation = true
        navigator.geolocation.getCurrentPosition(this.getPosition, this.error)
      } else {
        alert('No geolocation? What year and browser is this!?')
      }
    },
    methods: {
      getPosition(position) {
        this.longitude = position.coords.longitude
        this.latitude = position.coords.latitude
        this.getWeather()
      },
      error(err) {
        if (err.code == err.PERMISSION_DENIED) {
          console.log("you denied me :-(");
          this.getRandomWeather()
        } else {
          this.errmsg = err.message
        }
      },
      async getGeoLocation() {
        try {
          const reverseGeo = await fetch(
            `https://us1.locationiq.com/v1/reverse.php?key=${this.geolocationApi}&lat=${this.latitude}&lon=${this.longitude}&format=json`, {
              mode: 'cors'
            });
          const zipCode = await reverseGeo.json();
          this.postalCode = zipCode.address.postcode
          this.state = zipCode.address.state
        } catch (error) {
          console.error(error)
        }
      },
      async getWeatherData() {
        try {
          const response = await fetch(
            `http://api.openweathermap.org/data/2.5/weather?q=${this.userSearch}&zip=${this.postalCode}&units=imperial&appid=${this.weatherApi}`, {
              mode: 'cors'
            });
          const weatherData = await response.json();
          console.log(weatherData)
          this.city = weatherData.name
          this.mainTemp = Math.round(weatherData.main.temp)
          this.maxTemp = Math.round(weatherData.main.temp_max)
          this.minTemp = Math.round(weatherData.main.temp_min)
          this.feelsLike = Math.round(weatherData.main.feels_like)
          this.weatherConditions = weatherData.weather[0].main
          this.weatherDescription = weatherData.weather[0].description
          this.longitude = weatherData.coord.lon
          this.latitude = weatherData.coord.lat
          this.iconUrl = `http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@4x.png`
          this.isError = true
        } catch (error) {
          alert("Hey that isn't a city, try again!")
          this.isError = false
        }
      },
      async getWeather() {
        let geoData = await this.getGeoLocation()
        let weatherData = await this.getWeatherData()
        this.name = this.city + ', ' + this.state
        return geoData, weatherData
      },
      async getUserWeather() {
        let weatherData = await this.getWeatherData()
        let geoData = await this.getGeoLocation()
        this.name = this.city + ', ' + this.state
        return geoData, weatherData
      },
      today(date) {
        return moment(date).format('ddd' + ', ' + 'MMM' + ' ' + 'DD' + ' ' + 'h' + ':' + 'mm' + 'a')
      },
      getRandomWeather() {
        var randomZip = Math.floor(Math.random() * 90000) + 10000;
        this.checked = false
        this.postalCode = randomZip
        this.getUserWeather()
      },
      submit(e) {
        e.preventDefault()
        this.checked = false
        this.getUserWeather()
        this.userSearch = ''
        this.isOpen = !this.isOpen
      },
      convert() {
        if (this.checked == true) {
          this.metric = String.fromCharCode(176) + 'C'
          this.mainTemp = Math.round((this.mainTemp - 32) * 5 / 9)
          this.maxTemp = Math.round((this.maxTemp - 32) * 5 / 9)
          this.minTemp = Math.round((this.minTemp - 32) * 5 / 9)
          this.feelsLike = Math.round((this.feelsLike - 32) * 5 / 9)
        } else {
          this.metric = String.fromCharCode(176) + 'F'
          this.mainTemp = Math.round((this.mainTemp * 9 / 5) + 32)
          this.maxTemp = Math.round((this.maxTemp * 9 / 5) + 32)
          this.minTemp = Math.round((this.minTemp * 9 / 5) + 32)
          this.feelsLike = Math.round((this.feelsLike * 9 / 5) + 32)
        }
      }
    }
  }
</script>

<style scoped>
  .header {
    margin: 5px;
  }

  .search-container {
    display: flex;
    justify-content: space-between;
  }

  form {
    display: flex;
    width: 100%;
  }

  .search-bar {
    display: flex;
    width: 100%;
    margin-top: 5px;
  }

  .search {
    background: none;
    border: none;
    outline: none;
    cursor: pointer;
    margin: 10px;
  }

  .fas {
    color: black;
    font-size: 28px;
  }

  input[type=search] {
    font-size: 16px;
    width: 100%;
    padding-left: 10px;
    transition: width 0.4s ease-in-out;
  }

  input[type=submit] {
    display: none;
  }

  .content {
    margin: 8%;
    min-height: 80%;
    display: flex;
    justify-content: center;
  }

  .content-box {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .location {
    font-size: 40px;
    margin: 25px 25px 5px 25px;
    background: linear-gradient(to bottom, #66ccff 0%, #00ccff 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .date {
    font-size: 20px;
    margin: 0px 25px 25px 25px;
    background: linear-gradient(to bottom, #66ccff 0%, #00ccff 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .mainTemp {
    display: flex;
    font-size: 250px;
    background: linear-gradient(to bottom, #00ff99 0%, #33ccff 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .metric {
    font-size: 50px;
  }

  .info-row {
    font-size: 22px;
    margin: 0 25px 10px 25px;
    background: linear-gradient(to bottom, #66ccff 0%, #00ccff 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .weather {
    text-align: center;
    background: linear-gradient(to bottom, #66ccff 0%, #00ccff 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    font-size: 50px;
  }

  .switch {
    position: relative;
    display: inline-block;
    margin: 0 25px 25px 25px;
    width: 46px;
    height: 15px;
  }

  .switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #ccc;
    -webkit-transition: .4s;
    transition: .4s;
  }

  .slider:before {
    position: absolute;
    content: "";
    height: 12px;
    width: 12px;
    left: 4px;
    top: 1px;
    background-color: white;
    -webkit-transition: .4s;
    transition: .4s;
  }

  input:checked+.slider {
    background-color: #00ff99;
  }

  input:focus+.slider {
    box-shadow: 0 0 1px #00ff99;
  }

  input:checked+.slider:before {
    -webkit-transform: translateX(26px);
    -ms-transform: translateX(26px);
    transform: translateX(26px);
  }

  .slider.round {
    border-radius: 34px;
  }

  .slider.round:before {
    border-radius: 50%;
  }
</style>