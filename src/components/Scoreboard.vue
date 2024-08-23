<script setup>
import Confetti from './Confetti.vue';
import { ref, watch } from 'vue'

const scorePlayerOne = ref(0)
const scorePlayerTwo = ref(0)
const playerOne = ref('Player 1')
const playerTwo = ref('Player 2')
const medalsPlayerOne = ref(0)
const medalsPlayerTwo = ref(0)

const colorPlayerOne = ref('bg-red-500')
const colorPlayerTwo = ref('bg-blue-500')

const showModal = ref(false)
const winnerModal = ref(false) // Tambahan untuk modal pemenang
const currentPlayer = ref('')
const currentName = ref('')
const showSettings = ref(false)

const maxScore = ref(localStorage.getItem('maxScore') || 10)
const maxRound = ref(localStorage.getItem('maxRound') || 3) // Ganti maxWin menjadi maxRound dan ambil dari localStorage
const deuceEnabled = ref(localStorage.getItem('deuceEnabled') === 'true' || false)

// History untuk Undo
const history = ref([])

watch([maxScore, maxRound, deuceEnabled], () => {
  localStorage.setItem('maxScore', maxScore.value)
  localStorage.setItem('maxRound', maxRound.value) // Simpan maxRound ke localStorage
  localStorage.setItem('deuceEnabled', deuceEnabled.value)
})

function incrementScore(player) {
  if (player === 'one') {
    scorePlayerOne.value++
    history.value.push({ player: 'one', score: scorePlayerOne.value })
  } else if (player === 'two') {
    scorePlayerTwo.value++
    history.value.push({ player: 'two', score: scorePlayerTwo.value })
  }
  checkWinCondition()
}

function checkWinCondition() {
  if (deuceEnabled.value) {
    if (Math.abs(scorePlayerOne.value - scorePlayerTwo.value) >= 2 &&
      (scorePlayerOne.value >= maxScore.value || scorePlayerTwo.value >= maxScore.value)) {
      awardMedal()
    }
  } else {
    if (scorePlayerOne.value >= maxScore.value || scorePlayerTwo.value >= maxScore.value) {
      awardMedal()
    }
  }
}

function awardMedal() {
  if (scorePlayerOne.value > scorePlayerTwo.value) {
    medalsPlayerOne.value++
  } else {
    medalsPlayerTwo.value++
  }
  resetScores()
  checkGameWinner() // Cek apakah sudah ada pemenang
}

function checkGameWinner() {
  if (medalsPlayerOne.value + medalsPlayerTwo.value >= maxRound.value) {
    winnerModal.value = true
  }
}

function undo() {
  const lastAction = history.value.pop()
  if (lastAction) {
    if (lastAction.player === 'one') {
      scorePlayerOne.value = lastAction.score - 1
    } else if (lastAction.player === 'two') {
      scorePlayerTwo.value = lastAction.score - 1
    }
  }
}

function resetScores() {
  scorePlayerOne.value = 0
  scorePlayerTwo.value = 0
  history.value = []
}

function flipPositions() {
  const tempScore = scorePlayerOne.value
  scorePlayerOne.value = scorePlayerTwo.value
  scorePlayerTwo.value = tempScore

  const tempPlayer = playerOne.value
  playerOne.value = playerTwo.value
  playerTwo.value = tempPlayer

  const tempColor = colorPlayerOne.value
  colorPlayerOne.value = colorPlayerTwo.value
  colorPlayerTwo.value = tempColor
}

function openModal(player) {
  currentPlayer.value = player
  currentName.value = player === 'one' ? playerOne.value : playerTwo.value
  showModal.value = true
}

function saveName() {
  if (currentPlayer.value === 'one') {
    playerOne.value = currentName.value
  } else if (currentPlayer.value === 'two') {
    playerTwo.value = currentName.value
  }
  showModal.value = false
}
function resetGame() {
  winnerModal.value = false
  resetScores()
  medalsPlayerOne.value = 0
  medalsPlayerTwo.value = 0
}

</script>



<template>
  <Confetti v-if="winnerModal" />
  <div class="flex items-center justify-center h-[100svh]" :class="[showSettings || winnerModal ? 'blur-md' : '']">
    <div class="h-full w-full sm:w-[40rem] sm:h-[40rem] flex flex-col relative">
      <!-- Player 1 -->
      <div :class="[colorPlayerOne, 'flex-1 flex items-center justify-center sm:rounded-t-xl']"
        @click="incrementScore('one')">
        <div class="absolute top-8 bg-slate-100 rounded-xl px-4 py-2 flex gap-2 items-center group">
          {{ playerOne }}
          <div v-if="medalsPlayerOne" class="flex items-center">
            <i v-for="i in medalsPlayerOne" :key="i" class="fad fa-medal text-orange-400"></i>
          </div>
          <button @click.stop="openModal('one')" class="hidden group-hover:block">
            <i class="fa-xs fal fa-pen"></i>
          </button>
        </div>
        <h1 class="text-white text-4xl">{{ scorePlayerOne }}</h1>
      </div>

      <!-- Control Buttons -->
      <div
        class="flex gap-4 items-center absolute top-1/2 -translate-y-1/2 -translate-x-1/2 left-1/2 bg-slate-50 w-fit rounded-xl h-10 px-4">
        <button class="flex gap-1 items-center text-slate-700" @click="undo">
          <i class="active:scale-90 fal fa-lg fa-rotate-left"></i></button>
        <button class="flex gap-1 items-center text-slate-700" @click="flipPositions">
          <i class="active:scale-90 fal fa-lg fa-arrow-up-arrow-down"></i></button>
        <button class="flex gap-1 items-center text-slate-700" @click="resetScores">
          <i class="active:scale-90 fal fa-lg fa-rotate"></i></button>
        <button class="flex gap-1 items-center text-slate-700" @click="showSettings = true">
          <i class="active:scale-90 fal fa-lg fa-gear"></i></button>
      </div>

      <!-- Player 2 -->
      <div :class="[colorPlayerTwo, 'flex-1 flex items-center justify-center sm:rounded-b-xl']"
        @click="incrementScore('two')">
        <h1 class="text-white text-4xl">{{ scorePlayerTwo }}</h1>
        <div class="absolute bottom-8 bg-slate-100 rounded-xl px-4 py-2 flex gap-2 items-center group">
          {{ playerTwo }}
          <div v-if="medalsPlayerTwo" class="flex items-center">
            <i v-for="i in medalsPlayerTwo" :key="i" class="fad fa-medal text-orange-400"></i>
          </div>
          <button @click.stop="openModal('two')" class="hidden group-hover:block">
            <i class="fa-xs fal fa-pen"></i>
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal untuk ganti nama -->
  <div v-if="showModal" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50">
    <div class="bg-white p-6 rounded-lg relative">
      <button @click="showModal = false" class="absolute -top-3 -right-2 bg-gray-500 text-white px-2 py-1 rounded-lg">
        <fal class="fal fa-xmark"></fal>
      </button>
      <h2 class="text-lg font-semibold mb-4">Change player name</h2>
      <div class="flex items-center">
        <input v-model="currentName" class="border p-2 w-full rounded-l-lg" />
        <button @click="saveName" class="bg-blue-500 text-white px-4 py-2 rounded-r-lg">
          <i class="active:scale-90 fa-xs fal fa-pen"></i>
        </button>
      </div>
    </div>
  </div>

  <!-- Modal untuk setting -->
  <div v-if="showSettings" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-60">
    <div class="bg-white p-6 rounded-lg relative">
      <button @click="showSettings = false" class="absolute -top-3 -right-2 bg-gray-500 text-white px-2 py-1 rounded-lg">
        <fal class="fal fa-xmark"></fal>
      </button>
      <h2 class="text-lg font-semibold mb-4">Match Settings</h2>
      <ul class="flex flex-col mb-4">
        <li class="flex justify-between border-b py-2 gap-8">
          <label class="flex items-center gap-2">
            <div class="text-center rounded-md bg-slate-200 px-2 w-8">
              <i class="fad  fa-star-circle"></i>
            </div>
            Score to win
          </label>
          <div class="flex gap-2 items-center">
            <button class="bg-gray-200 px-2 py-0 rounded" @click="maxScore--"><i class="active:scale-90 fal fa-minus"></i></button>
            <span class="w-5 text-center">{{ maxScore }}</span>
            <button class="bg-gray-200 px-2 py-0 rounded" @click="maxScore++"><i class="active:scale-90 fal fa-plus"></i></button>
          </div>
        </li>
        <li class="flex justify-between border-b py-2 gap-8">
          <label class="flex items-center gap-2">
            <div class="text-center rounded-md bg-slate-200 px-2 w-8">
              <i class="fad fa-sm fa-trophy-star"></i>
            </div>
            Round of game
          </label>
          <div class="flex gap-2 items-center">
            <button class="bg-gray-200 px-2 py-0 rounded" @click="maxRound--"><i class="active:scale-90 fal fa-minus"></i></button>
            <span class="w-5 text-center">{{ maxRound }}</span>
            <button class="bg-gray-200 px-2 py-0 rounded" @click="maxRound++"><i class="active:scale-90 fal fa-plus"></i></button>
          </div>
        </li>
        <li class="flex justify-between border-b py-2 gap-8">
          <label class="flex items-center gap-2">
            <div class="text-center rounded-md bg-slate-200 px-2 w-8">
              <i class="active:scale-90 fal fa-sm fa-asterisk"></i>
            </div>
            Enable deuce
          </label>
          <input type="checkbox" v-model="deuceEnabled" />
        </li>
      </ul>
    </div>
  </div>

  <!-- Modal pemenang -->
  <div v-if="winnerModal" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-60">
    <div class="bg-white p-6 rounded-lg text-center">
      <h2 class="text-2xl font-bold mb-4 text-slate-700">Congratulations!</h2>
      <p class="text-lg mb-4 flex gap-2 items-center">
        <i class="fad fa-trophy text-orange-500"></i>{{ medalsPlayerOne > medalsPlayerTwo ? playerOne : playerTwo }} wins
        the game!
      </p>
      <button @click="resetGame" class="bg-green-500 text-white px-4 py-2 rounded-lg active:scale-90">Play Again</button>
    </div>
  </div>
</template>


