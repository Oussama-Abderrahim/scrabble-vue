<template>
  <div class="container">
    <div class="score-row">
      <h1>SCRABBLE</h1>
      <h3>{{currentPlayer.name}}'s turn</h3>
    </div>
    <div class="game">
      <div class="game-player-one">
        <h2>{{players[0].name}}</h2>
        <ul class="game-palette">
          <li
            @click="onHandClick(0, i)"
            v-for="(letter, i) in players[0].hand"
            :key="i"
            class="cell"
          >{{letter}}</li>
        </ul>
      </div>
      <div class="game-board">
        <div class="row" v-for="(row,i) in board" :key="i">
          <div
            class="cell"
            @click="onBoardClick(i, j)"
            v-for="(col, j) in board[i]"
            :class="{'selected': col.selected, 'locked': col.isLocked}"
            :key="i*15+j"
          >{{board[i][j].content}}</div>
        </div>
      </div>
      <div class="game-player-two">
        <h2>{{players[1].name}}</h2>
        <ul class="game-palette">
          <li
            @click="onHandClick(1, i)"
            v-for="(letter, i) in players[1].hand"
            :key="i"
            class="cell"
          >{{letter}}</li>
        </ul>
      </div>
    </div>
    <button @click="cancel()">Clear</button>
    <button @click="validate()">Validate</button>
  </div>
</template>

<script>
import _ from "lodash";
const EMPTY_CELL = " ";

const LETTERS_DESTRIBUTION = {
  A: {
    count: 9,
    score: 1,
    letter: "A"
  },
  B: {
    count: 2,
    score: 3,
    letter: "B"
  },
  C: {
    count: 2,
    score: 3,
    letter: "C"
  },
  D: {
    count: 4,
    score: 2,
    letter: "D"
  },
  E: {
    count: 12,
    score: 1,
    letter: "E"
  },
  F: {
    count: 2,
    score: 4,
    letter: "F"
  },
  G: {
    count: 3,
    score: 2,
    letter: "G"
  },
  H: {
    count: 2,
    score: 4,
    letter: "H"
  },
  I: {
    count: 9,
    score: 1,
    letter: "I"
  },
  J: {
    count: 1,
    score: 10,
    letter: "J"
  },
  K: {
    count: 1,
    score: 5,
    letter: "K"
  },
  L: {
    count: 4,
    score: 1,
    letter: "L"
  },
  M: {
    count: 2,
    score: 3,
    letter: "M"
  },
  N: {
    count: 6,
    score: 1,
    letter: "N"
  },
  O: {
    count: 8,
    score: 1,
    letter: "O"
  },
  P: {
    count: 2,
    score: 3,
    letter: "P"
  },
  Q: {
    count: 1,
    score: 10,
    letter: "Q"
  },
  R: {
    count: 6,
    score: 1,
    letter: "R"
  },
  S: {
    count: 4,
    score: 1,
    letter: "S"
  },
  T: {
    count: 6,
    score: 1,
    letter: "T"
  },
  U: {
    count: 4,
    score: 1,
    letter: "U"
  },
  V: {
    count: 2,
    score: 4,
    letter: "V"
  },
  W: {
    count: 2,
    score: 4,
    letter: "W"
  },
  X: {
    count: 1,
    score: 8,
    letter: "X"
  },
  Y: {
    count: 2,
    score: 4,
    letter: "Y"
  },
  Z: {
    count: 1,
    score: 10,
    letter: "Z"
  }
};

export default {
  name: "GameBoard",
  data: () => ({
    board: [],
    lettersDeck: [],
    waiting: false,
    players: [
      {
        name: "Player 1",
        hand: []
      },
      {
        name: "Player 2",
        hand: []
      }
    ],
    currentPlayer: null,
    selectedCell: null
  }),
  methods: {
    applyToBoard(f) {
      this.board.forEach(v => v.forEach(f));
    },
    hasAdjacentCell(cell) {
      // Up
      return (
        (cell.i > 0 && this.board[cell.i - 1][cell.j].isLocked) ||
        // Down
        (cell.i < this.board.length - 1 &&
          this.board[cell.i + 1][cell.j].isLocked) ||
        // Left
        (cell.j > 0 && this.board[cell.i][cell.j - 1].isLocked) ||
        // Right
        (cell.j < this.board[cell.i].length - 1 &&
          this.board[cell.i][cell.j + 1].isLocked)
      );
    },
    isSelectedCellsAligned() {
      var selectedCells = [];

      this.applyToBoard(cell => {
        if (cell.selected && cell.content != EMPTY_CELL) {
          selectedCells.push(cell);
        }
      });

      return (
        (selectedCells.every(
          (cell, j, arr) =>
            (j == 0 || arr[j - 1].j == cell.j - 1) &&
            cell.i == selectedCells[0].i
        ) ||
          selectedCells.every(
            (cell, i, arr) =>
              (i == 0 || arr[i - 1].i == cell.i - 1) &&
              cell.j == selectedCells[0].j
          )) &&
        selectedCells.some(cell => this.hasAdjacentCell(cell))
      );
    },
    validate() {
      let isAligned = this.isSelectedCellsAligned();

      if (isAligned) {
        this.applyToBoard(cell => {
          if (cell.selected && cell.content != EMPTY_CELL) {
            cell.isLocked = true;
            cell.selected = false;
          }
          this.currentPlayer =
            this.currentPlayer == this.players[0]
              ? this.players[1]
              : this.players[0];
        });
      } else {
        this.cancel();
      }
    },
    cancel() {
      this.selectedCell = null;
      this.waiting = false;
      // Reset Board
      this.applyToBoard(cell => {
        if (cell.selected && cell.content != EMPTY_CELL) {
          this.currentPlayer.hand.push(cell.content);
          cell.content = EMPTY_CELL;
          cell.selected = false;
        }
        cell.selected = false;
      });
    },
    onHandClick(playerIndex, i) {
      if (this.players[playerIndex] != this.currentPlayer) return;

      this.selectedCell.content = this.currentPlayer.hand[i];
      this.currentPlayer.hand.splice(i, 1);
      // this.selectedCell.selected = false
      this.selectedCell = null;
      this.waiting = false;
    },
    onBoardClick(i, j) {
      if (this.board[i][j].isLocked) return;
      if (this.selectedCell && this.selectedCell.content == EMPTY_CELL)
        this.selectedCell.selected = false;
      this.selectedCell = this.board[i][j];
      this.selectedCell.selected = true;
      this.waiting = true;
    },
    initGameBoard() {
      this.board = Array(15)
        .fill([])
        .map((row, i) =>
          Array(15)
            .fill({})
            .map((col, j) => ({
              i: i,
              j: j,
              content: EMPTY_CELL,
              isLocked: false,
              selected: false
            }))
        );
    },
    generateLettersDeck() {
      this.lettersDeck = [];
      console.log(LETTERS_DESTRIBUTION);
      for (let letter in LETTERS_DESTRIBUTION) {
        this.lettersDeck = this.lettersDeck.concat(
          Array(LETTERS_DESTRIBUTION[letter].count)
            .fill(" ")
            .map(l => LETTERS_DESTRIBUTION[letter].letter)
        );
      }
      this.lettersDeck = _.shuffle(this.lettersDeck);
    }
  },
  beforeMount() {
    this.currentPlayer = this.players[0];
    this.initGameBoard();
    this.generateLettersDeck();
    for (let player of this.players) {
      for (let i = 0; i < 7; i++) player.hand.push(this.lettersDeck.pop());
    }
    this.cancel();
    this.board[7][7].content = "H";
    this.board[7][7].isLocked = true;
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang='scss' scoped>
$primary-color: #27ae60;
$cell-size: 24px;

.container {
  padding: 0;
  width: 100%;
  height: 100%;
  text-align: center;
  // max-width: 500px;
}
.score-row {
}

.game {
  display: flex;

  justify-content: space-between;
  width: 100%;
  margin: auto;

  background-color: #fff;

  &-palette {
    list-style-type: none;
    display: flex;
    li {
      display: block;
    }
  }

  &-player-one,
  &-player-two {
    display: flex;
    flex-direction: column;
    margin: auto;
    width: 300px;
    text-align: center;
  }

  &-board {
    display: table;

    .row {
      display: flex;
      border: 1px solid $primary-color;
      justify-content: space-between;
    }
  }

  .cell {
    height: $cell-size;
    line-height: 100%;
    background-color: #2ecc71;
    width: $cell-size;
    font-size: calc(#{$cell-size} + -2px);
    border: 1px solid $primary-color;
    margin: 1px;
    user-select: none;
    cursor: pointer;

    &.selected {
      background-color: #fff;
    }
    &.locked {
      background-color: grey;
    }
    &:hover {
      margin: 0px;
      border: 2px solid red;
    }
  }
}
</style>
