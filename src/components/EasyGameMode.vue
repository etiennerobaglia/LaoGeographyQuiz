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
      <form v-if="guessingVillage.properties && playState=='playing'">
        [{{ nbSuccess+nbFail+1 }}/{{totalAttemps}}] Guess the name of the <span class="guessing-village">purple village</span> on the map:<br>
        <span v-for="option in guessingOptions">
          <button @click="guess(option)">{{villageFullName(option)}}</button><br>
        </span>
      </form>
    </div>
    <div class="game-">
      <span v-if="playState=='failed'">
        ❌ Wrong, sorry the response was:<br>
        &gt;&gt; {{ villageFullName(guessingVillage) }} &lt;&lt;
      </span>
      <span v-if="playState=='success' && guessingVillage.properties.vname == guessedVillage.properties.vname">
        ✅ Success! You found:<br> 
        &gt;&gt; {{ villageFullName(guessingVillage) }} &lt;&lt; ...
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
    dataSet: Object,
  },
  setup(props) {
    let guessingVillage = ref({});
    let guessingVillageName = ref('');
    let guessedVillage = ref({})
    let nbFail = ref(0);
    let nbSuccess = ref(0);
    let playState = ref('notPlaying');
    let guessingOptions = ref([])
    let bestScore = ref(null)
    let totalAttemps = ref(10)

    function villageFullName(village) {
      let villageFullName = 
        village.properties?.vname + ", "
        + village.properties?.dname + " - "
        + village.properties?.l_vname + ", "
        + village.properties?.l_dname;
      return villageFullName
    }
    
    function guess(guessAttemp) {
      guessedVillage.value = guessAttemp
      props.mapPromise.then((map)=> {
        // success
        if (guessedVillage.value.id == guessingVillage.value.id) {
        nbSuccess.value ++;
        playState.value = 'success';
          map.setFeatureState(
            { source: 'vteLvl1', id: guessingVillage.value.id },
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
            { source: 'vteLvl1', id: guessingVillage.value.id },
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

    function setRandomVillages() {
      let previousVillage = guessingVillage.value;
      let newVillages = props.dataSet.features.sort(() => .5 - Math.random()).slice(0,5)
      let newVillage = newVillages[Math.floor(Math.random() * newVillages.length)]
      if (newVillage.id == previousVillage.id) setRandomVillages()
      else {
        guessingOptions.value = newVillages
        guessingVillage.value = newVillage
      }
    }

    function play() {
      playState.value = "playing"
      setRandomVillages()
      props.mapPromise.then((map)=> { 
        map.setFeatureState(
          { source: 'vteLvl1', id: guessingVillage.value.id },
          { guessing: true }
        );
      })
    }

    return {
      guessingVillage,
      guessingVillageName,
      guessedVillage,
      guessingOptions,
      villageFullName,
      play,
      replay,
      guess,
      playState,
      totalAttemps,
      nbFail,
      nbSuccess,
      bestScore,
      setRandomVillages
    };
  },
});
</script>

<style>
</style>