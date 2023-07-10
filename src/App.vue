<script setup>
import { ref, computed, onMounted, watch } from "vue";

const rowCount = 4;
const colCount = 4;

function rowColValue() {
  return Array(rowCount)
    .fill(null)
    .map((i) => Array(colCount).fill(0));
}

const gameBoard = ref(rowColValue());
const moveX = ref(rowColValue());
const moveY = ref(rowColValue());
const merge = ref(rowColValue());
const newResult = ref(rowColValue());

function setGameBoardToNewResult() {
  gameBoard.value = newResult.value;
  moveX.value = rowColValue();
  moveY.value = rowColValue();
  merge.value = rowColValue();
}

function gameStart() {
  randomTwoFillIn(2);
}

function movement(direction = "x") {
  let isMoved = false;

  moveX.value.forEach((row, r_index) => {
    row.forEach((move, c_index) => {
      if (move !== 0) {
        isMoved = true;

        if (newResult.value[r_index][c_index + move] === gameBoard.value[r_index][c_index]) {
          merge.value[r_index][c_index + move] = 1;
        }
        newResult.value[r_index][c_index + move] += gameBoard.value[r_index][c_index];
      }
    });
  });

  moveY.value.forEach((row, r_index) => {
    row.forEach((move, c_index) => {
      if (move !== 0) {
        isMoved = true;

        if (newResult.value[r_index + move][c_index] === gameBoard.value[r_index][c_index]) {
          merge.value[r_index + move][c_index] = 1;
        }

        newResult.value[r_index + move][c_index] += gameBoard.value[r_index][c_index];
      }
    });
  });

  if (isMoved) {
    gameBoard.value = Array(rowCount)
      .fill(null)
      .map((i) => Array(0));
  } else {
    setGameBoardToNewResult();
  }
}

function right() {
  for (let rowIndex = 0; rowIndex < rowCount; rowIndex++) {
    for (let colIndex = colCount - 1; colIndex >= 0; colIndex--) {
      const originNum = gameBoard.value[rowIndex][colIndex];

      if (!originNum) continue;

      let move = 0;
      const frontWithoutEmpty = gameBoard.value[rowIndex].filter((num, index) => !!num && index > colIndex);
      frontWithoutEmpty.reverse();

      move += colCount - 1 - colIndex - frontWithoutEmpty.length;
      frontWithoutEmpty.forEach((num, index, arr) => {
        if ((index + 1) % 2) {
          if (num === arr[index + 1]) move += 1;
          else if (index === arr.length - 1 && num === originNum) move += 1;
        } else {
          if (index === arr.length - 1 && num === originNum && num !== arr[index - 1]) move += 1;
        }
      });

      moveX.value[rowIndex][colIndex] = move;
    }
  }

  movement();
}

function left() {
  for (let rowIndex = 0; rowIndex < rowCount; rowIndex++) {
    for (let colIndex = 0; colIndex < colCount; colIndex++) {
      const originNum = gameBoard.value[rowIndex][colIndex];

      if (!originNum) continue;

      let move = 0;
      const frontWithoutEmpty = gameBoard.value[rowIndex].filter((num, index) => !!num && index < colIndex);

      move -= colIndex - frontWithoutEmpty.length;
      frontWithoutEmpty.forEach((num, index, arr) => {
        if ((index + 1) % 2) {
          if (num === arr[index + 1]) move -= 1;
          else if (index === arr.length - 1 && num === originNum) move -= 1;
        } else {
          if (index === arr.length - 1 && num === originNum && num !== arr[index - 1]) move -= 1;
        }
      });

      moveX.value[rowIndex][colIndex] = move;
    }
  }

  movement();
}

function top() {
  for (let colIndex = 0; colIndex < colCount; colIndex++) {
    for (let rowIndex = 0; rowIndex < rowCount; rowIndex++) {
      const originNum = gameBoard.value[rowIndex][colIndex];

      if (!originNum) continue;

      let move = 0;
      const frontWithoutEmpty = gameBoard.value
        .map((row) => row[colIndex])
        .filter((num, index) => !!num && index < rowIndex);

      move -= rowIndex - frontWithoutEmpty.length;
      frontWithoutEmpty.forEach((num, index, arr) => {
        if ((index + 1) % 2) {
          if (num === arr[index + 1]) move -= 1;
          else if (index === arr.length - 1 && num === originNum) move -= 1;
        } else {
          if (index === arr.length - 1 && num === originNum && num !== arr[index - 1]) move -= 1;
        }
      });

      moveY.value[rowIndex][colIndex] = move;
    }
  }

  movement();
}

function bottom() {
  for (let colIndex = 0; colIndex < colCount; colIndex++) {
    for (let rowIndex = rowCount - 1; rowIndex >= 0; rowIndex--) {
      const originNum = gameBoard.value[rowIndex][colIndex];

      if (!originNum) continue;

      let move = 0;
      const frontWithoutEmpty = gameBoard.value
        .map((row) => row[colIndex])
        .filter((num, index) => !!num && index > rowIndex);
      frontWithoutEmpty.reverse();

      move += rowCount - 1 - rowIndex - frontWithoutEmpty.length;
      frontWithoutEmpty.forEach((num, index, arr) => {
        if ((index + 1) % 2) {
          if (num === arr[index + 1]) move += 1;
          else if (index === arr.length - 1 && num === originNum) move += 1;
        } else {
          if (index === arr.length - 1 && num === originNum && num !== arr[index - 1]) move += 1;
        }
      });

      moveY.value[rowIndex][colIndex] = move;
    }
  }

  movement();
}

function keydownHandler(e) {
  if (e.code === "ArrowLeft") {
    left();
  } else if (e.code === "ArrowRight") {
    right();
  } else if (e.code === "ArrowUp") {
    top();
  } else if (e.code === "ArrowDown") {
    bottom();
  }
}

onMounted(() => {
  gameStart();
  window.addEventListener("keydown", keydownHandler);
});

function enterTransition(el, done) {
  const row = el.dataset.row;
  const col = el.dataset.col;
  if (!merge.value[row][col]) return done();

  const animation = el
    .querySelector(".box")
    .animate([{ transform: "scale(1.1)" }, { transform: "scale(0.9)" }, { transform: "scale(1)" }], { duration: 150 });
  animation.onfinish = () => {
    done();
  };
}

let completeElementCount = 0;

function leaveTransition(el, done) {
  const row = el.dataset.row;
  const col = el.dataset.col;

  const offsetX = moveX.value[row][col] * 112 + "px";
  const offsetY = moveY.value[row][col] * 112 + "px";

  const animation = el.querySelector(".box").animate([{}, { transform: `translate(${offsetX},${offsetY})` }], {
    duration: 100,
    easing: "ease-in",
  });
  animation.onfinish = () => {
    done();
    completeElementCount += 1;
    if (completeElementCount === rowCount * colCount) {
      console.log(1);
      completeElementCount = 0;
      setGameBoardToNewResult();
      randomTwoFillIn(1);
    }
  };
}

function randomBetween(min, max) {
  return Math.floor(Math.random() * (max - min + 1) + min);
}

function randomTwoFillIn(times) {
  let time = 0;
  while (time < times) {
    const randomRow = randomBetween(0, rowCount - 1);
    const randomCol = randomBetween(0, colCount - 1);
    if (!gameBoard.value[randomRow][randomCol]) {
      gameBoard.value[randomRow][randomCol] = 2;
      time += 1;
    }
  }
}
</script>

<template>
  <div class="board">
    <div v-for="(row, row_index) in gameBoard" :key="row_index" class="row">
      <TransitionGroup @enter="enterTransition" @leave="leaveTransition">
        <div v-for="(col, col_index) in row" :key="col_index" class="col" :data-row="row_index" :data-col="col_index">
          <div
            class="box"
            :class="{
              'box-2': col === 2,
              'box-4': col === 4,
              'box-8': col === 8,
              'box-16': col === 16,
              'box-32': col === 32,
              'box-64': col === 64,
              'box-128': col === 128,
              'box-256': col === 256,
            }"
          >
            {{ col ? col : "" }}
          </div>
        </div>
      </TransitionGroup>
    </div>
  </div>
</template>

<style scoped lang="scss">
$boxRadius: 100px;
$gutter: 12px;
.board {
  background: #b9ada1;
  border-radius: 8px;
}
.row {
  display: flex;
  justify-content: space-between;
  width: $boxRadius * 4 + $gutter * 3;
  padding: $gutter $gutter 0 $gutter;

  &:last-child {
    padding-bottom: $gutter;
  }
}

.col {
  width: $boxRadius;
  height: $boxRadius;
  border-radius: 8px;
  background: #cac1b5;

  .box {
    font-family: "Clear Sans", "Helvetica Neue", Arial, sans-serif;
    width: $boxRadius;
    height: $boxRadius;
    border-radius: 8px;
    line-height: $boxRadius;
    font-size: 42px;
    font-weight: bold;
    text-align: center;
    color: white;
    transition: 0.3s;
    &.box-2 {
      background-color: #eee4da;
      color: #756e66;
    }

    &.box-4 {
      background-color: #ede0c8;
      color: #756e66;
    }

    &.box-8 {
      background-color: #f2b179;
    }

    &.box-16 {
      background-color: #f59563;
    }

    &.box-32 {
      background-color: #f67c5f;
      transform: scale();
    }

    &.box-64 {
      background-color: #f65e3b;
    }

    &.box-128 {
      background-color: #edcf72;
    }

    &.box-256 {
      background-color: #edcc61;
    }
  }
}

.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
</style>
