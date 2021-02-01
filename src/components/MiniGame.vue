<script>
import startCase from 'lodash/startCase'
import { computed } from 'vue'

export default {
  props: {
    gameId: {
      type: String,
      required: true
    }
  },
  emits: ['select-screen'],
  setup(props, { emit }) {
    const gameTitle = computed(() => {
      return startCase(props.gameId)
    })
    const returnToHomeScreen = () => {
      emit('select-screen', 'Home')
    }
    return {
      gameTitle,
      returnToHomeScreen
    }
  }
}
</script>

<template>
  <section class="mini-game" id="mini-game">
    <slot name="progress" />
    <h1>{{ gameTitle }}</h1>
    <slot />
    <div class="mini-game-cta">
      <button class="nes-btn" @click="returnToHomeScreen">
        Back to Home
      </button>
    </div>
  </section>
</template>

<style>
.mini-game {
  position: relative;
  padding: 0 2rem;
}

.mini-game h1 {
  font-size: 4rem;
}

.mini-game-cta {
  margin-top: 45px;
  margin-bottom: 45px;
}
</style>
