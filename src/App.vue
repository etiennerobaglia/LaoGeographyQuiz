<template>
  <div id="map"></div>
  
  <EasyGameMode
    :mapPromise="mapPromise"
    :dataSet="vteLvl1"
  />

  <HardGameMode
    :mapPromise="mapPromise"
    :dataSet="vteLvl1"
  />

</template>

<script>
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import { onMounted, ref } from "vue";
import villagesT2 from './assets/layers/vientiane-capital-villages-t2.json'
import HardGameMode from './components/HardGameMode.vue';
import EasyGameMode from './components/EasyGameMode.vue';

export default {
  name: 'App',
  components: {
    HardGameMode,
    EasyGameMode,
  },
  setup() {
    let vteLvl1 = ref(villagesT2);
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
          [102.57813377330893, 17.927212142550417],
          [102.64211925985438, 17.98422754876998]
        ]
      });

      mapPromise.value = new Promise((resolve) => {
        map.on('load', () => resolve(map));
      });

      mapPromise.value.then((map) => {
        map.addSource("vteLvl1", {
          type: "geojson",
          data: vteLvl1.value,
        });
        map.addLayer({
          id: "vteLvl1Fill",
          type: "fill",
          source: "vteLvl1",
          layout: {},
          paint: {
            // 'fill-color': 'black',
            'fill-color': [
              'case',
                ['boolean', ['feature-state', 'guessed'], false],
                'black',
                ['boolean', ['feature-state', 'guessing'], false],
                'purple',
                ['boolean', ['feature-state', 'fail'], false],
                'red',
                ['boolean', ['feature-state', 'success'], false],
                'green',
                'black'
            ],
            'fill-opacity': [
              'case',
              ['boolean', ['feature-state', 'guessed'], false],
              .8,
              ['boolean', ['feature-state', 'guessing'], false],
              .8,
              ['boolean', ['feature-state', 'clicked'], false],
              1,
              ['boolean', ['feature-state', 'fail'], false],
              .65,
              ['boolean', ['feature-state', 'success'], false],
              .65,
              ['boolean', ['feature-state', 'hover'], false],
              .8,
              0.15
            ]
          },
        });
        map.addLayer({
          id: "vteLvl1Line",
          type: "line",
          source: "vteLvl1",
          layout: {},
          paint: {
            'line-color': 'black',
            'line-width': 1.5
          },
        });
      });

    });
    return {
      mapPromise,
      vteLvl1
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

}

.game-hard-mode {
  left:auto;
  bottom:auto;
  right: 10px;
  top: 10px;
}

.game-easy-mode {
  left: 10px;
  bottom: 10px;
  right: auto;
  top: auto;
}
.guessing-village {
  color: rgb(165, 60, 184);
  font-size: 1.05rem;
  font-weight: 700;
}
</style>