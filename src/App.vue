<template>
  <h2>
    当前难度：{{
      level.current === 10 ? "菜🐔" : level.current === 6 ? "普通" : '"地狱"'
    }}
  </h2>
  <button @click="changeLevel(l)" v-for="(l, i) in level.levels" :key="i">
    {{ l === 10 ? "菜🐔" : l === 6 ? "普通" : '"地狱"' }}
  </button>
  <main>
    <div class="table">
      <div class="row" v-for="(row, index) in baseDate" :key="index">
        <div
          class="item"
          v-for="item in row"
          :key="item.id"
          @click.left="check(item)"
          @click.right.prevent="mark(item)"
          @touchstart="touchstartHandler(item)"
          @touchend="touchendHandler"
          :class="item.isShow ? 'show' : 'close'"
        >
          <span v-show="item.isMark">⛳</span>
          <span v-show="item.isShow">{{ item.data }}</span>
        </div>
      </div>
    </div>
    <button @click="reStart">开始游戏</button>
  </main>
</template>

<script>
import { defineComponent, onMounted, ref, reactive } from "vue";
import { nanoid } from "nanoid";

const itemTemplate = {
  data: null,
  isBomb: null,
  isMark: false,
  isShow: false,
  id: "",
  y: null,
  x: null,
};
// ⛳ 💥 Bomb

export default defineComponent({
  setup() {
    const baseDate = ref([]);
    const resultArr = ref([]);
    const guessArr = ref([]);
    const level = reactive({
      current: 10,
      levels: [10, 6, 3],
    });

    function gameInit() {
      let arr = [];
      baseDate.value.length = 0;
      resultArr.value.length = 0;
      guessArr.value.length = 0;
      //初始化数据
      for (let y = 0; y < 10; y++) {
        for (let x = 0; x < 10; x++) {
          itemTemplate.id = nanoid();
          itemTemplate.y = y;
          itemTemplate.x = x;
          arr.push(JSON.parse(JSON.stringify(itemTemplate)));
          if (x === 9) {
            baseDate.value.push(arr);
            arr = [];
          }
        }
      }
      //初始化炸弹
      baseDate.value.forEach((row) => {
        row.forEach((item) => {
          item.isBomb =
            Math.ceil(Math.random() * level.current) === 3 ? true : false;
          item.data = item.isBomb ? "💥" : "";
          if (item.isBomb) {
            resultArr.value.push(item.id);
          }
        });
      });
      //初始化数字
      for (let y = 0; y < 10; y++) {
        for (let x = 0; x < 10; x++) {
          checkAround(y, x);
        }
      }
      /* 
    00 01 02
    10 11 12
    20 21 22
    */
      function checkAround(y, x) {
        let count = null;
        if (baseDate.value[y][x].isBomb) return;
        if (y !== 0 && x !== 0 && baseDate.value[y - 1][x - 1].isBomb) {
          //左上
          count += 1;
        }

        if (y !== 0 && baseDate.value[y - 1][x].isBomb) {
          //上
          count += 1;
        }
        if (y !== 0 && x !== 9 && baseDate.value[y - 1][x + 1].isBomb) {
          //右上
          count += 1;
        }
        if (x !== 0 && baseDate.value[y][x - 1].isBomb) {
          //左
          count += 1;
        }
        if (x !== 9 && baseDate.value[y][x + 1].isBomb) {
          //右
          count += 1;
        }
        if (y !== 9 && x !== 0 && baseDate.value[y + 1][x - 1].isBomb) {
          //左下
          count += 1;
        }
        if (y !== 9 && baseDate.value[y + 1][x].isBomb) {
          //下
          count += 1;
        }
        if (y !== 9 && x !== 9 && baseDate.value[y + 1][x + 1].isBomb) {
          //右下
          count += 1;
        }
        if (count) {
          baseDate.value[y][x].data = count;
        }
      }
    }

    function changeLevel(l) {
      level.current = l;
      gameInit()
    }

    function check(item) {
      item.isShow = true;
      if (item.isMark) {
        item.isMark = false;
        guessArr.value = guessArr.value.filter((id) => id !== item.id);
      }
      const { y, x } = item;
      if (item.data !== "💥") {
        for (let CY = y; CY < 10; CY++) {
          for (let CX = x; CX < 10; CX++) {
            if (
              !baseDate.value[CY][CX].data &&
              !baseDate.value[CY][CX].isMark
            ) {
              baseDate.value[CY][CX].isShow = true;
            } else if (
              baseDate.value[CY][CX].data &&
              baseDate.value[CY][CX].data !== "💥" &&
              !baseDate.value[CY][CX].isMark
            ) {
              baseDate.value[CY][CX].isShow = true;
              break;
            }
          }
          for (let CX = x; CX > -1; CX--) {
            if (
              !baseDate.value[CY][CX].data &&
              !baseDate.value[CY][CX].isMark
            ) {
              baseDate.value[CY][CX].isShow = true;
            } else if (
              baseDate.value[CY][CX].data &&
              baseDate.value[CY][CX].data !== "💥" &&
              !baseDate.value[CY][CX].isMark
            ) {
              baseDate.value[CY][CX].isShow = true;
              break;
            }
          }
        }
        for (let CY = y; CY > -1; CY--) {
          for (let CX = x; CX < 10; CX++) {
            if (
              !baseDate.value[CY][CX].data &&
              !baseDate.value[CY][CX].isMark
            ) {
              baseDate.value[CY][CX].isShow = true;
            } else if (
              baseDate.value[CY][CX].data &&
              baseDate.value[CY][CX].data !== "💥" &&
              !baseDate.value[CY][CX].isMark
            ) {
              baseDate.value[CY][CX].isShow = true;
              break;
            }
          }
          for (let CX = x; CX > -1; CX--) {
            if (
              !baseDate.value[CY][CX].data &&
              !baseDate.value[CY][CX].isMark
            ) {
              baseDate.value[CY][CX].isShow = true;
            } else if (
              baseDate.value[CY][CX].data &&
              baseDate.value[CY][CX].data !== "💥" &&
              !baseDate.value[CY][CX].isMark
            ) {
              baseDate.value[CY][CX].isShow = true;
              break;
            }
          }
        }
      } else {
        gameOver();
      }
    }

    function mark(item) {
      item.isMark = !item.isMark;
      if (item.isMark) {
        guessArr.value.push(item.id);
      } else {
        guessArr.value = guessArr.value.filter((id) => id !== item.id);
      }
      if (guessArr.value.length === resultArr.value.length) {
        if (isWin()) {
          baseDate.value.forEach((row) => {
            row.forEach((item) => {
              if (item.isMark) item.data = "⛳";
              item.isShow = true;
              item.isMark = false;
            });
          });
          const t = setTimeout(() => {
            alert("恭喜🎉🎉恭喜🎉🎉");
          }, 0);
        }
      }
    }

    let timer;
    function touchstartHandler(item) {
      timer = setTimeout(() => {
        mark(item);
      }, 500);
    }
    function touchendHandler() {
      clearTimeout(timer);
    }

    function isWin() {
      return guessArr.value.every((item) => {
        return resultArr.value.includes(item);
      });
    }
    function gameOver() {
      baseDate.value.forEach((row) => {
        row.forEach((item) => {
          item.isShow = true;
          item.isMark = false;
        });
      });
      const t = setTimeout(() => {
        alert("游戏结束💥💥💥");
      }, 0);
    }

    function reStart() {
      gameInit();
    }
    onMounted(() => {
      gameInit();
    });

    return {
      baseDate,
      check,
      mark,
      touchstartHandler,
      touchendHandler,
      reStart,
      level,
      changeLevel,
    };
  },
});
</script>

<style lang="scss">
button {
  margin: 5px;
}
main {
  background-color: #efefef;

  .table {
    display: flex;
    flex-wrap: wrap;
    width: 420px;

    .row {
      display: flex;
      flex-wrap: wrap;

      &::before,
      &::after {
        content: "";
        display: table;
        clear: both;
      }

      .item {
        width: 30px;
        height: 30px;
        margin: 5px;
        cursor: pointer;
        color: rgb(209, 4, 4);
        font-weight: bold;
        display: flex;
        justify-content: center;
        align-items: center;
        border: solid #7d7d7d 1px;
      }

      .close {
        background-color: #7d7d7d;
      }

      .show {
        border: dashed #5e5e5e 1px;
      }
    }
  }
}
@media only screen and (max-width: 425px) {
  main .table {
    width: 390px;
  }
  main .table .row .item {
    width: 31px;
    height: 31px;
    margin: 3px;
  }
}
</style>
