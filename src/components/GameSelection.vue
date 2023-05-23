<template>
  <div 
    v-if="!playing"
    class="game game-selection-portal"
  >
    <h1>ບ້ານລາວຢູ່ໃສ</h1>
    <h1>BanLaoYuSai</h1>
    <br>
    <label for="game-selection-playground">Select Playground</label>
    <br>
    <select 
      v-model="playgroundName"
      name="game-difficulty"
    >
      <option disabled value="">- Playground -</option>
      <option v-for="layer in layers" :value="layer.fileName">
        {{ layer.fullName }}
      </option>
    </select>
    <br>
    <br>
    <label for="game-selection-difficulty">Select Game Difficulty</label>
    <br>
    <select 
      v-model="difficulty" 
      name="game-difficulty"
    >
      <option disabled value="">- Difficulty -</option>
      <option value="easy">Easy</option>
      <option value="hard">Hard</option>
    </select>
    <br>
    <br>
    <button
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
  />

</template>

<script>
import { defineComponent } from 'vue';
import { ref, computed } from "vue";
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
    const difficulty = ref("");
    let playgroundLayer = ref({})
    let playing = ref(false)
    
    let laoBounds = [[99.6585338255183, 13.7970728629234], [108.3101822387705, 22.83759711581436]];
    
    let layers = ref([
      {
        "fileName": "vte-pro_and_cap-districts",
        "fullName":"Extended Vientiane - Districts",
        "bounds": [[101.17486890018642, 17.74580310199005], [103.35160930021289, 19.45418532763813]]
      },
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
        "fileName": "lao-provinces",
        "fullName":"Lao - Provinces",
        "bounds": laoBounds
      },
      {
        "fileName": "lao-districts",
        "fullName":"Lao - Districts",
        "bounds": laoBounds
      }
    ])

    function initGame() {
      import(`../assets/layers/${playgroundName.value}.json`).then((importLayer) => {
        playgroundLayer.value = importLayer;
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
            id: playgroundLayer.value.name+"Line",
            type: "line",
            source: playgroundLayer.value.name,
            layout: {},
            paint: {
              'line-color': 'black',
              'line-width': 1.5
            },
          });

          map.fitBounds(
            layers.value.find(
              layer => layer.fileName == playgroundName.value
              ).bounds,
            {padding: 20}
          );

        })
        playing.value = true;
      });
    }

    return {
      layers,
      initGame,
      playgroundName,
      playgroundLayer,
      difficulty,
      playing,
    };
  },
});
</script>

<style>
.game-selection-portal {
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>