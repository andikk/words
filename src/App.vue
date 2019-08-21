<template>
  <div class="container" v-bind:class="{overlay: showModal}">
    <div class="loading" v-show="loading">Загружаю слово...</div>

    <app-modal class="modal" v-bind:class="{active: showModal}" :gameStarted="gameStarted" :showButtons="showButtons" :message="message" @gameStarted="startGame" @continueGame="continueGame" @showStatistics="showStatistics">
      <div class="stat" v-show="showStat">
        <table cellpadding="10" class="stat__table">
          <tr class="stat__header">
            <td><a @click="sortParam='num'">Номер</a></td>
            <td><a @click="sortParam='word'">Слово</a></td>
            <td><a @click="sortParam='time'">Время</a></td>
          </tr>
          <tr v-for="(el, index) in sortedList">
            <td>{{el.num}}</td><td>{{el.word}}</td><td>{{el.time}}</td>
          </tr>
        </table>
      </div>
    </app-modal>
    <app-displayed-word :displayedWord="displayedWord" @letterClicked="pasteLetter"></app-displayed-word>
    <app-typed-word :typedWord="typedWord" @letterClicked="deleteLetter" @clearClicked="clear"></app-typed-word>

  </div>
</template>

<script>
import axios from 'axios';
import Modal from './components/Modal.vue';
import DisplayedWord from './components/DisplayedWord.vue';
import TypedWord from './components/TypedWord.vue';

export default {
  data() {
    return {
      word: '',
      typedWord: '',
      displayedWord: '',
      numbers: [],
      words: [],
      counter: 0,
      timer: 1,
      log: [],
      gameStarted: false,
      showModal: true,
      message: 'Для начала игры нажми кнопку',
      showButtons: false,
      showStat: false,
      sortParam: '',
      tryNum: 0,
      loading: true,
    };
  },
  methods: {
    clear() {
      this.typedWord = '';
    },
    showStatistics() {
      this.showButtons = false;
      this.showStat = true;
      this.message = 'Для начала игры нажми кнопку';
      this.gameStarted = false;
    },
    continueGame() {
      this.showStat = false;
      this.showModal = false;
      this.selectWord();
    },
    pasteLetter(index) {
      this.typedWord = this.typedWord + this.displayedWord[index];
      this.check();
    },
    deleteLetter(index) {
      const newStr = this.typedWord.split('');
      newStr.splice(index, 1);
      this.typedWord = newStr.join('');
      this.check();
    },
    check() {
      if (this.typedWord == this.word) {
        this.typedWord = '';
        this.message = 'Вы выиграли! Повторить?';
        this.showButtons = true;
        this.showStat = false;
        this.showModal = true;
        this.tryNum++;
        const stat = {
          num: this.tryNum,
          time: this.timer,
          word: this.word,
        };
        this.log.push(stat);
        this.timer = 0;
      }
    },
    selectWord() {
      const shuffleWord = (word) => {
        let shuffledWord = '';
        word = word.split('');
        while (word.length > 0) {
          shuffledWord += word.splice(word.length * Math.random() << 0, 1);
        }
        return shuffledWord;
      };

      if (this.counter < this.numbers.length) {
        const fileNum = this.numbers[this.counter];

        axios
          .get(`https://apidir.pfdo.ru/v1/directory-program-activities/${fileNum}`)
          .then((response) => {
            if (response.data.result_message == 'Полный успех') {
              if (this.words.some(elem => elem == response.data.data.name.toString()) == false) {
                this.words.push(response.data.data.name.toUpperCase());

                const wordToGuess = response.data.data.name.split(' ').join('_');
                this.word = wordToGuess.toUpperCase();
                this.displayedWord = shuffleWord(wordToGuess.toUpperCase());
                console.log(this.word);
                setInterval(() => (this.timer++), 1000);
                this.loading = true;
              }
              this.counter++;
            } else {
              this.counter++;
              this.selectWord();
            }
          })
          .catch((error) => {
            console.log(error);
          })
          .finally(() => (this.loading = false));
      } else {
        this.message = 'Вы отгадали все слова. Начните игру снова.';
        this.showModal = true;
        this.showButtons = false;
        this.gameStarted = false;
      }
    },

    startGame() {
      this.numbers.length = 0;
      this.words.length = 0;
      this.log.length = 0;
      this.tryNum = 0;
      this.gameStarted = true;
      this.showModal = false;

      const shuffle = (array) => {
        let currentIndex = array.length; let temporaryValue; let
          randomIndex;
        while (currentIndex !== 0) {
          randomIndex = Math.floor(Math.random() * currentIndex);
          currentIndex -= 1;
          temporaryValue = array[currentIndex];
          array[currentIndex] = array[randomIndex];
          array[randomIndex] = temporaryValue;
        }
        return array;
      };

      for (let i = 2; i < 180; i++) {
        this.numbers.push(i);
      }

      shuffle(this.numbers);
      this.selectWord();
    },
  },
  computed: {
    sortedList() {
      const sortByWord = (d1, d2) => ((d1.word.toLowerCase() > d2.word.toLowerCase()) ? 1 : -1);
      const sortByTime = (d1, d2) => ((d1.time > d2.time) ? 1 : -1);
      const sortByNum = (d1, d2) => ((d1.num > d2.num) ? 1 : -1);

      switch (this.sortParam) {
        case 'num': return this.log.sort(sortByNum);
        case 'word': return this.log.sort(sortByWord);
        case 'time': return this.log.sort(sortByTime);
        default: return this.log;
      }
    },
  },
  components: {
    appModal: Modal,
    appDisplayedWord: DisplayedWord,
    appTypedWord: TypedWord,
  },
};
</script>

<style>
.stat__table {
  margin-top: 30px;
  margin-left: auto;
  margin-right: auto;
  border-spacing: 0;
}

.stat tr {
  border: 1px solid grey;
}

.stat td {
  padding-top: 4px;
  padding-bottom: 4px;
  word-break: break-all;
}

.stat__header {
  background-color: #d8d8d8;
}

.loading {
  position: fixed;
  top: 40px;
  left: 50%;
  padding-left: 10px;
  padding-right: 10px;
  padding-top: 20px;
  padding-bottom: 20px;
  border: 1px solid #ded1d1;
  border-radius: 4px;
  background-color: #f1ebeb;
  text-align: center;
  transform: translateX(-50%);
}

.modal {
  display: none;
  position: fixed;
  top: 40px;
  left: 50%;
  z-index: 100;
  padding-left: 40px;
  padding-right: 40px;
  padding-top: 10px;
  padding-bottom: 40px;
  background-color: #fff;
  border-radius: 4px;
  text-align: center;
  transform: translateX(-50%);
}

.active {
  display: block;
}

.overlay {
  overflow-y: hidden;
}

.overlay::before {
  position: absolute;
  content: '';
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: block;
  z-index: 100;
  background-color: rgba(0, 0, 0, 0.7);
}

button {
  display: block;
  padding-left: 35px;
  padding-right: 35px;
  padding-bottom: 10px;
  padding-top: 10px;
  font-size: 16px;
  font-weight: normal;
  line-height: 24px;
  text-align: center;
  color: #fff;
  background-color: #008000;
  border: none;
  border-radius: 40px;
  cursor: pointer;
  user-select: none;
  margin: 0 auto;
}

button:hover {
  background-color: #49a549;
}

button:focus {
  background-color: #286128;
}

.container {
  width: 100%;
}

</style>
