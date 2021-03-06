<script>
import { PASSWORD_STATUS } from '../constants'
import { toRefs, computed, reactive, onMounted, watch } from 'vue'

export default {
  setup(props, { emit }) {
    const state = reactive({
      gameStatus: computed(() => {
        if (state.status === PASSWORD_STATUS.FAIL) {
          return {
            styles: 'is-red',
            text: 'Access Denied'
          }
        } else if (state.status === PASSWORD_STATUS.PASS) {
          return {
            styles: 'is-green',
            text: 'Access Granted'
          }
        } else {
          return {
            styles: '',
            text: 'Locked'
          }
        }
      }),
      userWins: computed(() => {
        return state.status === PASSWORD_STATUS.PASS
      }),
      status: 'In Progress',
      passwordInput: '',
      correctPassword: 0
    })
    const checkPassword = () => {
      if (state.correctPassword === state.passwordInput) {
        state.status = PASSWORD_STATUS.PASS
        emit('mini-game-won', 'password-game')
      } else {
        state.status = PASSWORD_STATUS.FAIL
      }
    }
    const generateNewPassword = () => {
      return Math.floor(Math.random() * 1000000 + 1000).toString()
    }
    onMounted(() => {
      state.correctPassword = generateNewPassword()
    })
    watch(
      () => state.status,
      gameState => {
        if (gameState === PASSWORD_STATUS.FAIL) {
          state.correctPassword = generateNewPassword()
        }
      }
    )

    return {
      ...toRefs(state),
      checkPassword,
      generateNewPassword
    }
  }
}
</script>

<template>
  <form class="password-task" @submit.prevent>
    <p class="password-task-status" :class="gameStatus.styles">
      <span>{{ gameStatus.text }}</span>
    </p>
    <div class="password-task-input">
      <label for="enter-password">Enter Password</label>
      <input
        id="enter-password"
        class="nes-input"
        type="text"
        v-model="passwordInput"
        @keyup.enter="checkPassword"
      />
    </div>
    <p class="post-it nes-container">
      Secret Password: <br />{{ correctPassword }}
    </p>
    <button
      class="password-task-submit nes-btn is-primary"
      type="submit"
      @click="checkPassword"
    >
      Submit
    </button>
  </form>
</template>

<style>
.password-task {
  display: grid;
}

.password-task-input {
  display: grid;
  margin-bottom: 30px;
}

.password-task-input label {
  margin-bottom: 1rem;
  font-size: 1.5rem;
}

.password-task-input input {
  padding: 0.5rem;
  font-size: 1.5rem;
  text-align: center;
}

.password-task-submit {
  font-size: 1.2rem;
}

.password-task-status {
  padding: 1rem;
  padding-top: 1.25rem;
  margin-top: 0.5rem;
  margin-bottom: 1.5rem;
  color: #ccc;
  border: 5px dashed #ccc;
}

.password-task-status.is-green {
  border-color: var(--green);
}

.password-task-status.is-red {
  border-color: var(--red);
}

.post-it {
  position: absolute;
  bottom: -200px;
  right: -200px;
  padding: 1rem;
  width: 300px;
  background-color: yellow;
  color: black;
}
</style>
