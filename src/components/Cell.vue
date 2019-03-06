<template>
  <td
    :class="{
      'show-boom': cellData.isShowBoom,
      'mine-clear': cellData.isClear,
      'mark-ed': cellData.isMarked
    }"
    @mousedown="cellClick"
    :style="{ width: size.w + 'px', height: size.h + 'px' }"
    class="cell"
  >
    <!-- <template v-if="cellData.isBoom">
      B
    </template> -->
    <!-- <template v-else> -->
    <span
      :style="{ color: getNumColor(cellData.data) }"
      v-if="cellData.data != 0 && cellData.isClear"
      >{{ cellData.data }}</span
    >
    <!-- </template> -->
  </td>
</template>

<script>
import EventBus from "../eventBus.js";

export default {
  name: "cell",
  data() {
    return {};
  },
  props: {
    size: {
      type: Object,
      required: false,
      default: function() {
        return {
          w: 30,
          h: 30
        };
      }
    },
    cellData: {
      type: Object,
      required: true
    }
  },
  methods: {
    //
    getNumColor(num) {
      switch (num) {
        case 1:
          return "red";
        case 2:
          return "blue";
        case 3:
          return "yellow";
        case 4:
          return "green";
        case 5:
          return "orange";
        case 6:
          return "pink";
        case 7:
          return "purple";
        default:
          return "black";
      }
    },
    cellClick(e) {
      console.log(e.button);
      //  如果是点击了鼠标的右键，标记为marked状态
      if (this.cellData.isClear) {
        return; //  误操作 直接结束
      }
      if (e.button === 2) {
        EventBus.$emit("click-cell");
        this.$set(this.cellData, "isMarked", !this.cellData.isMarked);
        return;
      }
      //点击鼠标左键
      if (e.button === 0) {
        if (this.cellData.isBoom) {
          EventBus.$emit("boom-end");
          //  发送游戏结束的事件
          console.log("boom");
          alert("游戏失败");
        } else {
          // 如果标记了小旗子
          if (this.cellData.isMarked) {
            return;
          }
          EventBus.$emit("click-cell");
          //  不是炸弹  让当前的 单元格:显示数字
          // this.$set(this.cellData, "isClear", true);
          console.log("clear");
          //  把要 清理 的坐标  index发送给父容器
          this.$emit("clearboom", this.cellData.cellIndex);
        }
      }
    }
  }
};
</script>

<style lang="scss" scoped>
td {
  text-align: center;
  vertical-align: middle;
}
.mine-clear {
  background-color: #333 !important;
}
.mark-ed {
  background-image: url("/flag.png") !important;
  background-size: cover;
}
.show-boom {
  position: relative;
  &::before {
    content: "";
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-image: url("/bz.png") !important;
    background-size: contain;
  }
}
</style>
