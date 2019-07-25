<template>
  <div id="app">
    <section class="panel">這是面板
      <button @click="onClickNewGame">New Game</button>
    </section>
    <section class="deck">
      <div class="top-deck">
        <div class="temp-deck">
          <div class="block" @drop="drop" @dragover="allowDrop"></div>
          <div class="block" @drop="drop" @dragover="allowDrop"></div>
          <div class="block" @drop="drop" @dragover="allowDrop"></div>
          <div class="block" @drop="drop" @dragover="allowDrop"></div>
        </div>
        <div class="finish-deck">
          <div class="block"></div>
          <div class="block"></div>
          <div class="block"></div>
          <div class="block"></div>
        </div>
      </div>
      <div class="main-deck">
        <div class="column" v-for="(column, index) in mainDeck" :key="index">
          <div class="card-wrap" v-for="(card, index) in column" :key="index">
            <img
              draggable
              @dragstart="drag"
              :id="card"
              class="card-image"
              :src="require(`@/assets/card-${card}.svg`)"
              alt="card">
          </div>
          <!-- drop 區塊 -->
          <div class="card-wrap" @drop="drop" @dragover="allowDrop"></div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>

export default {
  name: 'app',
  components: {
  },
  data() {
    return {
      shuffledCards: [],
      cardType: ['spade', 'heart', 'diamond', 'club'],
      mainDeck: [[], [], [], [], [], [], [], []],
      tempDeck: ['', '', '', ''],
      finishDeck: ['', '', '', ''],
      timer: 0,
      screenshot: [],
    }
  },
  methods: {
    drag(e) {
      e.dataTransfer.setData("text", e.target.id);
      console.log(e.target.id);
    },
    drop(e) {
      e.preventDefault();
      const data = e.dataTransfer.getData("text");
      e.target.appendChild(document.getElementById(data));
    },
    allowDrop(e) {
      e.preventDefault();
    },
    onClickNewGame() {
      let cards = [];
      this.mainDeck = [[], [], [], [], [], [], [], []];
      this.cardType.forEach((type) => {
        for (let i = 1; i <= 13; i++) {
          cards.push(`${type}-${i}`);
        }
      });
      this.shuffledCards = this.shuffle(cards);
      for (let i = 0; i < 52; i++) {
        if (i >= 46) {
          this.mainDeck[7].push(this.shuffledCards[i]);
        } else if (i >= 40) {
          this.mainDeck[6].push(this.shuffledCards[i]);
        } else if (i >= 34) {
          this.mainDeck[5].push(this.shuffledCards[i]);
        } else if (i >= 28) {
          this.mainDeck[4].push(this.shuffledCards[i]);
        } else if (i >= 21) {
          this.mainDeck[3].push(this.shuffledCards[i]);
        } else if (i >= 14) {
          this.mainDeck[2].push(this.shuffledCards[i]);
        } else if (i >= 7) {
          this.mainDeck[1].push(this.shuffledCards[i]);
        } else {
          this.mainDeck[0].push(this.shuffledCards[i]);
        }
      }
    },
    shuffle(array) {
      let currentIndex = array.length;
      let temporaryValue, randomIndex;

      // While there remain elements to shuffle...
      while (0 !== currentIndex) {
        // Pick a remaining element...
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex -= 1;

        // And swap it with the current element.
        temporaryValue = array[currentIndex];
        array[currentIndex] = array[randomIndex];
        array[randomIndex] = temporaryValue;
      }

      return array;
    },
  },
  mounted() {
    this.onClickNewGame();
  }
}
</script>

<style lang="scss">
/* reset */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  // outline: 1px solid red;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: white;
  background-color: #000;
  width: 100%;
  height: 100vh;
  display: flex;
}

.panel {
  background-color: pink;
  height: 100%;
  flex: 0 0 74px;
}
.deck {
  flex: 1;
  display: flex;
  flex-direction: column;
  .top-deck {
    width: 100%;
    flex: 0 0 173px;
    background-color: #112233;
    padding: 40px 0 5px 0;
    display: flex;
    justify-content: center;
    .temp-deck, .finish-deck {
      display: flex;
      .block {
        width: 150px;
        height: 231.7px;
        background: #667788;
        &:not(:first-child) {
          margin-left: 20px;
        }
        .card-image {
          width: 100%;
          height: 100%;
        }
      }
    }
    .finish-deck {
      margin-left: 20px;
    }
  }
  .main-deck {
    width: 100%;
    flex: 1;
    background-color: #112233;
    display: flex;
    justify-content: center;
    padding: 20px 40px;
    .column {
      display: flex;
      flex-direction: column;
      &:not(:first-child) {
        margin-left: 20px;
      }
      .card-wrap {
        width: 150px;
        height: 231.7px;
        &:not(:first-child) {
          margin-top: -190px;
        }
        &:last-child {
          z-index: 1;
        }
        .card-image {
          width: 100%;
          height: 100%;
          filter: hue-rotate(35deg);
        }
      }
    }
  }
}
</style>
