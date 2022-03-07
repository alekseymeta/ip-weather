<template>
  <div class="temp" v-if="weather.main">
    <img :src="'http://openweathermap.org/img/wn/'+ weather.weather[0].icon + '@2x.png'" alt="">
    <div class="d-flex">
      <div class="main_temp">
        <div>{{ weather.main.temp }}</div>
        <span class="celc_icon">°</span>
      </div>
      <div class="feel">Feels like: {{ weather.main.feels_like }}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Weather",
  data(){
    return {
      weather: []
    }
  },
  props: {
    loc: {
      type: String,
      require: true
    }
  },
  mounted() {
    fetch('http://dich.tech/weather/api.php?loc='+this.loc)
        .then(resp => resp.json())
        .then(data => {
          this.weather = data
          console.log(data)
        })
  }
}
</script>

<style scoped>
.temp {
  display: flex;
  align-items: center;
}

img {
  width: 50px;
  margin-right: 10px;
  margin-left: -10px;
}

.celc_icon {
  color: #9999;
  font-weight: 100;
}

.main_temp {
  display: flex;
}

.feel {
  font-size: 10px;
  color: #9999;
  font-weight: 100;
}
</style>