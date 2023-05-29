<template>
  <div class="game-info">

    <Teleport to="#game-modal-info" v-if="playState != 'notPlaying'">
      <div class="game-data">
        <div class="game-mode">
          {{difficulty.charAt(0).toUpperCase() + difficulty.slice(1)}} Mode
        </div>
        <div class="game-scores">
          <div class="game-score">
            <span>
              <span class="green">■</span> 
              Success
            </span>
            <span>{{ nbSuccess }}</span>
          </div>
          <div class="game-score">
            <span>
              <span class="red">■</span> 
              Failed
            </span>
            <span>{{ nbFail }}</span>
          </div>
        </div>
        <span class="game-best-score" v-if="bestScore != null"> 
          Best Score: {{ bestScore }}/{{totalAttemps}}
        </span>
      </div>
    </Teleport>

    <div class="game-instructions">

      <span class="game-instruction-text" v-if="playState!='finished'">
        [{{ nbSuccess+nbFail }}/{{totalAttemps}}]
      </span>

      <div 
        class="game-difficulty game-difficulty-hard" 
        v-if="difficulty == 'hard' && guessingFeature.properties && playState!='finished'"
      >

        <span class="game-instruction-text">
          Instructions: Find 
          <span class="guessing-feature">this administrative</span>
          area by clicking on the map
        </span>

        <button
          :disabled="playState != 'playing'"
          title="Find me on the map!"
          class="button button-selection button-not-hoverable"
          :class="{
            'button-yellow-blue': 
              !guessedFeature.id ||
              (guessingFeature.id != guessedFeature.id)
            ,
            'button-green':
              guessingFeature.id == guessedFeature.id
            ,
            'button-red':
              guessingFeature.id == guessedFeature.id
            ,
            'button-border-blue':
              guessedFeature.id
              && guessingFeature.id != guessedFeature.id
          }"
        >
          {{ featureFullName(guessingFeature) }} 
        </button>
        
        <button
          v-if="
            playState == 'next'
            && guessingFeature.id != guessedFeature.id
          "
          disabled=true
          class="button button-selection button-not-hoverable button-red"
        >
          {{ featureFullName(guessedFeature) }} 
        </button>

      </div>

      <div class="game-difficulty game-difficulty-easy" v-if="difficulty == 'easy' && guessingFeature.properties && playState!='finished'">
        
        <span class="game-instruction-text">
          Instructions: Guess the name of the 
          <span class="guessing-feature">yellow shape</span>
          on the map
        </span>
        
        <div class="game-buttons">
          <button
            :disabled="
              playState != 'playing'
            "
            v-for="option in guessingOptions"
            class="button button-selection"
            :class="{
              'button-yellow-blue': 
                !guessedFeature.id ||
                ( option.id != guessedFeature.id)
              ,
              'button-green':
                option.id == guessingFeature.id
                && option.id == guessedFeature.id
              ,
              'button-red':
                option.id != guessingFeature.id
                && option.id == guessedFeature.id
              ,
              'button-border-blue':
                guessedFeature.id
                && option.id == guessingFeature.id
                && option.id != guessedFeature.id
              ,
            }"
            @click="guess(option)"
          >
            {{featureFullName(option)}}
          </button>
        
        </div>
      </div>
    </div>

    <div class="game-feedback">

      <button 
        @click="next()"
        :disabled="playState != 'next'"
        v-if="nbSuccess + nbFail != totalAttemps"
        class="button button-yellow button-next"
      >
        Next Round!
      </button>

      <button
        v-else-if="playState != 'finished'"
        @click="score()"
        class="button button-yellow button-next"
      >
        See score!
      </button>

      <div class="game-finished" v-if="playState=='finished'">
        [Game finished!] 
        <br>
        Your score is {{ nbSuccess }}/{{ totalAttemps }}.
        <button 
          @click="replay()"
          ref:enterButton
          class="button button-yellow"
        >
          RePlay!
        </button>
      </div>

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

    function score() {
        // end of game
        playState.value = "finished";

        // set bestscore
        if (playState.value == "finished" && (nbSuccess.value > bestScore.value || bestScore.value == null))
          bestScore.value = nbSuccess.value;
        
        next()
    }

    function next() {

      props.mapPromise.then((map)=> {
        map.setFeatureState(
          { source: props.playgroundLayer.name, id: guessedFeature.value.id },
          { fail: false, guessing: false, success: false }
        )

        map.setFeatureState(
          { source: props.playgroundLayer.name, id: guessingFeature.value.id },
          { fail: false, guessing: false, success: false }
        );

        if (guessingFeature.value.id == guessedFeature.value.id) {
          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessingFeature.value.id },
            { done: true }
          );
        }

        guessingFeature.value = {}
        guessedFeature.value = {}

        // game must go on
        if (playState.value != "finished") play()

      })
    }

    function replay() {
      nbSuccess.value = 0;
      nbFail.value = 0;
      props.mapPromise.then((map)=> {
        map.removeFeatureState(
          {source: props.playgroundLayer.name}
        )
        play()
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
          
          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessingFeature.value.id },
            { guessing: false, success: true }
          );
        }

        // fail
        else {
          nbFail.value ++;
          success.value = false;

          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessedFeature.value.id },
            { fail: true }
          )
          map.setFeatureState(
            { source: props.playgroundLayer.name, id: guessingFeature.value.id },
            { guessing: true }
          )
        }
      })
    }

    function featureFullName(feature) {
      let featureFullName;
      if (feature.properties?.vname)
        featureFullName = 
          feature.properties?.vname 
          // + ", "
          // + feature.properties?.dname 
            + " - "
          + feature.properties?.l_vname 
          //   + ", "
          // + feature.properties?.l_dname;
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
        newFeatures = props.playgroundLayer.features.sort(() => .5 - Math.random()).slice(0,4)
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
          if (e.features.length > 0 && hoveredFeatureId !== null && playState.value == 'playing') {
            map.setFeatureState(
              { source: props.playgroundLayer.name, id: hoveredFeatureId },
              { hover: false }
            );
          }
          if (e.features.length > 0 && playState.value == 'playing') {
            map.getCanvas().style.cursor = 'pointer'
            hoveredFeatureId = e.features[0].id;
            map.setFeatureState(
              { source: props.playgroundLayer.name, id: hoveredFeatureId },
              { hover: true }
            );
          }
          if (e.features.length > 0 && playState.value != 'playing')
            map.getCanvas().style.cursor = 'not-allowed'
        });
        map.on('mouseleave', props.playgroundLayer.name+"Fill", () => {
          map.getCanvas().style.cursor = ''
          if (hoveredFeatureId !== null && playState.value == 'playing') {
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
      score,
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