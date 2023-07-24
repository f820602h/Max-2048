<script setup>
import { ref, computed, onMounted } from "vue";

const rowCount = 4;
const colCount = 4;
const animationDuration = 100;

const isRunning = ref(true);
const score = ref(0);

const gameBoard = ref(initRowColValue(0));
const move = ref(initRowColValue({ x: 0, y: 0 }));
const merge = ref(initRowColValue(0));
const newResult = ref(initRowColValue(0));

function createSameValueArray(length, value) {
  return Array(length)
    .fill(null)
    .map((i) => JSON.parse(JSON.stringify(value)));
}

function initRowColValue(value) {
  return createSameValueArray(rowCount, createSameValueArray(colCount, value));
}

function resetStep() {
  merge.value = initRowColValue(0);
  move.value = initRowColValue({ x: 0, y: 0 });
  newResult.value = initRowColValue(0);
  isRunning.value = false;
}

const isBoardFull = computed(() => {
  return gameBoard.value.every((row) => row.length === colCount && row.every((col) => !!col));
});

const isWin = computed(() => {
  return gameBoard.value.some((row) => row.some((col) => col === 2048));
});

const isLose = computed(() => {
  return (
    isBoardFull.value &&
    !isWin.value &&
    !gameBoard.value.some((row, rowIndex) =>
      row.some((col, colIndex) => {
        const nextRow = gameBoard.value[rowIndex + 1];
        const nextCol = gameBoard.value[rowIndex][colIndex + 1];
        return (nextRow && col === nextRow[colIndex]) || (nextCol && col === nextCol);
      })
    )
  );
});

function randomTwoFillIn(times) {
  function randomBetween(min, max) {
    return Math.floor(Math.random() * (max - min + 1) + min);
  }

  let time = 0;
  while (time < times) {
    if (isBoardFull.value) return;
    const randomRow = randomBetween(0, rowCount - 1);
    const randomCol = randomBetween(0, colCount - 1);
    if (!gameBoard.value[randomRow][randomCol]) {
      gameBoard.value[randomRow][randomCol] = 2;
      time += 1;
    }
  }
}

function gameStart() {
  gameBoard.value = initRowColValue(0);
  randomTwoFillIn(2);
  resetStep();
}

function gameOver() {
  if (isWin.value) {
    alert("You Win!");
  } else if (isLose.value) {
    alert("You Lose!");
  }
}

function countMove(array, direction = 1) {
  const withoutEmptyArray = array.filter((num, index) => !!num);

  let move = 0;
  move += direction * (array.length - withoutEmptyArray.length);
  withoutEmptyArray.forEach((num, index, arr) => {
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

function left() {
  gameBoard.value.forEach((row, rowIndex) => {
    row.forEach((col, colIndex, cols) => {
      if (!col) return;
      const sliceCol = cols.filter((num, index) => index <= colIndex);
      move.value[rowIndex][colIndex].x = countMove(sliceCol, -1);
    });
  });
}
function right() {
  gameBoard.value.forEach((row, rowIndex) => {
    row.forEach((col, colIndex, cols) => {
      if (!col) return;
      const sliceCol = cols.filter((num, index) => index >= colIndex).toReversed();
      move.value[rowIndex][colIndex].x = countMove(sliceCol);
    });
  });
}
function top() {
  gameBoard.value
    .map((row, rowIndex, rows) => {
      return rows.map((r) => r[rowIndex]);
    })
    .forEach((row, rowIndex) => {
      row.forEach((col, colIndex, cols) => {
        if (!col) return;
        const sliceCol = cols.filter((num, index) => index <= colIndex);
        move.value[colIndex][rowIndex].y = countMove(sliceCol, -1);
      });
    });
}
function bottom() {
  gameBoard.value
    .map((row, rowIndex, rows) => {
      return rows.map((r) => r[rowIndex]);
    })
    .forEach((row, rowIndex) => {
      row.forEach((col, colIndex, cols) => {
        if (!col) return;
        const sliceCol = cols.filter((num, index) => index >= colIndex).toReversed();
        move.value[colIndex][rowIndex].y = countMove(sliceCol);
      });
    });
}

function calcNewResult() {
  move.value.forEach((row, r_index) => {
    row.forEach((move, c_index) => {
      const originNum = gameBoard.value[r_index][c_index];
      if (newResult.value[r_index + move.y][c_index + move.x] === originNum && originNum !== 0) {
        merge.value[r_index + move.y][c_index + move.x] = 1;
        score.value += originNum * 2;
      }
      newResult.value[r_index + move.y][c_index + move.x] += originNum;
    });
  });
}

function keydownHandler(e) {
  if (isRunning.value) return;
  isRunning.value = true;

  switch (e.code) {
    case "ArrowLeft":
      left();
      break;
    case "ArrowRight":
      right();
      break;
    case "ArrowUp":
      top();
      break;
    case "ArrowDown":
      bottom();
      break;
    default:
      break;
  }

  calcNewResult();
  gameBoard.value = createSameValueArray(rowCount, []);
}

let completeElementCount = 0;
let movement = 0;

function enterTransition(el, done) {
  const row = el.dataset.row;
  const col = el.dataset.col;
  const box = el.querySelector(".box");
  const { x, y } = move.value[row][col];
  movement += x + y;

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
    completeElementCount += 1;
    if (completeElementCount === rowCount * colCount) {
      if (movement) randomTwoFillIn(1);
      resetStep();
      movement = 0;
      completeElementCount = 0;
    }
  };
}

function leaveTransition(el, done) {
  const row = el.dataset.row;
  const col = el.dataset.col;
  const box = el.querySelector(".box");
  const { x, y } = move.value[row][col];

  let animation;

  if (box) {
    animation = box.animate([{ transform: `translate(${x * 112 + "px"},${y * 112 + "px"})` }], {
      duration: animationDuration,
    });
  } else {
    animation = el.animate([], { duration: animationDuration });
  }

  animation.onfinish = () => {
    done();
    completeElementCount += 1;
    if (completeElementCount === rowCount * colCount) {
      gameBoard.value = newResult.value;
      completeElementCount = 0;
    }
  };
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
