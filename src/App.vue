<template>
  <div class="container centered">
  <transition name="fade">
        <div v-if="!hasCorrectSize" class="centered">
        <div class="popup-backdrop centered" :class="{'show':hasCorrectSize}">
          <div class="popup-container centered">
            <h2>
              Please enter board size here:
            </h2>
            <p>must be between 3 and 5</p>
            <input type="number" @input="enterSize($event.target.value)" min="3" max="5">
          </div>
        </div>
    </div>
  </transition>
    <div v-if="!hasCorrectSize" class="centered">
        <div class="popup-backdrop centered" :class="{'show':hasCorrectSize}">
          <div class="popup-container centered">
            <h2>
              Please enter board size here:
            </h2>
            <p>between 3 and 5</p>
            <input type="number" @input="enterSize($event.target.value)" min="3" max="5">
          </div>
        </div>
    </div>

    <div class="results">
      <div class="score-board centered">
        <h2>Results:</h2>
        <span>X: {{results.x}}</span>
        <span>O: {{results.o}}</span>
      </div>
      <div class="player-turn centered">
        <h2>Current player:</h2>
        <span>{{currentPlayer}}</span>
      </div>
      <div class="current-round centered">
        <h2>Round:</h2>
        <span>{{rounds}}</span>
      </div>
       <div class="current-round centered">
        <h2>Draw:</h2>
        <span>{{draw}}</span>
      </div>
    </div>
    <div class="board centered">
      <div class="row centered" v-for="(_, rowIndex) in board.length" :key="rowIndex">
        <div class="col centered" v-for="(_, colIndex) in board.length" :key="colIndex" :id="`${rowIndex}${colIndex}`" @click="play(rowIndex,colIndex,$event)">{{board[rowIndex][colIndex]}}</div>
      </div>
    </div>
    <div class="signature">
      Made by Dayana Ilieva
    </div>
  </div>

</template>

<script>

export default {
  name: 'App',

  data() {
    return {
      currentPlayer:'x',
      boardSize: 0,
      enteredSize:0,
      board: [],
      rounds:0,
      results: {
        x: 0,
        o: 0
      },
      moves: 0,
      draw: 0,
    }
  },
  created() {
    this.clearBoard();
  },
  methods: {
    enterSize(value){
      this.enteredSize = Number(value);
      if(this.hasCorrectSize) {
        this.boardSize = Number(this.enteredSize);
        this.clearBoard();
      }
    },
    start() {
      this.$confetti.start();
    },
    stop() {
      this.$confetti.stop();
    },
    play(row,col,ev) {
      if(!this.board[row][col]) {
        this.makeMove(row,col,ev.target);
      }
      this.finishTurn();
      this.currentPlayer = this.currentPlayer === 'x' ? 'o' : 'x';
      setTimeout(() => {
          let [x,y] = this.generateRandomPositions();
          while (this.board[x][y]) {
            [x,y] = this.generateRandomPositions();
          }
          this.makeMove(x,y,document.getElementById(`${x}${y}`));
          this.finishTurn();
          this.currentPlayer = this.currentPlayer === 'x' ? 'o' : 'x';
        },400
      )
    },
    finishTurn() {
      if(this.hasWinner){
        this.results[this.currentPlayer]++;
        this.clearBoard();
        this.rounds++;
        this.start();
        setTimeout(()=>this.stop(),700);
        return;
      }
      if(this.isDraw) {
        this.draw++;
        this.clearBoard();
        this.rounds++
        return;
      }
    },
    makeMove(row,col,target){
      target.textContent = this.currentPlayer;
      this.board[row][col] = this.currentPlayer;
      this.moves++;
    },
    generateRandomPositions() {
      return [Math.floor(Math.random() * this.boardSize-1) + 1, Math.floor(Math.random() * this.boardSize-1) + 1];
    },
    clearBoard(){
      document.querySelectorAll('.col').forEach(e=>e.textContent="");
      this.board = [];
      this.moves = 0;
      this.currentPlayer = 'x';
      for (let step = 0; step < this.boardSize; step++) {
        let row = [];
        for (let step2 = 0; step2 < this.boardSize; step2++){
          row.push('');
        }
        this.board.push(row);
      }
    }
  },
  computed:{
    totalMoves(){
      return this.boardSize * this.boardSize;
    },
    hasCorrectSize(){
      return Number(this.enteredSize) >= 3 && Number(this.enteredSize)<=5;
    },
    checkInRow(){
      let result = [];
      for (let step = 0; step < this.boardSize; step++) {
        result.push(this.board[step].every( e  => e === this.currentPlayer));
      }

      return result.some(el=> el===true)
    },
    checkInCol() {
      let result = [];
      for (let step = 0; step < this.boardSize; step++) {
        let colArr = []
        for(let step2=0; step2 < this.boardSize; step2++) {
          colArr.push(this.board[step2][step]);
        }
        result.push(colArr.every( e  => e === this.currentPlayer));

      }
      return result.some(el=> el===true)
    },
    checkInDiagonals() {
      let diagonal1 = [];
      let diagonal2 = [];
      let result = [];
      for (let step = 0; step < this.boardSize; step++) {
        diagonal1.push(this.board[step][step]);
        diagonal2.push(this.board[step][this.boardSize - (step + 1)]);
      }
      result.push(diagonal1.every( e  => e === this.currentPlayer));
      result.push(diagonal2.every( e  => e === this.currentPlayer));
      return result.some(el=> el===true);
    },
    isDraw() {
      return this.totalMoves === this.moves
    },
    hasWinner() {
      return [this.checkInRow,this.checkInCol,this.checkInDiagonals].some(el=> el===true);
    },
  }
}
</script>

<style>
.popup-backdrop {
  position: absolute;
  z-index: 1;
  background-color: rgba(103, 53, 150, 0.73);
  top:0;
  left: 0;
  right: 0;
  bottom: 0;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity 700ms;
}
input{
  height: 20px;
  width: 100px;
}
 .fade-leave-active {
  opacity: 0;
}
.popup-container {
  position: absolute;
  flex-direction: column;
  padding: 20px;
  z-index: 2;
  width: auto;
  margin: 10px;
  text-align: center;
  height: auto;
  background-color: #9ba4ef;
  border-radius: 5px;
}
h2{
  margin: 0;
}
.centered {
  display: flex;
  align-items: center;
  justify-content: center;
  text-shadow: 2px 2px 2px #b68826;
}
body{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Roboto Slab', serif;
  color: #f6f3f3;
}
.container{
  width: 100%;
  height: 100vh;
  flex-direction: column;
  position: relative;
  background-color: rgba(25,15,48,0.89);
}
@media screen and (min-width: 600px) {
  .container{
    flex-direction: row;
  }
}
.board {
  flex-direction: column;
  width: auto;
  height: auto;
  margin: 20px;
  background-color: #9ba4ef;
  padding: 30px 20px;
  -webkit-box-shadow: 5px 5px 26px 5px #673596;
  box-shadow: 5px 5px 26px 5px #673596;
  border-radius: 5px;
}
.row {
  flex-direction: row;
  height: 50px;
  border: 1px solid black;
  font-size: 32px;
}
.col {
  flex-direction: row;
  width: 50px;
  height: 100%;
  border: 1px solid black;
}
.col:first-of-type{
  border-left: none;
}
.col:last-of-type{
  border-right: none;
}
.results{
  display: flex;
  align-items: center;
  flex-direction: column;
  justify-content: space-around;
  width: 300px;
  background-color: #ebaa2e;
  border-radius: 5px;
  -webkit-box-shadow: 5px 5px 26px 5px rgba(221,165,46,0.79);
  box-shadow: 5px 5px 26px 5px rgba(221,165,46,0.79);
}
.player-turn, .current-round, .score-board {
  flex-direction: column;
  margin: 10px;
  padding: 5px;
  border-radius: 5px;
  width: 80%;
  background-color: #f1c78e;
  -webkit-box-shadow: 5px 5px 26px 5px #b68826;
  box-shadow: 5px 5px 26px 5px #b68826;
}
.signature{
  font-family: 'Permanent Marker', cursive;
  color: #fbb403;
  text-shadow: #5c4903;
  font-size: 20px;
  position: absolute;
  bottom: 10px;
  right:20px;
}

@media screen and (min-width: 600px) {
  .container{
    flex-direction: row;
  }
  .signature{
    font-size: 34px;
  }
}
</style>
