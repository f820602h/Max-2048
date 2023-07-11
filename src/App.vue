<script setup>
import { ref, onMounted } from "vue";

const rowCount = 4;
const colCount = 4;
const animationDuration = 100;

const isRunning = ref(false);
const score = ref(0);

const gameBoard = ref(rowColValue());
const moveX = ref(rowColValue());
const moveY = ref(rowColValue());
const merge = ref(rowColValue());
const newResult = ref(rowColValue());

function rowColValue() {
  return Array(rowCount)
    .fill(null)
    .map((i) => Array(colCount).fill(0));
}

function setGameBoardToNewResult() {
  gameBoard.value = newResult.value;
  moveX.value = rowColValue();
  moveY.value = rowColValue();
  newResult.value = rowColValue();
}

function movementX() {
  let isMoved = false;

  moveX.value.forEach((row, r_index) => {
    row.forEach((move, c_index) => {
      const originNum = gameBoard.value[r_index][c_index];
      if (move !== 0) {
        isMoved = true;
      }
      if (newResult.value[r_index][c_index + move] === originNum && originNum !== 0) {
        merge.value[r_index][c_index + move] = 1;
        score.value += originNum * 2;
      }
      newResult.value[r_index][c_index + move] += originNum;
    });
  });

  if (isMoved) {
    gameBoard.value = Array(rowCount)
      .fill(null)
      .map((i) => Array(0));
  } else {
    setGameBoardToNewResult();
    isRunning.value = false;
  }
}

function movementY() {
  let isMoved = false;

  moveY.value.forEach((row, r_index) => {
    row.forEach((move, c_index) => {
      const originNum = gameBoard.value[r_index][c_index];
      if (move !== 0) {
        isMoved = true;
      }
      if (newResult.value[r_index + move][c_index] === originNum && originNum !== 0) {
        merge.value[r_index + move][c_index] = 1;
        score.value += originNum * 2;
      }
      newResult.value[r_index + move][c_index] += originNum;
    });
  });
function gameStart() {
  gameBoard.value = initRowColValue(0);
  randomTwoFillIn(2);
  resetStep();
}

  if (isMoved) {
    gameBoard.value = Array(rowCount)
      .fill(null)
      .map((i) => Array(0));
  } else {
    setGameBoardToNewResult();
    isRunning.value = false;
  }
}

function countMove(frontWithoutEmptyArray, defaultMove, direction = 1) {
  let move = 0;
  move += direction * (defaultMove - frontWithoutEmptyArray.length);
  frontWithoutEmptyArray.forEach((num, index, arr) => {
    let checkIndex = 1;
    let sameNumCount = 0;
    while (arr[index - checkIndex] === num) {
      checkIndex++;
      sameNumCount++;
    }
    if (sameNumCount % 2 !== 0) move += direction;
  });
  return move;
}

function right() {
  isRunning.value = true;
  gameBoard.value.forEach((row, rowIndex) => {
    row.forEach((col, colIndex, cols) => {
      if (!col) return;
      const frontWithoutEmpty = cols.filter((num, index) => !!num && index >= colIndex);
      frontWithoutEmpty.reverse();
      moveX.value[rowIndex][colIndex] = countMove(frontWithoutEmpty, colCount - colIndex, 1);
    });
  });

  movementX();
}

function left() {
  isRunning.value = true;
  gameBoard.value.forEach((row, rowIndex) => {
    row.forEach((col, colIndex, cols) => {
      if (!col) return;
      const frontWithoutEmpty = cols.filter((num, index) => !!num && index <= colIndex);
      moveX.value[rowIndex][colIndex] = countMove(frontWithoutEmpty, colIndex + 1, -1);
    });
  });

  movementX();
}

function top() {
  isRunning.value = true;

  gameBoard.value
    .map((row, rowIndex, rows) => {
      return rows.map((r) => r[rowIndex]);
    })
    .forEach((row, rowIndex) => {
      row.forEach((col, colIndex, cols) => {
        if (!col) return;
        const frontWithoutEmpty = cols.filter((num, index) => !!num && index <= colIndex);
        moveY.value[colIndex][rowIndex] = countMove(frontWithoutEmpty, colIndex + 1, -1);
      });
    });

  movementY();
}

function bottom() {
  isRunning.value = true;

  gameBoard.value
    .map((row, rowIndex, rows) => {
      return rows.map((r) => r[rowIndex]);
    })
    .forEach((row, rowIndex) => {
      row.forEach((col, colIndex, cols) => {
        if (!col) return;
        const frontWithoutEmpty = cols.filter((num, index) => !!num && index >= colIndex);
        frontWithoutEmpty.reverse();
        moveY.value[colIndex][rowIndex] = countMove(frontWithoutEmpty, colCount - colIndex, 1);
      });
    });

  movementY();
}

let completeElementCount = 0;

function enterTransition(el, done) {
  const row = el.dataset.row;
  const col = el.dataset.col;
  const box = el.querySelector(".box");

  let animation;

  if (merge.value[row][col] && box) {
    animation = box.animate([{ transform: "scale(1.1)" }, { transform: "scale(0.9)" }, { transform: "scale(1)" }], {
      duration: animationDuration,
    });
  } else {
    animation = el.animate([], { duration: animationDuration });
  }

  animation.onfinish = () => {
    done();
    merge.value[row][col] = 0;
    completeElementCount += 1;
    if (completeElementCount === rowCount * colCount) {
      completeElementCount = 0;
      randomTwoFillIn(1);
    }
  };
}

function leaveTransition(el, done) {
  const row = el.dataset.row;
  const col = el.dataset.col;
  const box = el.querySelector(".box");

  const offsetX = moveX.value[row][col] * 112 + "px";
  const offsetY = moveY.value[row][col] * 112 + "px";
  let animation;

  if (box) {
    animation = box.animate([{}, { transform: `translate(${offsetX},${offsetY})` }], {
      duration: animationDuration,
      easing: "ease-in",
    });
  } else {
    animation = el.animate([], { duration: animationDuration, easing: "ease-in" });
  }

  animation.onfinish = () => {
    done();
    completeElementCount += 1;
    if (completeElementCount === rowCount * colCount) {
      completeElementCount = 0;
      setGameBoardToNewResult();
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

  isRunning.value = false;
}

function keydownHandler(e) {
  if (isRunning.value) return;
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
  window.addEventListener("keydown", keydownHandler);
  gameStart();
});
</script>

<template>
  <div class="controller">
    <h2>Score: {{ score }}</h2>
    <button @click="gameStart">New Game</button>
  </div>
  <div class="board">
    <div v-for="(row, row_index) in gameBoard" :key="row_index" class="row">
      <TransitionGroup @enter="enterTransition" @leave="leaveTransition">
        <div v-for="(col, col_index) in row" :key="col_index" class="col" :data-row="row_index" :data-col="col_index">
          <Transition name="pop" @after-enter="gameOver">
            <div v-if="col" class="box" :class="`box-${col}`">{{ col }}</div>
          </Transition>
        </div>
      </TransitionGroup>
    </div>
  </div>
</template>

<style scoped lang="scss">
$boxRadius: 100px;
$gutter: 12px;

.controller {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

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
    font-size: 36px;
    font-weight: bold;
    text-align: center;
    color: f9f6f2;
    transition: 0.3s;

    $boxBg: (
      2: #eee4da,
      4: #ede0c8,
      8: #f2b179,
      16: #f59563,
      32: #f67c5f,
      64: #f65e3b,
      128: #edcf72,
      256: #edcc61,
      512: #edc850,
      1024: #edc53f,
      2048: #edc22e,
    );

    @each $key, $value in $boxBg {
      &.box-#{$key} {
        background-color: $value;
      }
    }

    &.box-2,
    &.box-4 {
      color: #756e66;
    }
  }
}

.pop-enter-active,
.pop-leave-active {
  transition: all 0.15s ease;
}
.pop-enter-from,
.pop-leave-to {
  transform: scale(0);
}
</style>
