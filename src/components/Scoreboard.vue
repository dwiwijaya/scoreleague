<script setup>
import { ref } from 'vue'

// State untuk skor dan pemain
const scorePlayerOne = ref(0)
const scorePlayerTwo = ref(0)
const playerOne = ref('Player 1')
const playerTwo = ref('Player 2')

// Warna latar belakang
const colorPlayerOne = ref('bg-red-500')
const colorPlayerTwo = ref('bg-blue-500')

// State untuk modal ganti nama
const showModal = ref(false)
const currentPlayer = ref('')
const currentName = ref('')

// History untuk Undo
const history = ref([])

// Fungsi untuk menambah skor
function incrementScore(player) {
  if (player === 'one') {
    scorePlayerOne.value++
    history.value.push({ player: 'one', score: scorePlayerOne.value })
  } else if (player === 'two') {
    scorePlayerTwo.value++
    history.value.push({ player: 'two', score: scorePlayerTwo.value })
  }
}

// Fungsi untuk undo
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

// Fungsi untuk reset skor
function resetScores() {
  scorePlayerOne.value = 0
  scorePlayerTwo.value = 0
  history.value = []
}

// Fungsi untuk flip posisi
function flipPositions() {
  // Tukar skor
  const tempScore = scorePlayerOne.value
  scorePlayerOne.value = scorePlayerTwo.value
  scorePlayerTwo.value = tempScore

  // Tukar nama pemain
  const tempPlayer = playerOne.value
  playerOne.value = playerTwo.value
  playerTwo.value = tempPlayer

  // Tukar warna
  const tempColor = colorPlayerOne.value
  colorPlayerOne.value = colorPlayerTwo.value
  colorPlayerTwo.value = tempColor
}

// Fungsi untuk membuka modal ganti nama
function openModal(player) {
  currentPlayer.value = player
  currentName.value = player === 'one' ? playerOne.value : playerTwo.value
  showModal.value = true
}

// Fungsi untuk menyimpan nama baru
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
      <!-- Bagian Player 1 -->
      <div :class="[colorPlayerOne, 'flex-1 flex items-center justify-center sm:rounded-t-xl']" @click="incrementScore('one')">
        <div class="absolute top-8 bg-slate-100 rounded-xl px-4 py-2 flex gap-2 items-center group">
          <i v-if="scorePlayerOne > scorePlayerTwo" class="fad fa-crown text-orange-400"></i>
          {{ playerOne }} 
          <button @click.stop="openModal('one')" class="hidden group-hover:block">
            <i class="fa-xs fal fa-pen"></i>
          </button>
        </div>
        <h1 class="text-white text-4xl">{{ scorePlayerOne }}</h1>
      </div>

      <!-- Tombol Undo, Flip, Reset -->
      <div
        class="flex gap-4 items-center absolute top-1/2 -translate-y-1/2 -translate-x-1/2 left-1/2 bg-slate-100 w-fit rounded-2xl h-10 px-4">
        <button class="flex gap-1 items-center text-slate-700" @click="undo"><i class="fal fa-rotate-left"></i></button>
        <button class="flex gap-1 items-center text-slate-700" @click="flipPositions"><i class="fal fa-arrow-up-arrow-down"></i></button>
        <button class="flex gap-1 items-center text-slate-700" @click="resetScores"><i class="fal fa-rotate"></i></button>
      </div>

      <!-- Bagian Player 2 -->
      <div :class="[colorPlayerTwo, 'flex-1 flex items-center justify-center sm:rounded-b-xl']" @click="incrementScore('two')">
        <h1 class="text-white text-4xl">{{ scorePlayerTwo }}</h1>
        <div class="absolute bottom-8 bg-slate-100 rounded-xl px-4 py-2 flex gap-2 items-center group">
          <i v-if="scorePlayerTwo > scorePlayerOne" class="fad fa-crown text-orange-400"></i>
          {{ playerTwo }} 
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
</template>
