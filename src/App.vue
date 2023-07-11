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

function gameStart() {
  randomTwoFillIn(2);
}

onMounted(() => {
  gameStart();
  window.addEventListener("keydown", keydownHandler);
});
</script>

<template>
  <h2>Score: {{ score }}</h2>
  <div class="board">
    <div v-for="(row, row_index) in gameBoard" :key="row_index" class="row">
      <TransitionGroup @enter="enterTransition" @leave="leaveTransition">
        <div v-for="(col, col_index) in row" :key="col_index" class="col" :data-row="row_index" :data-col="col_index">
          <Transition name="pop">
            <div
              v-if="col"
              class="box"
              :class="{
                'box-2': col === 2,
                'box-4': col === 4,
                'box-8': col === 8,
                'box-16': col === 16,
                'box-32': col === 32,
                'box-64': col === 64,
                'box-128': col === 128,
                'box-256': col >= 256,
              }"
            >
              {{ col }}
            </div>
          </Transition>
        </div>
      </TransitionGroup>
    </div>
  </div>
</template>

<style scoped lang="scss">
$boxRadius: 100px;
$gutter: 12px;

h2 {
  text-align: left;
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

.pop-enter-active,
.pop-leave-active {
  transition: all 0.15s ease;
}
.pop-enter-from,
.pop-leave-to {
  transform: scale(0);
}
</style>
