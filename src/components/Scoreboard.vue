<script setup>
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
const currentPlayer = ref('')
const currentName = ref('')
const showSettings = ref(false)

const maxScore = ref(localStorage.getItem('maxScore') || 10)
const deuceEnabled = ref(localStorage.getItem('deuceEnabled') === 'true' || false)

// History untuk Undo
const history = ref([])

watch([maxScore, deuceEnabled], () => {
  localStorage.setItem('maxScore', maxScore.value)
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
</script>


<template>
  <div class="flex items-center justify-center h-[100svh]">
    <div class="h-full w-full sm:w-[40rem] sm:h-[40rem] flex flex-col relative">
      <!-- Player 1 -->
      <div :class="[colorPlayerOne, 'flex-1 flex items-center justify-center sm:rounded-t-xl']" @click="incrementScore('one')">
        <div class="absolute top-8 bg-slate-100 rounded-xl px-4 py-2 flex gap-2 items-center group">
          <i v-if="scorePlayerOne > scorePlayerTwo" class="fad fa-crown text-orange-400"></i>
          {{ playerOne }} ({{ medalsPlayerOne }})
          <button @click.stop="openModal('one')" class="hidden group-hover:block">
            <i class="fa-xs fal fa-pen"></i>
          </button>
        </div>
        <h1 class="text-white text-4xl">{{ scorePlayerOne }}</h1>
      </div>

      <!-- Control Buttons -->
      <div class="flex gap-4 items-center absolute top-1/2 -translate-y-1/2 -translate-x-1/2 left-1/2 bg-slate-100 w-fit rounded-2xl h-10 px-4">
        <button class="flex gap-1 items-center text-slate-700" @click="undo"><i class="fal fa-rotate-left"></i></button>
        <button class="flex gap-1 items-center text-slate-700" @click="flipPositions"><i class="fal fa-arrow-up-arrow-down"></i></button>
        <button class="flex gap-1 items-center text-slate-700" @click="resetScores"><i class="fal fa-rotate"></i></button>
        <button class="flex gap-1 items-center text-slate-700" @click="showSettings = true"><i class="fal fa-gear"></i></button>
      </div>

      <!-- Player 2 -->
      <div :class="[colorPlayerTwo, 'flex-1 flex items-center justify-center sm:rounded-b-xl']" @click="incrementScore('two')">
        <h1 class="text-white text-4xl">{{ scorePlayerTwo }}</h1>
        <div class="absolute bottom-8 bg-slate-100 rounded-xl px-4 py-2 flex gap-2 items-center group">
          <i v-if="scorePlayerTwo > scorePlayerOne" class="fad fa-crown text-orange-400"></i>
          {{ playerTwo }} ({{ medalsPlayerTwo }})
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
      <button @click="showModal = false" class="absolute -top-3 -right-2 bg-gray-500 text-white px-2 py-1 rounded-lg"><fal class="fal fa-xmark"></fal></button>
      <h2 class="text-lg font-semibold mb-4">Change player name</h2>
      <div class="flex items-center">
        <input v-model="currentName" class="border p-2 w-full rounded-l-lg" />
        <button @click="saveName" class="bg-blue-500 text-white px-4 py-2 rounded-r-lg"><div class="fa-xs fal fa-pen"></div></button>
      </div>
    </div>
  </div>

  <!-- Modal untuk setting -->
  <div v-if="showSettings" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50">
    <div class="bg-white p-6 rounded-lg relative">
      <button @click="showSettings = false" class="absolute -top-3 -right-2 bg-gray-500 text-white px-2 py-1 rounded-lg"><fal class="fal fa-xmark"></fal></button>
      <h2 class="text-lg font-semibold mb-4">Settings</h2>
      <div class="mb-4">
        <label class="block mb-1">Max Score:</label>
        <input v-model="maxScore" type="number" class="border p-2 w-full rounded-lg" />
      </div>
      <div class="mb-4">
        <label class="flex items-center">
          <input v-model="deuceEnabled" type="checkbox" class="mr-2" />
          Enable Deuce (2 point lead required)
        </label>
      </div>
      <button @click="showSettings = false" class="bg-blue-500 text-white px-4 py-2 rounded-lg">Save</button>
    </div>
  </div>
</template>

