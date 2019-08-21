<template>
  <div>
    <h1>Игра "Угадай слово"</h1>
    <p class="message">{{ message }}</p>
    <button v-show="!gameStarted" @click="startGame">Начать игру</button>
    <div class="btn-container" v-show="showButtons">
      <button @click="continueGame">Конечно!</button>
      <button @click="showStatistics">Я уже устал</button>
    </div>
    <div>
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
 	methods: {
    startGame() {
  	 	this.$emit('gameStarted');
  	},
    continueGame() {
      this.$emit('continueGame');
    },
    showStatistics() {
      this.$emit('showStatistics');
    },
 	},
  
  props: ['gameStarted', 'message', 'showButtons'],
};
</script>

<style scoped>
  .btn-container {
    display: flex;
    justify-content: space-around;
  }

  .message {
    font-size: 24px;
  }
</style>
