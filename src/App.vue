<template>
  <div id="map"></div>
  
  <GameSelection
    :mapPromise="mapPromise"
  />
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
    
    function randomVillageSelection(source, number) {
      return source.sort(() => .5 - Math.random()).slice(0,number) 
    }
    
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
#map {
  width: 100%;
  height: 100%;
}
.game {
  background-color: #2f2f2f;
  color: white;
  position: fixed;
  padding: .5rem;
}

.game-info {
  left:auto;
  bottom:auto;
  right: 10px;
  top: 10px;
}


.guessing-feature {
  color: rgb(165, 60, 184);
  font-size: 1.05rem;
  font-weight: 700;
}
</style>