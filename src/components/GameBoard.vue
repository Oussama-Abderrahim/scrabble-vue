<template>
  <div class="container">
    <div class="score-row"></div>
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
const EMPTY_CELL = " ";

export default {
  name: "GameBoard",
  data: () => ({
    board: [],
    waiting: false,
    players: [
      {
        name: "Player 1",
        hand: ["A", "B", "C", "D", "E", "F", "G"]
      },
      {
        name: "Player 2",
        hand: ["A", "B", "C", "D", "E", "F", "G"]
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
    }
  },
  mounted() {
    this.currentPlayer = this.players[0];
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
  width: 100%;
  text-align: center;
  // max-width: 500px;
  border: 1px solid black;
}

.game {
  display: flex;

  justify-content: space-between;
  width: 100%;
  margin: auto;

  background-color: #fff;

  &-palette {
    list-style-type: none;

    li {
      display: block;
    }
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

  &-player-one,
  &-player-two {
    width: 300px;
    text-align: center;
  }
}
</style>
