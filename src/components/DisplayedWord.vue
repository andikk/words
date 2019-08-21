<template>
  <div class="disp-word-container">
    <h2 class="title">Нажимай на буквы, чтобы составить из них слово</h2>
  	<ul class="word">
      <li class="letter" v-for="(letter, index) in displayedWord" @click="pasteLetter(index)">{{ letter }}</li>
    </ul>
  </div>
</template>

<script>
  export default {
  	methods: {
  	  pasteLetter(index) {
  	  	this.$emit('letterClicked', index); 
  	  } 
  	},
    props: ['displayedWord']
  }
</script>

<style scoped>
  .disp-word-container {
    text-align: center;
  }

  .letter {
    display: inline-block;
    font-size: 36px;
    border: 2px solid #008000;
    padding: 15px;
    margin-right: 4px;
    margin-bottom: 8px;
    border-radius: 4px;
    transition: all 0.3s;
  }

  .letter:hover {
    color: #fff;
    background-color: #008000;
    transform: scale(1.1);   
  }
</style>