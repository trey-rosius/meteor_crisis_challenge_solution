<script>
import shuffle from 'lodash/shuffle'
import { WIRE_OPTIONS } from '../constants'
import { reactive, computed, watch, toRefs, onMounted, onUnmounted } from 'vue'

export default {
  setup(props, { emit }) {
    const state = reactive({
      correctWires: computed(() => shuffle([...state.userWires])),
      userWins: computed(() => !state.matchStatus.includes(false)),
      userWires: [],
      matchStatus: [false, false, false, false],
      userSelection: {
        selectedWire: 'black',
        matchedWire: 'white'
      },
      wireLines: [],
      drawWire: {
        display: false,
        label: null,
        stroke: null,
        x1: 0,
        y1: 0,
        offsetLeft: 0,
        offsetTop: 0
      },
      mousePosition: {
        x: 0,
        y: 0
      }
    })
    watch(
      () => state.userWins,
      currentValue => {
        if (currentValue) {
          emit('mini-game-won', 'wire-game')
        }
      }
    )
    onMounted(() => {
      window.addEventListener('mousemove', state.handleMouseMove)
      state.userWires = shuffle(WIRE_OPTIONS)
    })
    onUnmounted(() => {
      window.removeEventListener('mousemove', state.handleMouseMove)
    })

    const handleMouseMove = $event => {
      state.mousePosition.x = $event.x
      state.mousePosition.y = $event.y
    }
    const checkWireColors = () => {
      const { selectedWire, matchedWire } = state.userSelection

      if (selectedWire === matchedWire) {
        const selectedIndex = state.userWires.findIndex(
          wire => wire.label === selectedWire
        )

        state.matchStatus[selectedIndex] = true
        state.wireLines.push({
          ...state.drawWire,
          x2: state.mousePosition.x - state.drawWire.offsetLeft,
          y2: state.mousePosition.y - state.drawWire.offsetTop
        })
      }

      state.drawWire.display = false
    }
    const registerMatchColor = wire => {
      state.userSelection.matchedWire = wire.label
    }
    const registerWireColor = ($event, wire) => {
      state.userSelection.selectedWire = wire.label

      const wireIndex = state.userWires.findIndex(userWire => userWire === wire)
      const elWireRect = $event.target.getBoundingClientRect()

      state.drawWire = {
        display: true,
        label: wire.label,
        stroke: wire.label,
        x1: 0,
        y1: 30 + 50 * wireIndex,
        offsetLeft: elWireRect.left + elWireRect.width,
        offsetTop: elWireRect.top - 50 * wireIndex
      }
    }
    const findCorrectWire = wire => {
      return state.correctWires.findIndex(
        correctWire => correctWire.label === wire.label
      )
    }

    return {
      ...toRefs(state),
      handleMouseMove,
      checkWireColors,
      registerMatchColor,
      registerWireColor,
      findCorrectWire
    }
  }
}
</script>

<template>
  <article class="wire-game" id="wire-game">
    <p class="wire-game-description">
      Instructions: Click on the color on the left to select the wire, then
      click on the matching color to connect the power.
    </p>
    <div class="wire-game-status">
      <p class="wire-game-status-content" :class="userWins ? 'is-active' : ''">
        <i class="nes-icon coin is-medium" :class="userWins ? '' : 'is-off'" />
        <span>Power {{ userWins ? 'On' : 'Off' }}</span>
      </p>
    </div>
    <div class="wire-game-wireboard">
      <div class="wire-game-panel">
        <ul>
          <li v-for="wire in userWires" :key="`user-${wire.label}`">
            <button
              class="nes-btn"
              @mousedown="registerWireColor($event, wire)"
              :style="`background-color: ${wire.label};`"
              :id="`wire-${wire.label}`"
            >
              {{ wire.label }}
            </button>
          </li>
        </ul>
      </div>

      <div class="wire-game-panel" style="flex: 1;">
        <svg class="wire-game-svg">
          <line
            v-show="drawWire.display"
            :key="`wire-line-draw`"
            class="wire-game-line"
            :x1="drawWire.x1"
            :y1="drawWire.y1"
            :x2="mousePosition.x - drawWire.offsetLeft"
            :y2="mousePosition.y - drawWire.offsetTop"
            :stroke="drawWire.stroke"
          />
          <line
            v-for="wire in wireLines"
            :key="`wire-line-${wire.label}`"
            class="wire-game-line"
            :x1="wire.x1"
            :y1="wire.y1"
            :x2="wire.x2"
            :y2="wire.y2"
            :stroke="wire.stroke"
          />
        </svg>
      </div>

      <div class="wire-game-panel">
        <ul>
          <button
            v-for="wire in correctWires"
            class="nes-btn"
            :key="`user-${wire.label}`"
            :style="`background-color: ${wire.label}`"
            @mouseenter="registerMatchColor(wire)"
            @mouseup="checkWireColors"
            :id="`wire-${wire.label}-end`"
          >
            {{ wire.label }}
          </button>
        </ul>
      </div>
    </div>
  </article>
</template>

<style>
.wire-game-description.wire-game-description {
  margin-bottom: 2rem;
  font-size: 1rem;
  text-align: left;
}

.wire-game-status {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 2rem;
  margin-bottom: 1rem;
}

.wire-game-status .wire-game-status-content {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  border: 5px dashed #ccc;
}

.wire-game-status .wire-game-status-content.is-active {
  color: yellow;
  border-color: yellow;
}

.wire-game-status .nes-icon.coin {
  margin-right: 40px;
}

.wire-game-status .nes-icon.coin.is-off {
  filter: grayscale(1);
}

.wire-game .wire-game-svg {
  width: 100%;
  height: 100%;
}

.wire-game-line {
  stroke-width: 10px;
}

.wire-game-wireboard {
  display: flex;
}

.wire-game-panel ul,
.wire-game-panel li {
  list-style: none;
  margin: 0;
  padding: 0;
}

.wire-game-panel ul {
  display: grid;
}

.wire-game-panel .nes-btn {
  width: 100%;
}
</style>
