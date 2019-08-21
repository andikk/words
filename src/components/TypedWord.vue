<template>
  <div class="typed-word-container">
    <h3>Если ты ошибся, то нажми на букву, чтобы удалить её</h3>
    <ul class="word">
      <li class="letter" v-for="(letter, index) in typedWord" @click="deleteLetter(index)">{{ letter }}</li>
    </ul>	
    <button @click="clear">Очистить</button>
  </div>

</template>

<script>
  export default {
  	methods: {
  	  deleteLetter(index) {
  	  	this.$emit('letterClicked', index); 
  	  },
      clear() {
        this.$emit('clearClicked');
      } 
  	},
    props: ['typedWord'] 
  }
</script>

<style scoped>

  .typed-word-container {
    text-align: center;
  }

  .word {
    border: 2px solid #008000;
    border-radius: 4px; 
    min-height: 50px;
  }

  .letter {
    display: inline-block;
    font-size: 36px;
    position: relative;
  }

  .letter:hover {
    color: grey;
  }
 
</style>