<template>
  <div id="app">
    <section class="panel">
      <button @click="onClickNewGame">New Game</button>
    </section>
    <section class="deck">
      <div class="top-deck">
        <div class="temp-deck">
          <div
            v-for="(cardId, i) in tempDeck" :key="i"
            class="block"
            @drop="onDropTempBlock(i)"
            @dragover="allowDrop"
          >
            <card v-if="cardId" :cardId="cardId" />
          </div>
        </div>
        <div class="finish-deck">
          <div
            v-for="(cardId, i) in finishDeck" :key="i"
            class="block"
            @drop="onDropFinishBlock(i)"
            @dragover="allowDrop"
          >
            <card v-if="cardId" :cardId="cardId" />
          </div>
        </div>
      </div>
      <div class="main-deck">
        <div class="cards-column" v-for="(cards, columnIndex) in mainDeck" :key="columnIndex">
          <cards-column
            :cards="cards"
            :columnIndex="columnIndex"
            @onDropCardColumn="onDropCardColumn"
          >
            <card
              v-for="(cardId, cardIndex) in cards"
              :cardId="cardId"
              :columnIndex="columnIndex"
              :cardIndex="cardIndex"
              :key="cardId"
              @drag="onDragSingleCard"
            />
          </cards-column>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import Card from '@/components/Card.vue';
import CardsColumn from '@/components/CardsColumn.vue';

export default {
  name: 'app',
  components: {
    Card, CardsColumn,
  },
  data() {
    return {
      // basic setting
      cardType: ['spade', 'heart', 'diamond', 'club'],
      mainDeckCardNum: [7, 7, 7, 7, 6, 6, 6, 6,],

      // per game setting
      shuffledCards: [],
      mainDeck: [[], [], [], [], [], [], [], []],
      tempDeck: ['', '', '', ''],
      finishDeck: ['', '', '', ''],

      // per move
      dragTargetCardId: '',
      dragTargetCardIndex: '',
      dragTargetColumnIndex: '',
      timer: 0,
      screenshot: [],
    }
  },
  computed: {
  },
  methods: {
    // main game logic
    onDragSingleCard(cardId, cardIndex, columnIndex) {
      this.dragTargetCardId = cardId;
      this.dragTargetCardIndex = cardIndex;
      this.dragTargetColumnIndex = columnIndex;
    },
    onDropTempBlock(i) {
      if (this.tempDeck[i]) return;

      this.$set(this.tempDeck, i, this.dragTargetCardId);
      this.mainDeck[this.dragTargetColumnIndex].pop();
    },
    onDropFinishBlock(i) {
      const card1 = this.finishDeck[i];
      const card2 = this.dragTargetCardId

      if (
        (this.isSameCardType(card1, card2) && this.isAscendentCardNumber(card1, card2))
        || (this.dragTargetCardId.split('-')[1] === '1' && !this.finishDeck[i])
      ) {
        this.$set(this.finishDeck, i, this.dragTargetCardId);
        this.mainDeck[this.dragTargetColumnIndex].pop();
      };
    },
    onDropCardColumn(columnIndex) {
      // 取得最末牌 card id
      const length = this.mainDeck[columnIndex].length;
      const endCardId = this.mainDeck[columnIndex][length - 1];
      
      // 判定不同顏色、數字小一號
      if (
        this.isDiffCardColor(this.dragTargetCardId, endCardId)
        && this.isDescentCardNumber(endCardId, this.dragTargetCardId)
        ) {
        this.mainDeck[this.dragTargetColumnIndex].pop();
        this.mainDeck[columnIndex].push(this.dragTargetCardId);
      }
    },

    // new game
    onClickNewGame() {
      // reset
      this.mainDeck = [[], [], [], [], [], [], [], []];
      this.tempDeck = ['', '', '', ''];
      this.finishDeck = ['', '', '', ''];

      let cards = [];
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

    // some utils methods
    isSameCardType(a, b) {
      return a.split('-')[0] === b.split('-')[0]
    },
    isAscendentCardNumber(a, b) {
      return parseInt(a.split('-')[1]) === parseInt(b.split('-')[1]) - 1
    },
    isDescentCardNumber(a, b) {
      return parseInt(a.split('-')[1]) === parseInt(b.split('-')[1]) + 1
    },
    isDiffCardColor(a, b) {
      const aColor = (a.split('-')[0] === 'heart' || a.split('-')[0] === 'diamond') ? 'red' : 'black';
      const bColor = (b.split('-')[0] === 'heart' || b.split('-')[0] === 'diamond') ? 'red' : 'black';
      return aColor !== bColor;
    },
    allowDrop(e) {
      e.preventDefault();
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
@import '@/scss/reset.scss';
@import '@/scss/app.scss';

</style>
