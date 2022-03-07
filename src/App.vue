<template>
  <div class="wrapper">
    <div class="main_container" v-if="ipinfo.length !== 0">
        <MapBoxPulsingIp :zoom="zoom"/>
        <div class="content">
          <Weather v-if="ipinfo.length !== 0" :loc="ipinfo.loc"/>
          <div><b>We know you're in <br> {{ipinfo.city}}</b></div>
          <div class="timezone">{{ ipinfo.timezone }}</div>
          <small class="loc">{{ ipinfo.loc }}</small>
        </div>
    </div>
    <div v-else>
      <Spinner/>
    </div>
  </div>

</template>

<script>
import MapBoxPulsingIp from "@/components/MapBoxPulsingIp";
import Weather from "@/components/Weather";
import Spinner from "@/components/Spinner";

export default {
  name: 'App',
  components: {
    MapBoxPulsingIp, Weather, Spinner
  },
  data() {
    return {
      ipinfo: []
    }
  },
  mounted() {
    fetch('https://ipinfo.io?token=' + process.env.VUE_APP_IP_TOKEN)
        .then(resp => resp.json())
        .then(data => {
          this.ipinfo = data
          console.log(data)
        })
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans:wght@400;700&display=swap');

@media (min-device-width: 320px) and (max-device-width: 568px) {
  .main_container {
    width: 100% !important;
    height: 100vh !important;
    margin-top: 0 !important;
  }
}

body {
  background-color: #131313;
  color: white;
  font-family: 'Noto Sans', sans-serif;
  margin: 0;
}
.wrapper {
  display: flex;
  justify-content: center;
}
.main_container {
  width: 350px;
  height: 650px;
  border-radius: 15px;
  position: relative;
  overflow: hidden;
  margin-top: 50px;
}
.mapboxgl-ctrl-bottom-right {
  display: none;
}


.content {
  z-index: 50;
  position: absolute;
  bottom: 20px;
  padding: 20px;
}

.temp {
  font-weight: bold;
  font-size: 35px;
  margin-bottom: 15px;
  display: flex;
}

.loc {
  color: #797979;
  font-size: 10px;
}

.timezone {
  margin-top: 20px;
}

.celc_icon {
  margin-left: 10px;
}

.btn_container {
  position: absolute;
  z-index: 500;
  top: 20px;
  right: 20px;
}

</style>
