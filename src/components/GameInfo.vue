<template>
  <div class="game game-info">
    --- {{difficulty.charAt(0).toUpperCase() + difficulty.slice(1)}} Mode ---
    
    <span v-if="bestScore != null"> 
        - Best Score: {{ bestScore }}/{{totalAttemps}}
    </span>
    <br>
    
    <div class="game-score">
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;----------------&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;🟢 Success: {{ nbSuccess }}&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;🔴 Failed: &nbsp;&nbsp;&nbsp; {{ nbFail }}&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;----------------&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
    </div>

    <span class="game-instructions">

      <span v-if="playState!='finished'">
        [{{ nbSuccess+nbFail }}/{{totalAttemps}}]
      </span>

      <span v-if="difficulty == 'hard' && playState=='playing'">
        Guess: <br>&gt;&gt;<span class="guessing-feature" v-html="featureFullName(guessingFeature)"></span>&lt;&lt;
      </span>
      <span v-if="difficulty == 'easy' && guessingFeature.properties && playState=='playing'">
        Guess the name of the <span class="guessing-feature">purple feature</span> on the map:<br>
        <span v-for="option in guessingOptions">
          <button @click="guess(option)">{{featureFullName(option)}}</button><br>
        </span>
      </span>
    </span>

    <span class="game-feedback">
      
      <span v-if="difficulty == 'easy' && success==false && playState != 'playing'">
        ❌ Wrong, sorry the response was:<br>
        &gt;&gt; <span v-html="featureFullName(guessingFeature)"></span> &lt;&lt;
      </span>

      <span v-if="difficulty == 'hard' && success==false && playState != 'playing'">
        ❌ Wrong, sorry this is:<br>
        &gt;&gt; <span v-html="featureFullName(guessedFeature)"></span> &lt;&lt;
      </span>
      
      <span v-if="success==true && playState != 'playing'">
        ✅ Success! You found:<br> 
        <span class="guessing-feature" v-html="featureFullName(guessedFeature)"></span>
      </span>
      
      <span v-if="playState == 'next'">
        <br>Next round:
        <button @click="play()">
          Next!
        </button>
      </span>

      <span v-if="playState=='finished'">
        <br>[Game finished!] Your score is {{ nbSuccess }}/{{ totalAttemps }}.
        <br>You can 
        <button @click="replay()">
          RePlay!
        </button>
      </span>

    </span>

  </div>
</template>

<script>
import { defineComponent, onMounted } from 'vue';
import { ref } from "vue";


export default defineComponent({
  props: {
    mapPromise: Object,
    playgroundLayer: Object,
    difficulty: String,
  },
  setup(props) {
    let guessingOptions = ref([])
    let guessingFeature = ref({});
    let guessedFeature = ref({})
    let hoveredFeatureId = null;
    let nbFail = ref(0);
    let nbSuccess = ref(0);
    let playState = ref('playing');
    let success = ref(null);
    let bestScore = ref(null)
    let totalAttemps = ref(10)

    onMounted(() => { play()});


    function play() {
      setRandomFeatures();
      playState.value = "playing";
    }

    function replay() {
      nbSuccess.value = 0;
      nbFail.value = 0;
      play()
    }

    function guess(guessAttemp) {
      
      guessedFeature.value = guessAttemp

      props.mapPromise.then((map)=> {
        playState.value = "next";

        // success
        if ( guessedFeature.value.id == guessingFeature.value.id) {
          nbSuccess.value ++;
          success.value = true;
          
          let newFeatureState;
          if (props.difficulty == 'hard') newFeatureState = { success: true }
          if (props.difficulty == 'easy') newFeatureState = { guessing: false, success: true }
          
          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessingFeature.value.id },
            newFeatureState
          );
        }

        // fail
        else {
          nbFail.value ++;
          success.value = false;

          let newFeatureState;
          if (props.difficulty == 'hard') newFeatureState = { fail: true }
          if (props.difficulty == 'easy') newFeatureState = { fail: true, guessing: false }

          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessingFeature.value.id },
            newFeatureState
          )
        }

        // end of game
        if (nbSuccess.value + nbFail.value == totalAttemps.value) {
            playState.value = "finished";
        }

        // set bestscore
        if (playState.value == "finished" && (nbSuccess.value > bestScore.value || bestScore.value == null))
          bestScore.value = nbSuccess.value;
      })
    }

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

    function setRandomFeatures() {
      let previousFeature = guessingFeature.value;
      let newFeature;
      let newFeatures;
      
      if (props.difficulty == 'easy') {
        newFeatures = props.playgroundLayer.features.sort(() => .5 - Math.random()).slice(0,5)
        newFeature = newFeatures[Math.floor(Math.random() * newFeatures.length)]
      }

      if (props.difficulty == 'hard') {
        newFeature = props.playgroundLayer.features[Math.floor(Math.random() * props.playgroundLayer.features.length)];
      }

      if (newFeature.id == previousFeature.id)
        setRandomFeatures()

      else if (props.difficulty == 'easy') {
        guessingOptions.value = newFeatures;
        guessingFeature.value = newFeature;
        props.mapPromise.then((map)=> { 
          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessingFeature.value.id },
            { guessing: true }
          );
        })
      }

      else if (props.difficulty == 'hard') guessingFeature.value = newFeature;
    }

    // hard mode map listeners
    if (props.difficulty == 'hard') {
      props.mapPromise.then((map)=> { 
        // hover feature color
        map.on('mousemove', props.playgroundLayer.name+"Fill", (e) => {
          if (e.features.length > 0 && hoveredFeatureId !== null) {
            map.setFeatureState(
              { source: props.playgroundLayer.name, id: hoveredFeatureId },
              { hover: false }
            );
          }
          if (e.features.length > 0) {
            map.getCanvas().style.cursor = 'pointer'
            hoveredFeatureId = e.features[0].id;
            map.setFeatureState(
              { source: props.playgroundLayer.name, id: hoveredFeatureId },
              { hover: true }
            );
          }
        });
        map.on('mouseleave', props.playgroundLayer.name+"Fill", () => {
          if (hoveredFeatureId !== null) {
            map.getCanvas().style.cursor = ''
            map.setFeatureState(
              { source: props.playgroundLayer.name, id: hoveredFeatureId },
              { hover: false }
            );
          }
          hoveredFeatureId = null;
        });

        // user guess attempt hard
        map.on(
          'click',
          props.playgroundLayer.name+"Fill",
          (clickedFeature) => { 
            if (playState.value == 'playing') guess(clickedFeature.features[0]) 
          }
        )
      })
    }

    return {
      success,
      guessingFeature,
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
    };
  },
});
</script>

<style>
</style>