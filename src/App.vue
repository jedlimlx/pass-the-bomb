<script setup>
import Title from './components/Title.vue'
import Rule from "./components/Rule.vue"

import foodThemes from "./assets/themes/food.txt?raw"
import natureThemes from "./assets/themes/nature.txt?raw"
import geographyThemes from "./assets/themes/geography.txt?raw"
import gamesThemes from "./assets/themes/games.txt?raw"
import mediaThemes from "./assets/themes/media.txt?raw"
import sportsThemes from "./assets/themes/sports.txt?raw"
import medicalThemes from "./assets/themes/medical.txt?raw"
import miscThemes from "./assets/themes/misc.txt?raw"
import {ref} from "vue";

// loading themes
const options = {
  "Food 🍝": foodThemes,
  "Nature 🌱": natureThemes,
  "Geography 🌏": geographyThemes,
  "Games 🎲": gamesThemes,
  "Media 🎥": mediaThemes,
  "Sports 🏀": sportsThemes,
  "Medical 💉": medicalThemes,
  "Miscellaneous": miscThemes
}

let namesString = ""
let enabledThemes = Object.keys(options)

function randomElement(lst) {
  return lst[Math.floor(Math.random() * lst.length)]
}

// loading sounds
const tickingSound = new Audio("/clock.ogg")
tickingSound.loop = true

const bombSound = new Audio("/bomb.ogg")

// loading other variables
const windowWidth = ref(window.innerWidth)

const gameStarted = ref(false)
const gameOver = ref(false)

const startingName = ref()
const theme = ref()

const time = ref("1:30")
const timePlusMinus = ref("0:30")

async function startGame() {
  windowWidth.value = window.innerWidth

  gameStarted.value = true
  gameOver.value = false

  const names = namesString.split("\n")
  startingName.value = randomElement(names)

  let themes = []
  enabledThemes.forEach((type) => {
    themes.push(...options[type].split("\n"))
  })

  theme.value = randomElement(themes)

  // setting a time limit for the bomb
  const averageTime = parseInt(time.value.split(":")[0]) * 60 + parseInt(time.value.split(":")[1])
  const timeRange = parseInt(timePlusMinus.value.split(":")[0]) * 60 + parseInt(timePlusMinus.value.split(":")[1])
  const bombTiming = averageTime + (Math.random() - 0.5) * 2 * timeRange

  await tickingSound.play()
  await new Promise(r => setTimeout(r, bombTiming * 1000))

  // bomb explodes
  gameOver.value = true
  tickingSound.pause()
  await bombSound.play()
}

function endGame() {
  gameStarted.value = false
  tickingSound.pause()
}

</script>

<template>
  <link
    rel="stylesheet"
    href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200&icon_names=arrow_range,avg_time,category,favorite,person,rule"
  />
  <main>
    <a id="top" style="pointer-events: none; all: unset;">
      <Title msg="Pass the Bomb 💣" style="grid-area: header"/>
    </a>

    <div class="setup" style="grid-area: setup" v-show="!gameStarted || windowWidth >= 1024">
      <Rule>
        <template #icon>
          <span class="material-symbols-outlined">
            rule
          </span>
        </template>
        <template #heading>How to Play?</template>

        Open this website on someone's phone. Then, start the bomb ticking!

        Each person has to say a word related to the theme shown.
        After the person has said the word, the phone can be passed to the next person.

        After an unknown time interval, the bomb will explode and the person who is holding onto the bomb at that time, loses.
      </Rule>

      <Rule>
        <template #icon>
          <span class="material-symbols-outlined">
            person
          </span>
        </template>
        <template #heading>Names</template>

        Enter your names below! The person who starts is randomly chosen. Enter one name per line.

        <br>
        <br>

        <div class="textarea-container">
          <FloatLabel variant="on">
              <Textarea v-model="namesString" class="names-textarea" id="names"
                        autoResize rows="10"/>
              <label for="names">Enter names below</label>
          </FloatLabel>
        </div>
      </Rule>

      <Rule>
        <template #icon>
          <span class="material-symbols-outlined">
            category
          </span>
        </template>
        <template #heading>Themes</template>

        Select which categories of themes you would like to include.

        <br>
        <br>

        <Listbox v-model="enabledThemes" :options="Object.keys(options)" checkmark multiple/>

      </Rule>

      <Rule>
        <template #icon>
          <span class="material-symbols-outlined">
            avg_time
          </span>
        </template>
        <template #heading>Time</template>

        Adjust the time taken for the bomb to explode.

        <br>
        <br>

        <InputGroup>
          <IftaLabel>
            <InputMask id="avg_time" v-model="time" mask="9:99" placeholder="1:30" />
            <label for="avg_time">Average Time</label>
          </IftaLabel>
          <IftaLabel>
            <InputMask id="time_range" v-model="timePlusMinus" mask="9:99" placeholder="0:30" />
            <label for="time_range">Time Range</label>
          </IftaLabel>
        </InputGroup>
      </Rule>

      <Rule>
        <template #icon>
          <span class="material-symbols-outlined">
            favorite
          </span>
        </template>
        <template #heading>Have Fun!</template>

        You can start the game by pressing the button below.

        <br>
        <br>

        <a href="#top">
          <Button label="Start Game" @click="startGame"></Button>
        </a>
      </Rule>
    </div>

    <div class="game" v-show="gameStarted" style="grid-area: game">
      <Card :class="{ shake: gameOver }">
        <template #title>
          <div class="card-title-container">
            <h3 style="font-weight: 500; margin: 0;">{{ startingName }} starts...</h3>
            <ProgressSpinner
              style="width: 50px; height: 50px" strokeWidth="6" fill="transparent"
              animationDuration="1s" class="color-changing-spinner" v-show="!gameOver"
            />
          </div>
        </template>
        <template #content>
          <h1 style="font-weight: 350; font-size: 2rem" v-show="!gameOver">{{ theme }}</h1>
          <h1 style="font-weight: 700; font-size: 3rem" v-show="gameOver">BOOM! 💥</h1>
        </template>
        <template #footer>
          <div class="card-button-container">
            <Button label="Back to Menu" variant="outlined" @click="endGame" class="card-button"/>
            <Button label="Skip" class="card-button" v-show="!gameOver" @click="startGame"/>
            <Button label="Next" class="card-button" v-show="gameOver" @click="startGame"/>
          </div>
        </template>
      </Card>
    </div>
  </main>
</template>

<style scoped>
main {
  display: contents;
}

.setup {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
  width: 100%;
  max-width: 1024px;
  margin: 0 auto;
  padding: 0.5rem;
}

.textarea-container {
  display: flex;
  flex-direction: column;
}

.names-textarea {
  flex-grow: 1;
  width: 100%;
  resize: none;
}

.card-button {
  margin-left: 5px;
  margin-right: 5px;
}

.card-button-container {
  display: flex;
  justify-content: space-between;
}

.card-title-container {
  display: flex;
  justify-content: space-between;
}

.card-title-container h3 {
  flex: 1;
  margin-right: 1rem;
}

@keyframes shake {
  0% { transform: translate(2px, 2px) rotate(0deg); }
  10% { transform: translate(-2px, -4px) rotate(-2deg); }
  20% { transform: translate(-6px, 0px) rotate(2deg); }
  30% { transform: translate(6px, 4px) rotate(0deg); }
  40% { transform: translate(2px, -2px) rotate(2deg); }
  50% { transform: translate(-2px, 4px) rotate(-2deg); }
  60% { transform: translate(-6px, 2px) rotate(0deg); }
  70% { transform: translate(6px, 2px) rotate(-2deg); }
  80% { transform: translate(-2px, -2px) rotate(2deg); }
  90% { transform: translate(2px, 4px) rotate(0deg); }
  100% { transform: translate(2px, -4px) rotate(-2deg); }
}

.shake {
  animation: shake 0.1s;
  animation-iteration-count: 2;
}
</style>
