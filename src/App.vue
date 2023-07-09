<script lang="ts" setup>
import GameHeader from './components/GameHeader.vue'
import GameFigure from './components/GameFigure.vue'
import GameWrongLetters from './components/GameWrongLetters.vue'
import GameWord from './components/GameWord.vue'
import GamePopup from './components/GamePopup.vue'
import GameNotification from './components/GameNotification.vue'
import { computed, ref, watch } from 'vue'

const getRandomWord = async () => {
  try {
    const data = await fetch('https://api.randomdatatools.ru/?unescaped=false&params=FirstName')
    const text = await data.json()
    console.log(text)
    word.value = text.FirstName.toLowerCase()
  } catch (err) {
    console.log(err)
    word.value = ''
  }
}

getRandomWord()

const word = ref('')
const letters = ref<string[]>([])
const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GamePopup> | null>(null)

const correctLetters = computed(() => {
  return letters.value.filter((l) => word.value.includes(l))
})
const wrongLetters = computed(() => {
  return letters.value.filter((l) => !word.value.includes(l))
})
const isLose = computed(() => wrongLetters.value.length === 6)
const isWin = computed(() => correctLetters.value.length === new Set(word.value.split('')).size)

watch(wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})

watch(correctLetters, () => {
  if (isWin.value) {
    popup.value?.open('win')
  }
})

window.addEventListener('keydown', ({ key }) => {
  if (isWin.value || isLose.value) {
    return
  }

  if (letters.value.includes(key)) {
    notification.value?.open()
    setTimeout(() => notification.value?.close(), 2000)
    return
  }

  if (/[а-яА-ЯёЁ]/.test(key)) {
    letters.value.push(key.toLowerCase())
  }
})

const restartGame = () => {
  letters.value = []
  popup.value?.close()
  getRandomWord()
}
</script>

<template>
  <GameHeader />

  <div class="game-container">
    <GameFigure :wrongLettersCount="wrongLetters.length" />

    <GameWrongLetters :wrongLetters="wrongLetters" />

    <GameWord :word="word" :correctLetters="correctLetters" />
  </div>

  <GamePopup ref="popup" @restart="restartGame" :word="word" />

  <GameNotification ref="notification" />
</template>
