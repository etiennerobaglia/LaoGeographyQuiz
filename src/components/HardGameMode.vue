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
      <span v-if="guessingFeature.properties && playState!='success' && playState!='finished'">
        Guess: &gt;&gt;<span v-html="featureFullName(guessingFeature)"></span>&lt;&lt;
      </span>
    </div>
    <div class="game-">
      <span v-if="playState=='failed'">
        ❌ Wrong, this is <span v-html="featureFullName(clickedFeature)"></span>
        <br>
      </span>
      <span v-if="playState=='success'">
        ✅ Success! <br>
        You found <span v-html="featureFullName(clickedFeature)"></span>...
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
    let clickedFeature = ref(null);
    let guessingFeatureIndex = ref({});
    let guessingFeature = ref({});
    let hoveredFeatureId = null;
    let nbFail = ref(0);
    let nbSuccess = ref(0);
    let playState = ref('notPlaying');

    function replay() {
      nbSuccess.value = 0;
      nbFail.value = 0;
      play()
    }

    function featureFullName(feature) {
      let featureFullName;
      if (feature.properties?.vname)
        featureFullName = 
          feature.properties?.vname + ", "
          + feature.properties?.dname + "<br>"
          + feature.properties?.l_vname + ", "
          + feature.properties?.l_dname;
          else if (feature.properties?.dname)
          featureFullName = 
          feature.properties?.dname + ", "
          + feature.properties?.pname + "<br>"
          + feature.properties?.l_dname + ", "
          + feature.properties?.l_pname;
      else if (feature.properties?.pname)
        featureFullName = 
          feature.properties?.pname + " - "
          + feature.properties?.l_pname;
      return featureFullName
    }

    function play() {
      if (playState.value == "notPlaying") {
        props.mapPromise.then((map)=> { 
          map.on('mousemove', props.playgroundLayer.name+"Fill", (e) => {
            if (e.features.length > 0) {
              map.getCanvas().style.cursor = 'pointer'
              if (hoveredFeatureId !== null) {
                map.setFeatureState(
                  { source: props.playgroundLayer.name, id: hoveredFeatureId },
                  { hover: false }
                );
              }
              hoveredFeatureId = e.features[0].id;
              map.setFeatureState(
                { source: props.playgroundLayer.name, id: hoveredFeatureId },
                { hover: true }
              );
            }
          });

          map.on('mouseleave', props.playgroundLayer.name+"Fill", () => {
            if (hoveredFeatureId !== null) {
              map.setFeatureState(
                { source: props.playgroundLayer.name, id: hoveredFeatureId },
                { hover: false }
              );
              map.getCanvas().style.cursor = ''
            }
            hoveredFeatureId = null;
          });

          map.on('click', props.playgroundLayer.name+"Fill", (e) => {
            clickedFeature.value = e.features[0]
            // success
            if (playState.value == 'playing' && clickedFeature.value.id == guessingFeature.value.id) {
              nbSuccess.value ++;
              playState.value = 'success';
              map.setFeatureState(
                { source: props.playgroundLayer.name, id: clickedFeature.value.id },
                { success: true }
              );
              // game must go on
              if (nbSuccess.value < 10) {
                window.setTimeout(() => {
                  map.setFeatureState(
                    { source: props.playgroundLayer.name, id: clickedFeature.value.id },
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
                    { source: props.playgroundLayer.name, id: clickedFeature.value.id },
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
                { source: props.playgroundLayer.name, id: clickedFeature.value.id },
                { fail: true }
              );
              window.setTimeout(() => {
                map.setFeatureState(
                  { source: props.playgroundLayer.name, id: clickedFeature.value.id },
                  { fail: false }
                );
                playState.value = 'playing';
              }, 2000);
            }
          });
        })
      }
      playState.value = "playing"
      guessingFeatureIndex.value = Math.floor(Math.random() * props.playgroundLayer.features.length);
      guessingFeature.value = props.playgroundLayer.features[guessingFeatureIndex.value];
    }
      
    return {
      featureFullName,
      clickedFeature,
      guessingFeature,
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