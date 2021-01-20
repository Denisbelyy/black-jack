<template>
  <div class="gameboard">
    <div class="game-area">
      <div class="game-area-item game-area-item__deal">
        <transition-group name="card-animation-deal" tag="div">
          <div v-for="(card, index) in dealCards" :key="card.id" class="card card-active" :style="{
            'background-position': card.bp,
            'left': `${index * 30}px`
          }"></div>
        </transition-group>
          <div class="game-area-score">
            Dealer Hand: {{dealHand}}
          </div>
        </div>
      <div class="game-area-item game-area-item____players">
        <transition-group name="card-animation-player" tag="div">
          <div
            v-for="(card, index) in playerCards"
            :key="card.id"
            class="card card-active" :style="{
              'background-position': card.bp,
              'left': `${index * 30}px`,
            }"></div>
        </transition-group>
          <div class="game-area-score">
            Your Hand: {{playerHand}}
          </div>
      </div>
    </div>
    <div class="menu-area">
      <div class="pack-card">
        <div class="shirt_cart"></div>
      </div>
      <div class="score-table">
        <span>Credit: {{credit}}$</span>
        <span>Bet: {{currentBet}}$</span>
      </div>
      <div class="actions">
        <button class="btn" @click="startGame" v-if="!isGame">Start Game</button>
        <button class="btn" @click="hit" v-if="isGame" :disabled="disableButton">Hit</button>
        <button class="btn" @click="stand" v-if="isGame" :disabled="disableButton">Stand</button>
        <button
          class="btn"
          @click="doubleBet"
          v-if="isGame"
          :disabled="disableButton"
        >
          Double
        </button>
      </div>
    </div>
    <div class="modal" v-if="isEndGame">
      <div class="result-game">
        {{ resultGame }}
      </div>
      <button class="btn" @click="startGame">Resume</button>
    </div>
    <div class="overlay" v-if="isEndGame"></div>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      deskCard: [],
      dealCards: [],
      playerCards: [],
      activeCards: [],
      resultGame: '',
      isGame: false,
      isEndGame: false,
      disableButton: false,
      dealHand: 0,
      playerHand: 0,
      credit: 100,
      bet: 5,
      currentBet: 5,
    };
  },
  mounted() {
    this.generateCards();
  },
  methods: {
    generateCards() {
      let id = 1;
      let currentX = 0;
      let currentY = 0;
      let cost = 2;
      const countSuit = 4;
      const countCards = 13;
      for (let i = 0; i < countSuit; i += 1) {
        for (let j = 0; j < countCards; j += 1) {
          if (j !== 0) {
            currentX -= 102;
          }
          this.deskCard.push({
            id,
            cost: cost > 10 ? 10 : cost,
            bp: `${currentX}px ${currentY}px`,
            aceCard: j === 12,
          });
          cost += 1;
          id += 1;
        }
        cost = 2;
        currentX = 0;
        currentY -= 144;
      }
    },
    startGame() {
      this.isEndGame = false;
      this.isGame = true;
      this.currentBet = this.bet;
      this.activeCards = this.shuffle([...this.deskCard]);
      this.dealCards = this.activeCards.splice(0, 1);
      this.playerCards = this.activeCards.splice(0, 2);
      this.checkScore(this.playerCards, 'playerHand');
      this.checkScore(this.dealCards, 'dealHand');
    },
    checkScore(array, hand) {
      this[hand] = 0;
      array.forEach((el) => {
        if (el.aceCard) {
          this[hand] = this[hand] + el.cost > 21
            ? this[hand] + 1
            : this[hand] + 11;
        } else {
          this[hand] += el.cost;
        }
      });
      if (this[hand] > 21) {
        setTimeout(() => {
          this.isEndGame = true;
          this.checkResult();
        }, 700);
      }
    },
    hit() {
      this.disableButton = true;
      this.playerCards = [
        ...this.playerCards,
        ...this.activeCards.splice(0, 1),
      ];
      setTimeout(() => {
        this.checkScore(this.playerCards, 'playerHand');
        this.disableButton = false;
      }, 700);
    },
    stand() {
      this.disableButton = true;
      this.dealCards = [
        ...this.dealCards,
        ...this.activeCards.splice(0, 1),
      ];
      this.checkScore(this.dealCards, 'dealHand');
      if (this.dealHand >= 21 || this.dealHand > this.playerHand) {
        setTimeout(() => {
          this.isEndGame = true;
          this.checkResult();
          this.disableButton = false;
        }, 700);
      } else {
        setTimeout(this.stand, 1000);
      }
    },
    doubleBet() {
      this.currentBet *= 2;
      this.hit();
    },
    checkResult() {
      if (this.dealHand === this.playerHand && this.dealHand === 21) {
        this.resultGame = 'Draw!';
      } else if (this.playerHand > 21) {
        this.credit -= this.currentBet;
        this.resultGame = 'You Lose!';
      } else if (this.dealHand > 21) {
        this.credit += this.currentBet;
        this.resultGame = 'You Win!';
      } else if (this.dealHand < this.playerHand) {
        this.credit += this.currentBet;
        this.resultGame = 'You Win!';
      } else if (this.dealHand > this.playerHand) {
        this.credit -= this.currentBet;
        this.resultGame = 'You Lose!';
      }
    },
    shuffle(array) {
      return array.sort(() => Math.random() - 0.5);
    },

  },
};
</script>
<style lang="scss">
.gameboard {
  width: 950px;
  height: 600px;
  margin: 20px auto;
  background: #155228;
  display: flex;
  justify-content: space-between;
  flex-direction: row;
  position: relative;
}
.menu-area {
  padding: 30px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  border-left: 10px solid gold;
  align-items: center;
  .score-table {
    span {
      color: white;
      display: block;
      margin: 10px 0;
    }
  }
  >div {
    height: 30%;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
}
.game-area {
  width: 100%;
  &-item {
    height: 50%;
    position: relative;
    margin: 20px 0 0 80px;
    box-sizing: border-box;
  }
  &-score {
    color: white;
    text-align: right;
    padding-right: 40px;
    padding-top: 40px;
  }
}
.card-animation-player {
  &-enter-active {
    transition: all 1s;
  }
  &-enter {
    transform: translate(600px, -300px)
  }
}
.card-animation-deal {
  &-enter-active {
    transition: all 1s;
  }
  &-enter {
    transform: translate(550px, 10px)
  }
}
.card {
  width: 93px;
  height: 134px;
  background: url(/img/cards.png);
  background-size: 1504px 566px;
  box-shadow: 0px 0px 2px 0px black;
  border-radius: 8px;
  display: inline-block;
  margin: 2px;
  &-active {
    position: absolute;
    top: 10px;
    left: 0px;
  }
}
.shirt_cart {
  width: 93px;
  height: 134px;
  background: url(/img/cards.png);
  background-position: -1325px -288px;
  background-size: 1504px 566px;
  box-shadow: 0px 0px 2px 0px black;
  border-radius: 8px;
}
.btn {
  display: block;
  margin: 10px 0;
  width: 120px;
  font-weight: 400;
  text-align: center;
  white-space: nowrap;
  vertical-align: middle;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  border: 1px solid transparent;
  padding: .375rem .75rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: .25rem;
  transition: color .15s ease-in-out,
    background-color .15s ease-in-out,
    border-color .15s ease-in-out,
    box-shadow .15s ease-in-out;
  color: #212529;
  background-color: #ffc107;
  border-color: #ffc107;
  cursor: pointer;
  &:hover {
    color: #212529;
    background-color: #e0a800;
    border-color: #d39e00;
  }
}
// modal
.modal {
  z-index: 2;
  margin: auto;
  position: absolute;
  top: 0; left: 0; bottom: 0; right: 0;
  background: #fff;
  width: 300px;
  height: 150px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  align-items: center;
  border-radius: 10px;
  box-shadow: 0 0 4px 2px #847878;
  .result-game {
    font-size: 24px;
    font-weight: bold;
    padding-top: 10px;
  }
}
.overlay {
  position: fixed;
  background-color:rgba(0, 0, 0, 0.21);
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;

}
</style>
