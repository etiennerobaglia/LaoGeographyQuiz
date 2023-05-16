<template>
  <div class="game game-hard-mode">
    --- Hard Mode ---<br>
    <div v-if="playState != 'notPlaying' && playState != 'finished'" class="game-score">
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---------------&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;🟢 Success: {{ nbSuccess }}&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;🔴 Failed: &nbsp;&nbsp;&nbsp; {{ nbFail }}&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;----------------&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
    </div>
    <div class="game-instructions">
      <button v-if="playState == 'notPlaying'" @click="play()">
        Play!
      </button>
      <button v-if="playState == 'finished'" @click="replay()">
        RePlay!
      </button>
      <span v-if="guessingVillage.properties && playState!='success' && playState!='finished'">
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
import { defineComponent } from 'vue';
import { ref } from "vue";


export default defineComponent({
  props: {
    mapPromise: Object,
    playgroundLayer: Object,
  },
  setup(props) {
    let clickedVillage = ref(null);
    let guessingVillageIndex = ref({});
    let guessingVillage = ref({});
    let hoveredVillageId = null;
    let nbFail = ref(0);
    let nbSuccess = ref(0);
    let playState = ref('notPlaying');

    function replay() {
      nbSuccess.value = 0;
      nbFail.value = 0;
      play()
    }

    function play() {
      if (playState.value == "notPlaying") {
        props.mapPromise.then((map)=> { 
          map.on('mousemove', props.playgroundLayer.name+"Fill", (e) => {
            if (e.features.length > 0) {
              map.getCanvas().style.cursor = 'pointer'
              if (hoveredVillageId !== null) {
                map.setFeatureState(
                  { source: props.playgroundLayer.name, id: hoveredVillageId },
                  { hover: false }
                );
              }
              hoveredVillageId = e.features[0].id;
              map.setFeatureState(
                { source: props.playgroundLayer.name, id: hoveredVillageId },
                { hover: true }
              );
            }
          });

          map.on('mouseleave', props.playgroundLayer.name+"Fill", () => {
            if (hoveredVillageId !== null) {
              map.setFeatureState(
                { source: props.playgroundLayer.name, id: hoveredVillageId },
                { hover: false }
              );
              map.getCanvas().style.cursor = ''
            }
            hoveredVillageId = null;
          });

          map.on('click', props.playgroundLayer.name+"Fill", (e) => {
            clickedVillage.value = e.features[0]
            // success
            if (playState.value == 'playing' && clickedVillage.value.id == guessingVillage.value.id) {
              nbSuccess.value ++;
              playState.value = 'success';
              map.setFeatureState(
                { source: props.playgroundLayer.name, id: clickedVillage.value.id },
                { success: true }
              );
              // game must go on
              if (nbSuccess.value < 10) {
                window.setTimeout(() => {
                  map.setFeatureState(
                    { source: props.playgroundLayer.name, id: clickedVillage.value.id },
                    { success: false }
                  );
                  play()
                }, 2000);
              }
              // end of game
              else {
                playState.value = "finished"
                window.setTimeout(() => {
                  map.setFeatureState(
                    { source: props.playgroundLayer.name, id: clickedVillage.value.id },
                    { success: false }
                  );
                }, 2000);
              }
            }
            // fail
            else {
              nbFail.value ++;
              playState.value = 'failed';
              map.setFeatureState(
                { source: props.playgroundLayer.name, id: clickedVillage.value.id },
                { fail: true }
              );
              window.setTimeout(() => {
                map.setFeatureState(
                  { source: props.playgroundLayer.name, id: clickedVillage.value.id },
                  { fail: false }
                );
                playState.value = 'playing';
              }, 2000);
            }
          });
        })
      }
      playState.value = "playing"
      guessingVillageIndex.value = Math.floor(Math.random() * props.playgroundLayer.features.length);
      guessingVillage.value = props.playgroundLayer.features[guessingVillageIndex.value];
    }
      
    return {
      clickedVillage,
      guessingVillage,
      replay,
      play,
      playState,
      nbFail,
      nbSuccess
    };
  },
});
</script>

<style>
</style>