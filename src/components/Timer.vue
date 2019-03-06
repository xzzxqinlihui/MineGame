<template>
  <div class="timer">{{ hh }} 小时 {{ mm }} 分钟 {{ ss }} 秒</div>
</template>

<script>
import EventBus from "../eventBus";
import eventBus from "../eventBus";
import { setInterval, clearInterval } from "timers";
export default {
  name: "timer",
  props: ["cellArray"],
  data() {
    return {
      hh: "00",
      mm: "00",
      ss: "00",
      startTimer: 0,
      endTimer: 0,
      timerId: 0,
      count: 0
    };
  },
  created() {
    EventBus.$on("boom-end", () => {
      this.endTimer = Date.now();
      this.showTimer(this.endTimer - this.startTimer);
      clearInterval(this.timerId);
      //  如果剩余不是炸弹的数量是0
      if (this.count != 0) {
        console.log("游戏失败");
        alert("游戏失败");
      } else {
        console.log("游戏成功");
        alert("游戏成功");
      }
    });
    eventBus.$on("start-timer", () => {
      this.startTimer = Date.now();
      this.timerId = setInterval(() => {
        let s = Date.now() - this.startTimer;
        this.showTimer(s);

        this.count = 0;
        for (let i = 0; i < this.cellArray.length; i++) {
          let item = this.cellArray[i];
          if (!item.isBoom) {
            this.count += !item.isClear;
          }
        }
        //  如果剩余不是炸弹的数量是0
        if (this.count == 0) {
          EventBus.$emit("boom-end");
        }
      }, 500);
    });
  },
  methods: {
    showTimer(s) {
      this.ss = String(Math.trunc(s / 1000) % 60).padStart(2, "0");
      this.mm = String(Math.trunc(s / 1000 / 60) % 60).padStart(2, "0");
      this.hh = String(Math.trunc(s / 1000 / 60 / 60) % 24).padStart(2, "0");
    },
    gameSuccess() {}
  }
};
</script>

<style lang="scss" scoped></style>
