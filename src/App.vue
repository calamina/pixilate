<script setup lang="ts">
import p5 from "p5"
import { ref } from "vue"

const toggleMenu = ref(true)
// TODO : COLOR SWAP !!
const COLORS = ["#F28585", "#FFA447", "#FFFC9B", "#B7E5B4", "#555", "#242424"]
const MODES = [
  "adn",
  "adn reverse",
  "adn messy",
  "adn messier",
  "adn pretty",
  "path",
  "path messy",
  "path grid",
  "mountains",
  "sun",
  "fill fall",
  "fill fall random"
]

let ACTIVE_MODE = ref("path grid")
let SIZE = ref(2)
let DISTANCE = ref(10)
let SPREAD = ref(10)

const FRAMERATE = 60
const RESET = 400
const PADDING = 50
let canvas: HTMLElement | null = null

let cols: number, rows: number
let alive: Array<any> = []
let active = []
let state = []
let counter = 0

// CONTROLS
let isDrawing = ref(false)
let isInstant = ref(false)
let isRegularShape = ref(false)
let isPaused = ref(false)
let mode: any


let sketch = function (p: any) {
  p.setup = function () {
    canvas = p.createCanvas(p.windowWidth - PADDING, p.windowHeight - PADDING)
    p.noStroke()
    p.frameRate(FRAMERATE)
    p.background(36, 36, 36)

    // CELLS
    cols = p.ceil(p.width / SIZE.value)
    rows = p.ceil(p.height / SIZE.value)
  }

  p.draw = function () {
    if (isInstant.value) {
      isInstant.value = false
      for (let i = 0; i < 3000; i++) {
        pixelate()
      }
    }

    isDrawing.value ? drawCurrent() : pixelate()
  }

  function toggleDraw() {
    isDrawing.value = !isDrawing.value
    if (isDrawing.value && isPaused.value) {
      playpause()
    }
  }

  function toggleInstant() {
    isInstant.value = !isInstant.value
    p.redraw()
  }

  function toggleShape() {
    isRegularShape.value = !isRegularShape.value
    p.reset()
  }

  function sizePlus() {
    if (SIZE.value < 20) SIZE.value++
    cols = p.ceil(p.width / SIZE.value)
    rows = p.ceil(p.height / SIZE.value)
  }

  function sizeMinus() {
    if (SIZE.value > 1) SIZE.value--
    cols = p.ceil(p.width / SIZE.value)
    rows = p.ceil(p.height / SIZE.value)
  }

  function distancePlus() {
    if (DISTANCE.value < 40) DISTANCE.value++
  }

  function distanceMinus() {
    if (DISTANCE.value > 1) DISTANCE.value--
  }

  function gapPlus() {
    if (SPREAD.value < 40) SPREAD.value++
  }

  function gapMinus() {
    if (SPREAD.value > 0) SPREAD.value--
  }

  function saveCanvas() {
    p.save(canvas, `pixel-${Date.now()}.png`)
  }

  function reset() {
    p.background(36, 36, 36)
    alive = active = []
    counter = 0
  }

  function swapShape() {
    isRegularShape.value = !isRegularShape.value
  }

  function playpause() {
    isPaused.value = !isPaused.value
    isPaused.value ? p.noLoop() : p.loop()
  }

  p.windowResized = function () {
    p.resizeCanvas(p.windowWidth - PADDING, p.windowHeight - PADDING)
    cols = p.ceil(p.width / SIZE.value)
    rows = p.ceil(p.height / SIZE.value)
  }

  function drawCurrent() {
    let mCol = p.floor(p.mouseX / SIZE.value)
    let mRow = p.floor(p.mouseY / SIZE.value)
    p.fill(COLORS[Math.floor(Math.random() * 5)])

    neighborCells(mCol, mRow).forEach((i) => {
      p.square(SIZE.value * i[0], SIZE.value * i[1], SIZE.value)
    })
    // active = [mouseX, mouseY];
  }

  function pixelate() {
    let x
    let y

    // if(counter > RESET) {
    //   noLoop()
    // }

    if (alive.length && counter <= RESET) {
      mode = ACTIVE_MODE.value
      // if(noPersist) {
      // fill(COLORS[5]);
      // state.forEach((cell) => {
      //   square(SIZE * cell[0], SIZE * cell[1], SIZE);
      // });
      // }
      setBorders()

      if (mode === "adn") {
        x =
          alive[0] -
          Math.floor(
            Math.random() < 0.5 ? -1 : 1 * Math.floor(Math.random() * DISTANCE.value)
          )
        y =
          alive[1] -
          Math.floor(
            Math.random() < 0.5 ? -1 : 1 * Math.floor(Math.random() * DISTANCE.value)
          )
      }
      if (mode === "adn reverse") {
        x =
          alive[0] +
          Math.floor(
            Math.random() < 0.5 ? -1 : 1 * Math.floor(Math.random() * DISTANCE.value)
          )
        y =
          alive[1] +
          Math.floor(
            Math.random() < 0.5 ? -1 : 1 * Math.floor(Math.random() * DISTANCE.value)
          )
      }
      if (mode === "adn messy") {
        x =
          alive[0] -
          Math.floor((Math.random() - 0.5) * 2 * Math.random() * DISTANCE.value)
        y =
          alive[1] -
          Math.floor((Math.random() - 0.5) * 2 * Math.random() * DISTANCE.value)
      }
      if (mode === "adn messier") {
        x = alive[0] - Math.floor((Math.random() - 0.5) * 2 * DISTANCE.value)
        y = alive[1] - Math.floor((Math.random() - 0.5) * 2 * DISTANCE.value)
      }

      if (mode === "adn pretty") {
        // x = sin(alive[0]) * DISTANCE.value + alive[0];
        x = alive[0] + 1 + p.sin(p.frameCount / DISTANCE.value) * 4
        y = alive[1] + DISTANCE.value * p.noise(1)
      }

      if (mode === "path") {
        x = alive[0] + DISTANCE.value * Math.random() * (Math.random() < 0.5 ? -1 : 1)
        y = alive[1] + DISTANCE.value * Math.random() * (Math.random() < 0.5 ? -1 : 1)
        counter++
      }

      if (mode === "path messy") {
        x = alive[0] + Math.round(DISTANCE.value * ((Math.random() - 0.5) * 2))
        y = alive[1] + Math.round(DISTANCE.value * ((Math.random() - 0.5) * 2))
        counter++
      }

      if (mode === "path grid") {
        x = alive[0] + DISTANCE.value * p.noise(1) * (Math.random() < 0.5 ? -1 : 1)
        y = alive[1] + DISTANCE.value * p.noise(1) * (Math.random() < 0.5 ? -1 : 1)
        counter++
      }

      if (mode === "sun") {
        x = alive[0] * (1 + p.noise(0))
        y = alive[1] * (1 + p.noise(0))
        counter++
      }

      if (mode === "mountains") {
        x = alive[0] + p.noise(1) * DISTANCE.value
        y = alive[1] + DISTANCE.value * p.noise(1) * (Math.random() < 0.5 ? -1 : 1)
        counter++
      }

      if (mode === "fill fall") {
        x = alive[1] > 7 ? alive[0] : alive[0] + DISTANCE.value
        y = alive[1] + DISTANCE.value
      }

      if (mode === "fill fall random") {
        x = alive[1] > 7 ? alive[0] : Math.random() * cols
        y = alive[1] + DISTANCE.value
      }

      // else{
      //   // mountains
      //   x = alive[0] + p.noise(1) * DISTANCE.value;
      //   y = alive[1] + DISTANCE.value * p.noise(10000) * (Math.random() < 0.5 ? -1 : 1);
      //   counter++;
      // }

      // // small mountains
      // x = alive[0] + noise(100) * 2 * DISTANCE.value;
      // y = alive[1] + DISTANCE.value *  noise(1) * (Math.random() < 0.5 ? -1 : 1);
      // counter++;

      // // path grid
      // x = alive[0] + Math.floor(DISTANCE.value * 2 *  noise(1)) * (Math.random() < 0.5 ? -1 : 1);
      // y = alive[1] + Math.floor(DISTANCE.value * 2 *  noise(1)) * (Math.random() < 0.5 ? -1 : 1);
      // counter++;

      // // adn grid up-left
      // x = alive[0] - DISTANCE.value * (noise(alive[0]));
      // y = alive[1] - DISTANCE.value * (noise(alive[1]));

      // // random position
      // x = cols * noise(SIZE * alive[0]);
      // y = rows * noise(SIZE * alive[1]);

      // // regular up-left
      // x = alive[0] - DISTANCE.value * noise(1);
      // y = alive[1] - DISTANCE.value * noise(1);

      // fill right
      // x = alive[0] + DISTANCE.value
      // y = x > 20 ? alive[1] : (alive[1] + DISTANCE.value);
    } else {
      x = Math.floor(Math.random() * cols)
      y = Math.floor(Math.random() * rows)
      counter = 0
    }

    p.noStroke()
    p.fill(COLORS[Math.floor(Math.random() * 5)])

    neighborCells(x, y).forEach((cell) => {
      p.square(SIZE.value * cell[0], SIZE.value * cell[1], SIZE.value)
    })

    alive = [x, y]
  }

  function setBorders() {
    if (alive[0] >= cols) {
      alive[0] = 1
    }
    if (alive[1] >= rows) {
      alive[1] = 1
    }
    if (alive[0] <= 0) {
      alive[0] = cols
    }
    if (alive[1] <= 0) {
      alive[1] = rows
    }
  }

  function northIndex(x: number, y: number) {
    // TODO add more shapes (diamond, circle, etc)
    return isRegularShape.value
      ? [x, y - SPREAD.value]
      : [x, y - Math.floor(Math.random() * SPREAD.value)]
  }
  function southIndex(x: number, y: number) {
    return isRegularShape.value
      ? [x, y + SPREAD.value]
      : [x, y + Math.floor(Math.random() * SPREAD.value)]
  }
  function westIndex(x: number, y: number) {
    return isRegularShape.value
      ? [x - SPREAD.value, y]
      : [x - Math.floor(Math.random() * SPREAD.value), y]
  }

  function eastIndex(x: number, y: number) {
    return isRegularShape.value
      ? [x + SPREAD.value, y]
      : [x + Math.floor(Math.random() * SPREAD.value), y]
  }

  function neighborCells(x: number, y: number) {
    let neighbors = []
    // TODO add button to toggle center OR neighbors ?
    neighbors.push([x, y])
    neighbors.push(northIndex(x, y))
    neighbors.push(southIndex(x, y))
    neighbors.push(eastIndex(x, y))
    neighbors.push(westIndex(x, y))
    state = neighbors
    return neighbors
  }

  return {
    sizePlus,
    sizeMinus,
    distancePlus,
    distanceMinus,
    gapPlus,
    gapMinus,
    reset,
    toggleDraw,
    saveCanvas,
    toggleInstant,
    playpause,
    swapShape
  }
}

let myp5 = new p5(sketch)
</script>

<template>
  <main></main>
  <div class="menu">
    <button @click="toggleMenu = !toggleMenu">
      <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 6h16"></path><path d="M4 12h16"></path><path d="M4 18h16"></path></g></svg>
      {{ toggleMenu ? 'Hide Menu' : 'Show Menu' }}
    </button>
    <div class="buttons" v-if="toggleMenu">
      <div class="title">
        <h3>
          <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="8" y="4" width="12" height="12" rx="2"></rect><rect x="4" y="8" width="12" height="12" rx="2"></rect></g></svg> 
          mode
        </h3>
      </div>
      <select name="mode" id="" v-model="ACTIVE_MODE" @change="console.debug(ACTIVE_MODE)">
        <option v-for="mode in MODES" :value="mode">{{ mode }}</option>
      </select>
      <div class="title">
        <h3>
          <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="8" y="4" width="12" height="12" rx="2"></rect><rect x="4" y="8" width="12" height="12" rx="2"></rect></g></svg> 
          generation
      </h3>
      </div>
      <button @click="sketch(myp5).toggleInstant()">
        <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="4.5" cy="9.5" r="1"></circle><circle cx="9.5" cy="4.5" r="1"></circle><circle cx="9.5" cy="14.5" r="1"></circle><circle cx="4.5" cy="19.5" r="1"></circle><circle cx="14.5" cy="9.5" r="1"></circle><circle cx="19.5" cy="4.5" r="1"></circle><circle cx="14.5" cy="19.5" r="1"></circle><circle cx="19.5" cy="14.5" r="1"></circle></g></svg>
        populate
      </button>
      <button :class="{active: isDrawing}" @click="sketch(myp5).toggleDraw()">
        <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 7H6a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h9a2 2 0 0 0 2-2v-3"></path><path d="M9 15h3l8.5-8.5a1.5 1.5 0 0 0-3-3L9 12v3"></path><path d="M16 5l3 3"></path></g></svg>
        {{ isDrawing ? 'drawing' : 'draw' }}
      </button>
      <button @click="sketch(myp5).swapShape()">
        <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="9.5" cy="9.5" r="6.5"></circle><rect x="10" y="10" width="11" height="11" rx="2"></rect></g></svg>
        change shape
      </button>
      <div class="title">
        <h3>
          <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="4" y="4" width="16" height="16" rx="2"></rect><circle cx="12" cy="12" r="1"></circle></g></svg>
          pixel size
        </h3>
      </div>
      <div class="plusminus">
        <span>{{ SIZE < 10 ? '0' + SIZE : SIZE }}</span>
        <button @click="sketch(myp5).sizeMinus()" :disabled="SIZE === 1">-</button>
        <button @click="sketch(myp5).sizePlus()" :disabled="SIZE === 10">+</button>
      </div>
      <div class="title">
        <h3>
          <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="7" r="4"></circle><circle cx="6.5" cy="17" r="4"></circle><circle cx="17.5" cy="17" r="4"></circle></g></svg>
          gap
        </h3>
      </div>
      <div class="plusminus">
        <span>{{ DISTANCE < 10 ? '0' + DISTANCE : DISTANCE }}</span>
        <button @click="sketch(myp5).distanceMinus()" :disabled="DISTANCE === 1">-</button>
        <button @click="sketch(myp5).distancePlus()" :disabled="DISTANCE === 40">+</button>
      </div>
      <div class="title">
        <h3>
          <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="4" y="4" width="16" height="16" rx="2"></rect><path d="M8 16v.01"></path><path d="M8 12v.01"></path><path d="M8 8v.01"></path><path d="M16 16v.01"></path><path d="M16 12v.01"></path><path d="M16 8v.01"></path><path d="M12 8v.01"></path><path d="M12 16v.01"></path></g></svg>
          shape spread
        </h3>
      </div>
      <div class="plusminus">
        <span>{{ SPREAD < 10 ? '0' + SPREAD : SPREAD }}</span>
        <button @click="sketch(myp5).gapMinus()" :disabled="SPREAD === 0">-</button>
        <button @click="sketch(myp5).gapPlus()" :disabled="SPREAD === 40">+</button>
      </div>
      <div class="title">
        <h3>
          <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 0 0 2.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 0 0 1.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 0 0-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 0 0-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 0 0-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 0 0-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 0 0 1.066-2.573c-.94-1.543.826-3.31 2.37-2.37c1 .608 2.296.07 2.572-1.065z"></path><circle cx="12" cy="12" r="3"></circle></g></svg>
          controls
        </h3>
      </div>
      <button :class="{active: !isPaused}" @click="sketch(myp5).playpause()">
        <svg v-if="!isPaused" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><path d="M7 4v16l13-8z" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg>
        <svg v-if="isPaused" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="6" y="5" width="4" height="14" rx="1"></rect><rect x="14" y="5" width="4" height="14" rx="1"></rect></g></svg>
        {{ !isPaused ? 'playing' : 'paused' }}
      </button>
      <button @click="sketch(myp5).saveCanvas()">
        <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M6 4h10l4 4v10a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2"></path><circle cx="12" cy="14" r="2"></circle><path d="M14 4v4H8V4"></path></g></svg>
        save
      </button>
      <button @click="sketch(myp5).reset()">
        <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M15 4.55a8 8 0 0 0-6 14.9M9 15v5H4"></path><path d="M18.37 7.16v.01"></path><path d="M13 19.94v.01"></path><path d="M16.84 18.37v.01"></path><path d="M19.37 15.1v.01"></path><path d="M19.94 11v.01"></path></g></svg>
        empty
      </button>
    </div>
  </div>
</template>

<style scoped lang="scss">
main {
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 100vw;
  min-height: 100vh;
}

.menu {
  position: fixed;
  bottom: 24px;
  left: 24px;
  background-color: #242424;
  color: #555;
  border-top: 1px solid #555;
  border-right: 1px solid #555;
  border-left: 1px solid #242424;
  border-bottom: 1px solid #242424;
  padding: 1.5rem 1.5rem 0 0;
  padding: 0.5rem 0.5rem 0 0;
  display: flex;
  flex-flow: column;
  gap: 0.5rem;
}

svg {
  width: 1rem;
  height: 1rem;
  fill: currentColor;
}

.buttons {
  gap: 0.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.title {
  width: 100%;
  display: flex;
  justify-content: space-between;
  padding-top: 0.5rem;
}

.plusminus {
  width: 100%;
  display: flex;
  gap: 0.5rem;

  span, button {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  button {
    width: fit-content;
  }
}

span,
h3,
p {
  margin: 0;
  display: flex;
  align-items: center;
  font-family: monospace;
  font-size: 1rem;
  font-weight: 200;
}

span, select, .active {
  color: v-bind('COLORS[0]') !important;
}

.active {
  border-color: v-bind('COLORS[0]');
}

p {
  width: 4ch;
  width: 15ch;
}

h3 {
  display: flex;
  gap: 0.25rem;
  padding-top: 0.25rem;
  width: 100%;
}

button,
select,
option, 
span {
  padding: 0.4rem 0.75rem;
  width: 100%;
  background-color: #242424;
  color: #555;
  border: 1px solid #555;
  cursor: pointer;
  text-align: left;
  font-family: monospace;
  font-size: 1rem;

  &:disabled {
    opacity: 0.4;
  }
}

span {
  display: flex;
  align-items: center;
  justify-content: center;
}

button {
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

input[type="range"] {
  width: 100%;
  background-color: #242424;
  color: #555;
  border: 1px solid #555;
  cursor: pointer;
}
</style>
