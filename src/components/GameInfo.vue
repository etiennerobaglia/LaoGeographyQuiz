<template>
  <div class="game-info">

    <Teleport to="#map-info" v-if="playState != 'notPlaying'">
      <div class="game-map-data">
        <div class="game-mode">
          {{difficulty.charAt(0).toUpperCase() + difficulty.slice(1)}} Mode
        </div>
        <div class="game-score">
          🟢 Success: {{ nbSuccess }}<br>
          ⚫️ Failed: {{ nbFail }}
        </div>
        <span class="game-best-score" v-if="bestScore != null"> 
          Best Score: {{ bestScore }}/{{totalAttemps}}
        </span>
      </div>
    </Teleport>

    <span class="game-instructions">

      <span v-if="playState!='finished'">
        [{{ nbSuccess+nbFail }}/{{totalAttemps}}]
      </span>

      <span v-if="difficulty == 'hard' && playState=='playing'">
        <span class="game-instructions-title">Instructions:</span> 
        Click on this vilage on the map <br>
        <span class="guessing-feature" v-html="featureFullName(guessingFeature)"></span>
      </span>

      <span v-if="difficulty == 'easy' && guessingFeature.properties && playState!='finished'">
        <span class="game-instructions-title">Instructions:</span> Guess the name of the <span class="guessing-feature">red feature</span> on the map:<br>
        
        <span v-for="option in guessingOptions">
          <button
            :disabled="playState != 'playing'"
            class="button-selection"
            :class="{
              'button-red': 
                !guessedFeature.id ||
                ( option.id != guessedFeature.id)
              ,
              'button-green':
                option.id == guessingFeature.id
                && option.id == guessedFeature.id
              ,
              'button-grey':
                option.id != guessingFeature.id
                && option.id == guessedFeature.id
              ,
              'button-border-green':
                guessedFeature.id
                && option.id == guessingFeature.id
                && option.id != guessedFeature.id
              ,
            }"
            @click="guess(option)"
          >
            {{featureFullName(option)}}
          </button>
        </span>

      </span>
    </span>

    <span class="game-feedback">

      <span v-if="difficulty == 'hard' && success==false && playState != 'playing'">
        ❌ Wrong, sorry this is:<br>
        <span v-html="featureFullName(guessedFeature)"></span>
      </span>
      
      <span v-if="difficulty == 'hard' && success==true && playState != 'playing'">
        ✅ Success! You found:<br> 
        <span class="guessing-feature" v-html="featureFullName(guessedFeature)"></span>
      </span>
      
      <span>
        <button 
          @click="next()"
          :disabled="playState != 'next'"
          class="button-yellow button-next"
        >
          Next Round!
        </button>
      </span>

      <span v-if="playState=='finished'">
        <br><br>[Game finished!] Your score is {{ nbSuccess }}/{{ totalAttemps }}.
        <br>You can 
        <button 
          @click="replay()"
          ref:enterButton
          class="button-yellow"
        >
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
    let gameDoneFeaturesIDs = ref([])

    onMounted(() => { play()});

    function play() {
      setRandomFeatures();
      playState.value = "playing";
    }

    function next() {
      let newFeatureState;
      if (props.difficulty == 'hard') newFeatureState = { fail: false }
      if (props.difficulty == 'easy') newFeatureState = { fail: false, guessing: false }
      props.mapPromise.then((map)=> {
        map.setFeatureState(
          { source: props.playgroundLayer.name, id: guessingFeature.value.id },
          newFeatureState
        );
        map.setFeatureState(
          { source: props.playgroundLayer.name, id: guessedFeature.value.id },
          newFeatureState
        )
        guessingFeature.value = {}
        guessedFeature.value = {}
        play()
      })
    }

    function replay() {
      nbSuccess.value = 0;
      nbFail.value = 0;
      props.mapPromise.then((map)=> {
        map.removeFeatureState(
          {source: props.playgroundLayer.name}
        )
        next()
      })
    }

    function guess(guessAttemp) {
      
      guessedFeature.value = guessAttemp
      playState.value = "next";

      props.mapPromise.then((map)=> {

        // success
        if (guessedFeature.value.id == guessingFeature.value.id) {
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
          if (props.difficulty == 'easy') newFeatureState = { fail: true }

          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessedFeature.value.id },
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
          feature.properties?.dname + " - "
          + feature.properties?.l_dname
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

      if (
        newFeature.id == previousFeature.id
        || gameDoneFeaturesIDs.value.includes(newFeature.id)
      )
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

      else if (props.difficulty == 'hard')
        guessingFeature.value = newFeature;
      
      gameDoneFeaturesIDs.value.push(newFeature.id)
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
      next,
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

<style scoped>



</style>