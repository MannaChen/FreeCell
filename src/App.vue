<template>
	<div id="app">
		<section class="panel">
			<button class="button" @click="onClickNewGame">Restart</button>
			<button class="button" @click="onClickUndo">Undo</button>
      <button class="button" @click="onClickRedo">Redo</button>
		</section>
		<section class="deck">
			<div class="top-deck">
				<div class="temp-deck">
					<div
						v-for="(cardId, i) in tempDeck"
						:key="i"
						class="block"
						@drop="onDropTempBlock(i, target.columnIndex)"
						@dragover="allowDrop"
					>
						<card
							v-if="cardId"
							:cardId="cardId"
							draggable
							@drag="onDragCard(cardId, i, tempDeckColumnIndex)"
						/>
					</div>
				</div>
				<div class="finish-deck">
					<div
						v-for="(cardId, i) in finishDeck"
						:key="i"
						class="block"
						@drop="onDropFinishBlock(i)"
						@dragover="allowDrop"
					>
						<card v-if="cardId" :cardId="cardId" :draggable="false" />
					</div>
				</div>
			</div>
			<div class="main-deck" v-if="!isGameFinished">
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
							:draggable="isDraggable(cardIndex, columnIndex)"
							:key="cardId"
							@drag="onDragCard"
						/>
					</cards-column>
				</div>
			</div>
      <div v-else class="main-deck finish-congratulations">WELL DONE!</div>
		</section>
	</div>
</template>

<script>
import Card from "@/components/Card.vue";
import CardsColumn from "@/components/CardsColumn.vue";

export default {
	name: "app",
	components: {
		Card,
		CardsColumn
	},
	data() {
		return {
			// basic setting
			cardType: ["spade", "heart", "diamond", "club"],
			mainDeckCardNum: [7, 7, 7, 7, 6, 6, 6, 6],
			tempDeckColumnIndex: 8,
			finishDeckColumnIndex: 9,

			// per game setting
			shuffledCards: [],
			mainDeck: [[], [], [], [], [], [], [], []],
			tempDeck: ["", "", "", ""],
      finishDeck: ["", "", "", ""],
      moves: 0,

			// 拖曳標的
			target: {
				cardId: "",
				cardIndex: 0,
				columnIndex: 0,
				children: []
			},
			timer: 0,
			screenshot: []
		};
	},
	computed: {
    isGameFinished() {
      return this.finishDeck.every((item) => {
        return item.includes('13');
      })
    }
  },
	methods: {
		// main game logic
		onDragCard(cardId, cardIndex, columnIndex) {
			this.target.cardId = cardId;
			this.target.cardIndex = cardIndex;
			this.target.columnIndex = columnIndex;

			if (columnIndex >= 8) return;

			let i = cardIndex;
			const column = this.mainDeck[columnIndex];
			// 是否有子牌
			while (column[i + 1] !== undefined) {
				this.target.children.push(column[i + 1]);
				i += 1;
			}
		},
		onDropTempBlock(i, columnIndex) {
			if (this.tempDeck[i]) return;
			if (this.target.children.length >= 1) return;

			this.$set(this.tempDeck, i, this.target.cardId);
			if (this.target.columnIndex === 8) {
				this.tempDeck[this.target.cardIndex] = "";
			} else if (this.target.columnIndex <= 7) {
				this.mainDeck[this.target.columnIndex].pop();
			}
			this.resetTarget();
		},
		onDropFinishBlock(i) {
			if (this.target.children.length >= 1) return;

			const card1 = this.finishDeck[i];
			const card2 = this.target.cardId;

			if (
				(this.isSameCardType(card1, card2) &&
					this.isAscendentCardNumber(card1, card2)) ||
				(this.target.cardId.split("-")[1] === "1" && !this.finishDeck[i])
			) {
				this.$set(this.finishDeck, i, this.target.cardId);
				if (this.target.columnIndex === 8) {
					this.tempDeck[this.target.cardIndex] = "";
				} else if (this.target.columnIndex <= 7) {
					this.mainDeck[this.target.columnIndex].pop();
				}
			}
			this.resetTarget();
		},
		onDropCardColumn(columnIndex) {
      
      const length = this.mainDeck[columnIndex].length;
      
      if (length > 0) {

				const endCardId = this.mainDeck[columnIndex][length - 1];
				if (
					this.isDiffCardColor(endCardId, this.target.cardId) &&
					this.isDescentCardNumber(endCardId, this.target.cardId)
				) {
					if (this.target.columnIndex === 8) {
						this.tempDeck[this.target.cardIndex] = "";
					} else if (this.target.columnIndex <= 7) {
						this.mainDeck[this.target.columnIndex].splice(
							this.target.cardIndex
						);
					}
					this.mainDeck[columnIndex].push(
						this.target.cardId,
						...this.target.children
					);
				}
			} else if (length === 0 && this.target.cardId.includes("13")) {
				console.log("13");
				if (this.target.columnIndex === 8) {
					this.tempDeck[this.target.cardIndex] = "";
				} else if (this.target.columnIndex <= 7) {
					this.mainDeck[this.target.columnIndex].splice(this.target.cardIndex);
				}
				this.mainDeck[columnIndex].push(
					this.target.cardId,
					...this.target.children
				);
			}
			this.resetTarget();
		},
		resetTarget() {
			this.target.cardId = "";
			this.target.cardIndex = 0;
			this.target.columnIndex = 0;
      this.target.children = [];
      
      this.moves += 1;
		},

		// panel click
		onClickNewGame() {
			// reset
			this.mainDeck = [[], [], [], [], [], [], [], []];
			this.tempDeck = ["", "", "", ""];
      this.finishDeck = ["", "", "", ""];
      this.moves = 0;

			let cards = [];
			this.cardType.forEach(type => {
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
    onClickUndo() {},
    onClickRedo() {},

		// some utils methods
		isDraggable(cardIndex, columnIndex) {
			let i = cardIndex;
			const column = this.mainDeck[columnIndex];
			// 是否有子牌
			while (column[i + 1] !== undefined) {
				if (
					this.isDiffCardColor(column[i], column[i + 1]) &&
					this.isDescentCardNumber(column[i], column[i + 1])
				) {
					i += 1;
				} else {
					return false;
					break;
				}
			}
			return true;
		},
		isSameCardType(a, b) {
			return a.split("-")[0] === b.split("-")[0];
		},
		isAscendentCardNumber(a, b) {
			return parseInt(a.split("-")[1]) === parseInt(b.split("-")[1]) - 1;
		},
		isDescentCardNumber(a, b) {
			return parseInt(a.split("-")[1]) === parseInt(b.split("-")[1]) + 1;
		},
		isDiffCardColor(a, b) {
			const aColor =
				a.split("-")[0] === "heart" || a.split("-")[0] === "diamond"
					? "red"
					: "black";
			const bColor =
				b.split("-")[0] === "heart" || b.split("-")[0] === "diamond"
					? "red"
					: "black";
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
		}
	},
	mounted() {
		this.onClickNewGame();
	}
};
</script>

<style lang="scss">
@import "@/scss/reset.scss";
@import "@/scss/app.scss";
@import "@/scss/button.scss";
</style>
