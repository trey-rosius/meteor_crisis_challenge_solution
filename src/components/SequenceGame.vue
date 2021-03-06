<script>
import shuffle from 'lodash/shuffle'
import { toRefs, computed, reactive, watch } from 'vue'
export default {
  setup(props, { emit }) {
    const state = reactive({
      correctSequence: computed(() => {
        return shuffle(['blue', 'green', 'red'])
      }),
      displaySequence: computed(() => {
        let defaultColors = [
          { label: 'black' },
          { label: 'black' },
          { label: 'black' }
        ]

        state.userSequence.forEach((item, index) => {
          defaultColors[index] = item
        })

        return defaultColors
      }),
      userWins: computed(() => {
        if (state.userSequence && state.userSequence.length === 3) {
          return state.userSequence.reduce((accumulator, currentValue) => {
            return accumulator && currentValue.matched
          }, true)
        } else {
          return false
        }
      }),

      gameStatus: 'In Progress',
      userSequence: [],
      displayValidation: false,
      colorOptions: [{ label: 'red' }, { label: 'blue' }, { label: 'green' }]
    })

    const addColorToSequence = color => {
      state.userSequence.push({
        ...color,
        matched: false
      })
    }
    const checkColorSequence = () => {
      state.userSequence.forEach((item, index) => {
        item.matched = item.label === state.correctSequence[index]
      })

      state.displayValidation = true

      if (state.userWins) {
        emit('mini-game-won', 'sequence-game')
      } else {
        setTimeout(() => {
          state.userSequence = []
          state.displayValidation = false
        }, 1000)
      }
    }
    watch(
      () => state.userSequence,
      currentValue => {
        if (currentValue.length === 3) {
          checkColorSequence()
        }
      },
      {
        deep: true
      }
    )

    return {
      ...toRefs(state),
      addColorToSequence,
      checkColorSequence
    }
  }
}
</script>

<template>
  <div>
    <section>
      <p class="sequence-game-description">
        Instructions: Figure out the correct color sequence using the input
        buttons.
      </p>
      <p class="sequence-game-status" :class="userWins ? 'is-green' : ''">
        <i
          class="nes-icon trophy is-medium"
          :class="userWins ? '' : 'is-off'"
        />
        <span class="sequence-game-status-text"
          >{{ userWins ? 'Correct' : 'Mystery' }} Sequence</span
        >
      </p>
      <div class="color-swatch-wrapper">
        <div
          v-for="(color, index) in displaySequence"
          :key="`color-${index}`"
          class="nes-container color-swatch"
          :style="`background-color: ${color.label};`"
        >
          <i
            class="color-swatch-heart nes-icon is-medium heart"
            :class="color.matched ? '' : 'is-empty'"
          ></i>
          <span class="color-swatch-text">
            {{ color.label === 'black' ? 'black' : color.label }}
          </span>
        </div>
      </div>
    </section>
    <section v-if="!userWins">
      <p class="sequence-directions">Enter Input</p>
      <div class="color-swatch-wrapper">
        <button
          v-for="(color, index) in colorOptions"
          :key="`color-${index}`"
          class="nes-container color-swatch"
          :style="`background-color: ${color.label};`"
          @click="addColorToSequence(color)"
        >
          {{ color.label }}
        </button>
      </div>
    </section>
  </div>
</template>

<style>
.color-swatch-heart {
  margin: 0;
  margin-bottom: 15px;
}

.color-swatch-text {
  margin-top: 15px;
}

.color-swatch-wrapper {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-column-gap: 10px;
  margin-bottom: 30px;
}

.color-swatch {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 1rem;
  border: 3px solid white;
  color: white;
  font-weight: bold;
}

.sequence-game-description.sequence-game-description {
  text-align: left;
  font-size: 1rem;
}

.sequence-game-status {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 1.5rem;
  padding: 1rem;
  color: #ccc;
  border: 5px dashed #ccc;
}

.sequence-game-status.is-green {
  border-color: var(--green);
}

.sequence-game-status .nes-icon.trophy.is-off {
  filter: grayscale(1);
}

.sequence-game-status-text {
  margin-left: 10px;
}

.sequence-directions {
  font-size: 1.5rem;
}
</style>
