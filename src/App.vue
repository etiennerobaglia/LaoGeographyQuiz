<template>
  <div id="map"></div>

  <div class="game">

    <div class="game-score">
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;----------------&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;🟢 Success: {{ nbSuccess }}&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;🔴 Failed: &nbsp;&nbsp;&nbsp; {{ nbFail }}&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;----------------&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
    </div>

    <div class="game-instructions">
      <span v-if="playState=='notPlaying'">
        <button v-if="playState == 'notPlaying'" @click="play">
          Play!
        </button>  
      </span>
  
      <span v-if="guessingVillage.properties && playState!='success'">
        Guess: <strong>&gt;&gt;{{ guessingVillage.properties.vname }}&lt;&lt;</strong> 
      </span>
    </div>

    <div class="game-">
      <span v-if="playState=='failed'">
        ❌ Wrong, this is {{ clickedVillage.properties.vname }}<br>
      </span>
      <span v-if="playState=='success' && guessingVillage.properties.vname == clickedVillage.properties.vname">
        ✅ Success! <br>
        You found {{ clickedVillage.properties.vname }}...
      </span>
    </div>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import { onMounted, ref, reactive } from "vue";
import villagesT2 from './assets/layers/vientiane-capital-villages-t2.json'

export default {
  setup() {
    let clickedVillage = ref(null);
    let randomVillages = ref([]);
    let guessingVillage = ref({});
    let guessingVillageIndex = ref(null);
    let hoveredVillageId = null;
    let nbFail = ref(0);
    let nbSuccess = ref(0);
    let playState = ref('notPlaying');
    let vteLvl1 = ref(villagesT2);
    const mapPromise = ref();
    
    function randomVillageSelection(source, number) {
      return source.sort(() => .5 - Math.random()).slice(0,number) 
    }

    function play() {
      playState.value = "playing"
      guessingVillageIndex.value = Math.floor(Math.random() * vteLvl1.value.features.length);
      guessingVillage.value = vteLvl1.value.features[guessingVillageIndex.value];
      guessingVillage.value.selected = true;
      // mapPromise.value.then((map)=> { })
    }
    
    onMounted(() => {
      mapboxgl.accessToken = import.meta.env.VITE_VUE_APP_MAPBOX_TOKEN;
      
      const map = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/etroba/clh1k3m1400l901qufshrdakc",
        // center: [102.61116548510415, 17.96297214903177],
        // zoom: 12.5,
        maxBounds:[
          [102.51981436706262, 17.89402516295418],
          [102.72512167503737, 18.04856960969768]
        ],
        bounds:[
          [102.5806565788472, 17.93210405768643],
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
            'fill-color': 'black',
            'fill-opacity': [
              'case',
              ['boolean', ['feature-state', 'clicked'], false],
              1,
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

        map.on('click', (e) => {
          console.log(e.lngLat.lng+", "+e.lngLat.lat)
        })
        
        map.on('mousemove', 'vteLvl1Fill', (e) => {
          if (e.features.length > 0) {
            if (hoveredVillageId !== null) {
              map.setFeatureState(
                { source: 'vteLvl1', id: hoveredVillageId },
                { hover: false }
              );
              map.getCanvas().style.cursor = 'pointer'
            }
            hoveredVillageId = e.features[0].id;
            map.setFeatureState(
              { source: 'vteLvl1', id: hoveredVillageId },
              { hover: true }
            );
          }
        });

        map.on('mouseleave', 'vteLvl1Fill', () => {
          if (hoveredVillageId !== null) {
            map.setFeatureState(
              { source: 'vteLvl1', id: hoveredVillageId },
              { hover: false }
            );
            map.getCanvas().style.cursor = ''
          }
          hoveredVillageId = null;
        });

        map.on('click', 'vteLvl1Fill', (e) => {
          clickedVillage.value = e.features[0]
          console.log(clickedVillage.value.properties.vname)
          if (playState.value == 'playing' || playState.value == 'failed') {
            if (clickedVillage.value.id == guessingVillage.value.id) {
              nbSuccess.value ++;
              playState.value = 'success';
              if (nbSuccess.value < 10) {
                window.setTimeout(play, 2000)
              } 
            }
            else {
              nbFail.value ++;
              playState.value = 'failed';
            }
          }
        });
      });

    });
    return {
      clickedVillage,
      randomVillages,
      guessingVillage,
      play,
      playState,
      nbFail,
      nbSuccess
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
  background-color: black;
  position: fixed;
  left: 10px;
  bottom: 10px;
}
</style>