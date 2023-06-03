<template>

    <div 
      class="game-menu-selection-portal"
      v-if="!playing"
    >
      <div class="game-menu-select">
        <label class="game-menu-selection-playground">Playground</label>
        <select 
          v-model="playgroundName"
          name="game-menu-difficulty"
        >
          <option disabled value="">- Select Playground -</option>
          <option v-for="layer in layersInfo" :value="layer.fileName">
            {{ layer.fullName }}
          </option>
        </select>
      </div>

      <div class="game-menu-select">
        <label class="game-menu-selection-difficulty">Game Difficulty</label>
        <select 
          v-model="difficulty" 
          name="game-menu-difficulty"
        >
          <option disabled value="">-&nbsp;&nbsp;&nbsp;Select Difficulty&nbsp;&nbsp;&nbsp;-</option>
          <option value="easy">Easy</option>
          <option value="hard">Hard</option>
        </select>
      </div>

      <button
        class="button button-yellow"
        :disabled="!playgroundName || !difficulty"
        @click="initGame()"
      >
        Play!
      </button>

    </div>

    
    <GameInfo
      v-if="playing && difficulty && playgroundName"
      :difficulty="difficulty"
      :mapPromise="mapPromise"
      :playgroundLayer="playgroundLayer"
      :playgroundInfo="playgroundInfo"
    />

</template>

<script>
import { defineComponent, ref } from 'vue';
import GameInfo from './GameInfo.vue';

export default defineComponent({
  props: {
    mapPromise: Object,
  },
  components: {
    GameInfo,
  },
  setup(props) {
    const playgroundName = ref("");
    let playgroundInfo = ref({})
    const difficulty = ref("");
    let playgroundLayer = ref({})
    let playing = ref(false)
    let laoBounds = [[99.6585338255183, 13.7970728629234], [108.3101822387705, 22.83759711581436]];
    let layersInfo = ref([
      {
        "fileName": "vte-villages-t2",
        "fullName":"Vientiane - City Center (39 villages)",
        "bounds": [[102.57813377330893, 17.927212142550417], [102.64211925985438, 17.98422754876998]],
      }, 
      {
        "fileName": "vte-villages-t4",
        "fullName":"Vientiane - T4 (112 villages)",
        "bounds": [[102.52367049329712,17.88439633410357], [102.721931487635, 18.03925442276218]]
      },
      {
        "fileName": "vte-villages-450",
        "fullName":"Vientiane - Road 450 (250 villages)",
        "bounds": [ [102.40350167471024, 17.784056559755257], [102.87552283140121, 18.12166650874893]]
      },
      {
        "fileName": "lao-districts",
        "fullName":"Lao - Districts",
        "bounds": laoBounds
      },
      {
        "fileName": "vte-pro_and_cap-districts",
        "fullName":"Extended Vientiane - Districts",
        "bounds": [[101.17486890018642, 17.74580310199005], [103.35160930021289, 19.45418532763813]]
      },
      {
        "fileName": "lao-provinces",
        "fullName":"Lao - Provinces",
        "bounds": laoBounds
      }
    ])

    function initGame() {
      import(`../assets/layers/${playgroundName.value}.json`).then((importLayer) => {
        playgroundLayer.value = importLayer;
        
        playgroundInfo.value = layersInfo.value.find(
          layer => layer.fileName == playgroundName.value
        ),
        
        props.mapPromise.then((map) => {
          map.addSource(playgroundLayer.value.name, {
            type: "geojson",
            data: playgroundLayer.value,
          });
          map.addLayer({
            id: playgroundLayer.value.name+"Fill",
            type: "fill",
            source: playgroundLayer.value.name,
            layout: {},
            paint: {
              'fill-color': [
                'case',
                  ['boolean', ['feature-state', 'fail'], false],
                  import.meta.env.VITE_red,
                  ['boolean', ['feature-state', 'success'], false],
                  import.meta.env.VITE_green,
                  ['boolean', ['feature-state', 'done'], false],
                  import.meta.env.VITE_blue,
                  ['boolean', ['feature-state', 'guessing'], false],
                  import.meta.env.VITE_yellow,
                  ['boolean', ['feature-state', 'hover'], false],
                  import.meta.env.VITE_yellow,
                  'black'
              ],
              'fill-opacity': [
                'case',
                ['boolean', ['feature-state', 'guessing'], false],
                1,
                ['boolean', ['feature-state', 'hover'], false],
                1,
                ['boolean', ['feature-state', 'fail'], false],
                .85,
                ['boolean', ['feature-state', 'success'], false],
                .85,
                ['boolean', ['feature-state', 'done'], false],
                .85,
                0
              ]
            },
          });

          map.addLayer({
            id: playgroundLayer.value.name+"Line",
            type: "line",
            source: playgroundLayer.value.name,
            layout: {},
            paint: {
              'line-color': import.meta.env.VITE_blue,
              'line-width': 2
            },
          });

          map.fitBounds(
            playgroundInfo.value.bounds,
            {padding: 20}
          )
        })
        playing.value = true;
      });
    }

    return {
      layersInfo,
      initGame,
      playgroundName,
      playgroundInfo,
      playgroundLayer,
      difficulty,
      playing,
    };
  },
});
</script>

<style>
</style>