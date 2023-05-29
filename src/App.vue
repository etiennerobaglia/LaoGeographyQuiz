<template>
  <main class="banlaoyusai">
    
    <div class="header">

      <div class="title-container">
        <div class="title-logo"></div>
        <div class="title-text">ບ້ານ ລາວ ຢູ່ໃສ<br>Ban Lao Yu Sai</div>
      </div>

      <div id="game-modal-info"></div>
      
    </div>
    

    <div class="map-container">
      <div id="map"></div>
    </div>

    <div class="game-container">
      <div class="game">

        <div class="game-title-container">
          <h1 class="game-title">
            <span class="game-title-lao">
              ບ້ານ<br>ລາວ ຢູ່ໃສ
            </span>
            <br>
            <span class="game-title-english">
              Ban&nbsp;Lao<br>
              Yu&nbsp;&nbsp;Sai
            </span>
          </h1>
        </div>

        <GameSelection :mapPromise="mapPromise" />

      </div>
    </div>
    

  </main>
  
</template>

<script>
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import { onMounted, ref } from "vue";
import GameSelection from './components/GameSelection.vue';

export default {
  name: 'App',
  components: {
    GameSelection
  },
  setup() {
    const mapPromise = ref();
    
    onMounted(() => {
      mapboxgl.accessToken = import.meta.env.VITE_VUE_APP_MAPBOX_TOKEN;
      
      const map = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/etroba/clh1k3m1400l901qufshrdakc",
        bounds:[
          [97.92952232149673, 12.372529503144591], 
          [109.21012831452117, 23.604023167268167]
        ]
      });

      mapPromise.value = new Promise((resolve) => {
        map.on('load', () => resolve(map));
      });

    });
    return {
      mapPromise,
    };
  },
};
</script>

<style>
</style>