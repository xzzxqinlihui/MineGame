<template>
  <div id="app">
    <h1>这是扫雷游戏</h1>
    <div class="container">
      <div class="main">
        <table class="mine-table">
          <tr v-for="(rowItem, rIndex) of rows" :key="'row_' + rIndex">
            <!-- 转换成对应的单元格组件 -->
            <!-- <td
              :style="{ width: cellWidth + 'px', height: cellHeight + 'px' }"
              v-for="(colItem, cIndex) of cols"
              :key="'col_' + cIndex"
            ></td> -->
            <cell
              @clearboom="clearBoom"
              :cell-data="cellArray[rIndex * cols + cIndex]"
              :style="{ width: cellWidth + 'px', height: cellHeight + 'px' }"
              v-for="(colItem, cIndex) of cols"
              :key="'col_' + cIndex"
            ></cell>
          </tr>
        </table>
      </div>
      <div class="aside">
        <!-- S => 设置难易程度 -->
        <div>
          <el-radio-group @change="changeLevel" v-model="level" size="mini">
            <el-radio-button label="简单"></el-radio-button>
            <el-radio-button label="中级"></el-radio-button>
            <el-radio-button label="高级"></el-radio-button>
          </el-radio-group>
        </div>
        <!-- E => 设置难易程度 -->

        <!-- S => 设置行数和列数 -->
        <div class="block">
          <span class="demonstration">列数:{{ cols }}</span>
          <el-slider v-model="cols" :min="5" :max="100"></el-slider>
        </div>
        <div class="block">
          <span class="demonstration">行数:{{ rows }}</span>
          <el-slider v-model="rows" :min="5" :max="100"></el-slider>
        </div>
        <!-- E => 设置行数和列数 -->

        <!-- S => 设置是否行列同步 -->
        <div class="block">
          <el-switch
            style="display: block"
            v-model="isCellWHSync"
            active-color="#13ce66"
            inactive-color="#ff4949"
            active-text="同步宽高"
            inactive-text="不同步宽高"
          >
          </el-switch>
        </div>
        <!-- E => 设置是否行列同步 -->
        <!-- S => 设置行高行宽 -->
        <div class="block">
          <span class="demonstration">行高:{{ cellWidth }}</span>
          <el-slider v-model="cellWidth" :min="5"></el-slider>
        </div>
        <div class="block">
          <span class="demonstration">列高:{{ cellHeight }}</span>
          <el-slider v-model="cellHeight" :min="5"></el-slider>
        </div>
        <!-- S => 设置行高行宽 -->

        <!-- S => 设置重置按钮 -->
        <div class="block">
          <el-button @click="initCellData" type="info">
            重置
          </el-button>
        </div>
        <!-- E => 设置重置按钮 -->
      </div>
    </div>
    <play-sound src-sound="/a.wav"></play-sound>
    <timer :cellArray="cellArray"></timer>
  </div>
</template>

<script>
// import HelloWorld from "./components/HelloWorld.vue";
import "element-ui/lib/theme-chalk/index.css";
import Cell from "./components/Cell.vue";
import PlaySound from "./components/PlaySound.vue";
import Timer from "./components/Timer.vue";
import EventBus from "./eventBus.js";

export default {
  name: "app",
  data: function() {
    return {
      level: "简单", //  简单 1 中级 2   高级 3  100*0.15
      cols: 10,
      rows: 10,
      isCellWHSync: true,
      cellWidth: 30,
      cellHeight: 30,
      //  isBoom:false,data:2,isMarked:false,isClear:false
      cellArray: [],
      //  是否是重置状态
      isRest: true,
      isShowBoom: false
    };
  },
  components: {
    // HelloWorld
    cell: Cell,
    "play-sound": PlaySound,
    timer: Timer
  },
  watch: {
    cellWidth(newVal) {
      this.isCellWHSync && (this.cellHeight = newVal);
    },
    cellHeight(newVal) {
      this.isCellWHSync && (this.cellWidth = newVal);
    },
    rows() {
      this.initCellData();
    },
    cols() {
      this.initCellData();
    }
  },
  created() {
    this.initCellData();
    document.oncontextmenu = () => {
      return false;
    };
    EventBus.$on("click-cell", () => {
      this.isRest && EventBus.$emit("start-timer");
      this.isRest = false;
    });
    EventBus.$on("boom-end", () => {
      this.boomEnd();
    });
  },
  methods: {
    boomEnd() {
      //  把所有的累显示，另外把所有的非雷clear
      this.cellArray.forEach(item => {
        if (item.isBoom) {
          this.$set(item, "isShowBoom", true);
        } else {
          this.$set(item, "isClear", true);
        }
      });
    },
    changeLevel() {
      this.initCellData();
    },
    //  初始化数组数据
    initCellData() {
      //  设置重置状态
      this.isRest = true;
      //  动态初始化以下:cell数组,动态随机生成对应的地雷数据
      let sum = this.cols * this.rows;
      // // //  计算一共需要随机设置多少地雷
      let randomBooms = this.getLevelNum() * 0.15 * sum;
      let randomIndexSet = new Set(); // 它的元素不能重复
      while (randomIndexSet.size < randomBooms) {
        randomIndexSet.add(Math.trunc(Math.random() * sum));
      }
      this.cellArray = []; //清空数组中的元素
      // console.log(randomIndexSet);
      for (let i = 0; i < sum; i++) {
        let isBoom = randomIndexSet.has(i);
        let data = this.getBoomsNum(i, randomIndexSet);
        this.cellArray.push({
          isBoom: isBoom,
          data: data,
          isMarked: false,
          isClear: false,
          cellIndex: i
        }); //  建议
        // this.$set( this.cellArray,i,{}); //这种是 可以被监控的
      }
    },

    //  判断 level 难易程度
    getLevelNum() {
      if (this.level === "简单") {
        return 1;
      } else if (this.level === "中级") {
        return 2;
      } else {
        return 3;
      }
    },
    getBoomsNum(index, boomsSet) {
      let count = 0; //总共的炸弹数
      for (let i = -1; i <= 1; i++) {
        let startIndex = index - i * this.cols - 1;
        if (index % this.cols > 0) {
          count += boomsSet.has(startIndex);
        }
        count += boomsSet.has(startIndex + 1);
        if (index % this.cols < this.cols - 1) {
          count += boomsSet.has(startIndex + 2);
        }
      }
      return count;
    },
    //  清理雷区
    clearBoom(index) {
      const innerClearBoom = cIndex => {
        let cell = this.cellArray[cIndex];

        if (cIndex >= 0 && cIndex < this.cellArray.length) {
          if (cell.isMarked || cell.isBoom || cell.isClear) {
            return;
          }
          this.$set(this.cellArray[cIndex], "isClear", true);
          //  如果自己也是0  继续clear
          this.clearBoom(cIndex);
        }
      };

      //  如果当前自己不是0  那么就直接把自己清理掉
      let cell = this.cellArray[index];
      if (cell.data !== 0) {
        this.$set(cell, "isClear", true);
        return;
      }
      //  如果是0  那么周围8个都要清空 clear
      for (let i = -1; i <= 1; i++) {
        let startIndex = index - this.cols * i - 1;
        if (index % this.cols > 0) {
          innerClearBoom(startIndex);
        }

        innerClearBoom(startIndex + 1);

        if (index % this.cols < this.cols - 1) {
          innerClearBoom(startIndex + 2);
        }
        //  如果 周围的元素 也是0 那么此元素的周围也clear
      }
    }
  }
};
</script>

<style lang="scss">
#app {
  h1 {
    text-align: center;
  }
  .container {
    display: flex;
    & > div {
      border: 1px solid #ccc;
    }
  }
  .main {
    flex: 5 0 500px;
    min-height: 500px;
  }
  .aside {
    flex: 2 0 200px;
    min-height: 300px;
  }
  .mine-table {
    border-collapse: collapse;
    td {
      width: 30px;
      height: 30px;
      border: 1px solid #c09;
      background-color: aliceblue;
    }
  }
  .block {
    border-top: 1px solid #c09;
    margin-top: 5px;
  }
}
</style>
