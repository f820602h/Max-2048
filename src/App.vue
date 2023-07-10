<script setup>
import { ref, onMounted } from "vue";

const rowCount = 4;
const colCount = 4;

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
  }
}

function countMove(frontWithoutEmptyArray, originNum, defaultMove, direction = 1) {
  let move = 0;
  move += direction * (defaultMove - frontWithoutEmptyArray.length);
  frontWithoutEmptyArray.forEach((num, index, arr) => {
    const isOdd = (index + 1) % 2 !== 0;
    const isEven = (index + 1) % 2 === 0;
    const isLastOne = index === arr.length - 1;

    const sameWithAfter = num === arr[index + 1];
    const sameWithBefore = num === arr[index - 1];
    const sameWithOrigin = num === originNum;

    if (isOdd && !isLastOne) {
      if (sameWithAfter) move += direction;
    } else if (isEven && !isLastOne) {
      if (!sameWithBefore && sameWithAfter) move += direction;
    } else if (isLastOne) {
      if (!sameWithBefore && sameWithOrigin) move += direction;
    }
  });
  return move;
}

function right() {
  gameBoard.value.forEach((row, rowIndex) => {
    row.forEach((col, colIndex, cols) => {
      if (!col) return;
      const frontWithoutEmpty = cols.filter((num, index) => !!num && index > colIndex);
      frontWithoutEmpty.reverse();
      moveX.value[rowIndex][colIndex] = countMove(frontWithoutEmpty, col, colCount - 1 - colIndex, 1);
    });
  });

  movementX();
}

function left() {
  gameBoard.value.forEach((row, rowIndex) => {
    row.forEach((col, colIndex, cols) => {
      if (!col) return;
      const frontWithoutEmpty = cols.filter((num, index) => !!num && index < colIndex);
      moveX.value[rowIndex][colIndex] = countMove(frontWithoutEmpty, col, colIndex, -1);
    });
  });

  movementX();
}

function top() {
  for (let colIndex = 0; colIndex < colCount; colIndex++) {
    for (let rowIndex = 0; rowIndex < rowCount; rowIndex++) {
      const originNum = gameBoard.value[rowIndex][colIndex];

      if (!originNum) continue;

      const frontWithoutEmpty = gameBoard.value
        .map((row) => row[colIndex])
        .filter((num, index) => !!num && index < rowIndex);

      moveY.value[rowIndex][colIndex] = countMove(frontWithoutEmpty, originNum, rowIndex, -1);
    }
  }

  movementY();
}

function bottom() {
  for (let colIndex = 0; colIndex < colCount; colIndex++) {
    for (let rowIndex = rowCount - 1; rowIndex >= 0; rowIndex--) {
      const originNum = gameBoard.value[rowIndex][colIndex];

      if (!originNum) continue;

      const frontWithoutEmpty = gameBoard.value
        .map((row) => row[colIndex])
        .filter((num, index) => !!num && index > rowIndex);
      frontWithoutEmpty.reverse();

      moveY.value[rowIndex][colIndex] = countMove(frontWithoutEmpty, originNum, rowCount - 1 - rowIndex, 1);
    }
  }

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
      duration: 150,
    });
  } else {
    animation = el.animate([], { duration: 150 });
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
      duration: 150,
      easing: "ease-in",
    });
  } else {
    animation = el.animate([], { duration: 150, easing: "ease-in" });
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

function gameStart() {
  randomTwoFillIn(2);
}

onMounted(() => {
  gameStart();
  window.addEventListener("keydown", keydownHandler);
});
</script>

<template>
  score: {{ score }}
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
