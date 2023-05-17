<template>
  <div class="game game-easy-mode">
    --- Easy Mode ---
    <span v-if="bestScore != null"> - Best Score: {{ bestScore }}/{{totalAttemps}}</span><br>
    <div v-if="playState != 'notPlaying' && playState != 'finished'" class="game-score">
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;----------------&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;🟢 Success: {{ nbSuccess }}&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;🔴 Failed: &nbsp;&nbsp;&nbsp; {{ nbFail }}&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;----------------&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
    </div>
    <div class="game-instructions">
      <span v-if="playState=='notPlaying'">
        Start game?
        <button v-if="playState == 'notPlaying'" @click="play">
          Play!
        </button>
      </span>
      <span v-if="playState=='finished'">
        Start a new game? 
        <button v-if="playState == 'finished'" @click="replay">
          RePlay!
        </button>
      </span>
      <div v-if="guessingFeature.properties && playState=='playing'">
        [{{ nbSuccess+nbFail+1 }}/{{totalAttemps}}] Guess the name of the <span class="guessing-feature">purple feature</span> on the map:<br>
        <span v-for="option in guessingOptions">
          <button @click="guess(option)">{{featureFullName(option)}}</button><br>
        </span>
      </div>
    </div>
    <div class="game-feedback">
      <span v-if="playState=='failed'">
        ❌ Wrong, sorry the response was:<br>
        &gt;&gt; {{ featureFullName(guessingFeature) }} &lt;&lt;
      </span>
      <span v-if="playState=='success'">
        ✅ Success! You found:<br> 
        &gt;&gt; {{ featureFullName(guessingFeature) }} &lt;&lt; ...
      </span>
    </div>
  </div>
</template>

<script>
import { defineComponent, onMounted } from 'vue';
import { ref } from "vue";


export default defineComponent({
  props: {
    mapPromise: Object,
    playgroundLayer: Object,
  },
  setup(props) {
    let guessingFeature = ref({});
    let guessingFeatureName = ref('');
    let guessedFeature = ref({})
    let nbFail = ref(0);
    let nbSuccess = ref(0);
    let playState = ref('playing');
    let guessingOptions = ref([])
    let bestScore = ref(null)
    let totalAttemps = ref(10)


    onMounted(() => { play()});

    function featureFullName(feature) {
      let featureFullName;
      if (feature.properties?.vname)
        featureFullName = 
          feature.properties?.vname + ", "
          + feature.properties?.dname + " - "
          + feature.properties?.l_vname + ", "
          + feature.properties?.l_dname;
          else if (feature.properties?.dname)
          featureFullName =
          feature.properties?.dname + ", "
          + feature.properties?.pname + " - "
          + feature.properties?.l_dname + ", "
          + feature.properties?.l_pname;
      else if (feature.properties?.pname)
        featureFullName = 
          feature.properties?.pname + " - "
          + feature.properties?.l_pname;
      return featureFullName
    }
    
    function guess(guessAttemp) {
      guessedFeature.value = guessAttemp
      props.mapPromise.then((map)=> {
        // success
        if (guessedFeature.value.id == guessingFeature.value.id) {
        nbSuccess.value ++;
        playState.value = 'success';
          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessingFeature.value.id },
            { 
              guessing: false,
              success: true,
            }
          );
        }
        // fail
        else {
          nbFail.value ++;
          playState.value = 'failed';
          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessingFeature.value.id },
            { 
              fail: true,
              guessing: false
            }
          )
        }
        // game must go on
        if (nbSuccess.value + nbFail.value < totalAttemps.value)
          window.setTimeout(() => play(), 2000);
        // end of game
        else {
          playState.value = "finished";
          if (nbSuccess.value > bestScore.value || bestScore.value == null)
            bestScore.value = nbSuccess.value;
        }
      })
    }

    function replay() {
      nbSuccess.value = 0;
      nbFail.value = 0;
      play()
    }

    function setRandomFeatures() {
      let previousFeature = guessingFeature.value;
      let newFeatures = props.playgroundLayer.features.sort(() => .5 - Math.random()).slice(0,5)
      let newFeature = newFeatures[Math.floor(Math.random() * newFeatures.length)]
      if (newFeature.id == previousFeature.id) setRandomFeatures()
      else {
        guessingOptions.value = newFeatures
        guessingFeature.value = newFeature
      }
    }

    function play() {
      playState.value = "playing"
      setRandomFeatures()
      props.mapPromise.then((map)=> { 
        map.setFeatureState(
          { source: props.playgroundLayer.name, id: guessingFeature.value.id },
          { guessing: true }
        );
      })
    }

    return {
      guessingFeature,
      guessingFeatureName,
      guessedFeature,
      guessingOptions,
      featureFullName,
      play,
      replay,
      guess,
      playState,
      totalAttemps,
      nbFail,
      nbSuccess,
      bestScore,
      setRandomFeatures
    };
  },
});
</script>

<style>
</style>