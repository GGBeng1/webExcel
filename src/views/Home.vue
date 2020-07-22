<template>
  <div class="home" ref="home">
    <div class="header">
      header
    </div>
    <div class="main">
      <div
        class="top"
        ref="top"
        :style="{
          height: topCanvas.canvasHeight + 'px'
        }"
      >
        <canvas
          ref="topCanvas"
          class="topCanvas"
          :width="topCanvas.canvasWidth * ratio"
          :height="topCanvas.canvasHeight * ratio"
          :style="{
            width: topCanvas.canvasWidth + 'px',
            height: topCanvas.canvasHeight + 'px',
            left: scrollBar.x + 'px'
          }"
        ></canvas>
      </div>
      <div
        class="left"
        ref="left"
        :style="{
          width: numCellWidth + 1 + 'px',
          top: cellHeight + 'px'
        }"
      >
        <canvas
          ref="leftCanvas"
          class="leftCanvas"
          :width="leftCanvas.canvasWidth * ratio"
          :height="leftCanvas.canvasHeight * ratio"
          :style="{
            width: leftCanvas.canvasWidth + 'px',
            height: leftCanvas.canvasHeight + 'px',
            top: scrollBar.y + 'px'
          }"
        ></canvas>
      </div>
      <div
        class="center"
        ref="center"
        :style="{ top: cellHeight + 'px', left: numCellWidth + 'px' }"
      >
        <canvas
          ref="centerCanvas"
          class="centerCanvas"
          @click="handlerCenterClick"
          :width="centerCanvas.canvasWidth * ratio"
          :height="centerCanvas.canvasHeight * ratio"
          :style="{
            width: centerCanvas.canvasWidth + 'px',
            height: centerCanvas.canvasHeight + 'px',
            top: scrollBar.y + 'px',
            left: scrollBar.x + 'px'
          }"
        >
        </canvas>
        <div
          class="clickCell"
          v-show="clickCell.show"
          :style="{
            width: cellWidth + 0.5 + 'px',
            height: cellHeight + 0.5 + 'px',
            left: clickCell.x + 'px',
            top: clickCell.y + 'px'
          }"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      ratio: window.devicePixelRatio,
      topCanvas: {
        canvasWidth: 0,
        canvasHeight: 0,
        ctx: null
      },
      leftCanvas: {
        canvasWidth: 0,
        canvasHeight: 0,
        ctx: null
      },
      centerCanvas: {
        canvasWidth: 0,
        canvasHeight: 0,
        ctx: null
      },
      cellWidth: 120,
      cellHeight: 25,
      wordsHead: [
        "A",
        "B",
        "C",
        "D",
        "E",
        "F",
        "G",
        "H",
        "I",
        "J",
        "K",
        "L",
        "M",
        "N",
        "O",
        "P",
        "Q",
        "R",
        "S",
        "T",
        "U",
        "V",
        "W",
        "X",
        "Y",
        "Z"
      ],
      numCellWidth: 40,
      scrollBar: {
        x: 0,
        y: 0
      },
      //表格数据
      centerCells: [],
      letfCells: [],
      headCells: [],
      //当前点击的cell
      clickCell: {
        x: 0,
        y: 0,
        show: false
      },
      //当前点击的cell信息
      clickCellInfo: {}
    };
  },
  methods: {
    init() {
      this.handlerDrawHead();
      this.handlerDrawLeft();
      this.handlerDrawCenter();
      this.handlerDomResize();
      this.handlerDomAddEvents();
    },
    //单击单元格
    handlerCenterClick(e) {
      // console.log(e);
      let { cellWidth, cellHeight, clickCell, clickCellInfo, scrollBar } = this;
      let { offsetX, offsetY } = e;
      let obj = {};
      clickCell.x = Math.floor(offsetX / cellWidth) * cellWidth + scrollBar.x;
      obj.x = Math.floor(offsetX / cellWidth) * cellWidth;
      clickCell.y = Math.floor(offsetY / cellHeight) * cellHeight + scrollBar.y;
      obj.y = Math.floor(offsetY / cellHeight) * cellHeight;
      clickCell.show = true;
      obj.txt = `${this.wordsHead[Math.floor(offsetX / cellWidth)]}${Math.floor(
        offsetY / cellHeight
      ) + 1}`;
      Object.assign(clickCellInfo, obj);
    },
    handlerDomAddEvents() {
      window.addEventListener(
        "mousewheel",
        e => {
          this.handlerWheel(e);
        },
        { passive: false }
      );
    },
    handlerWheel(e) {
      e.preventDefault();
      e.returnValue = false;
      const { deltaX, deltaY } = e;
      this.$nextTick(() => {
        if (Math.abs(deltaX) > Math.abs(deltaY)) {
          this.handlerScrollX(deltaX);
        } else {
          this.handlerScrollY(deltaY);
        }
      });
    },
    //横向滚动
    handlerScrollX(deltaX) {
      let { scrollBar, topCanvas, clickCell, clickCellInfo } = this;
      scrollBar.x = scrollBar.x - deltaX;
      if (
        -scrollBar.x + parseInt(this.$refs.top.style.width) >=
        topCanvas.canvasWidth
      ) {
        scrollBar.x = -(
          topCanvas.canvasWidth - parseInt(this.$refs.top.style.width)
        );
      } else if (scrollBar.x >= 0) {
        scrollBar.x = 0;
      }
      if (clickCell.show) {
        clickCell.x = clickCellInfo.x + scrollBar.x;
      }
    },
    //纵向滚动
    handlerScrollY(deltaY) {
      let { scrollBar, leftCanvas, clickCell, clickCellInfo } = this;
      scrollBar.y -= deltaY;
      if (
        -scrollBar.y + parseInt(this.$refs.left.style.height) >=
        leftCanvas.canvasHeight
      ) {
        scrollBar.y = -(
          leftCanvas.canvasHeight - parseInt(this.$refs.left.style.height)
        );
      } else if (scrollBar.y >= 0) {
        scrollBar.y = 0;
      }
      if (clickCell.show) {
        clickCell.y = clickCellInfo.y + scrollBar.y;
      }
    },
    handlerDomResize() {
      this.$refs.left.style.height = window.innerHeight - 40 - 26 + "px";
      this.$refs.center.style.height = window.innerHeight - 40 - 26 + "px";
      this.$refs.top.style.width = window.innerWidth - this.numCellWidth + "px";
      this.$refs.center.style.width =
        window.innerWidth - this.numCellWidth + "px";
    },
    handlerDrawHead() {
      let { cellWidth, cellHeight, topCanvas, wordsHead, headCells } = this;
      let ctx = this.$refs.topCanvas.getContext("2d");
      topCanvas.ctx = ctx;
      // let allCell = [];
      for (let i = 0; i < wordsHead.length; i++) {
        let cell = {
          x: i * cellWidth,
          y: 0,
          width: cellWidth,
          height: cellHeight,
          txt: wordsHead[i]
        };
        headCells.push(cell);
      }
      topCanvas.canvasWidth = wordsHead.length * cellWidth + cellWidth / 2;
      topCanvas.canvasHeight = cellHeight + 1;
      this.$nextTick(() => {
        ctx.lineWidth = 1 * this.ratio;
        ctx.strokeStyle = "#cecece";
        ctx.font = `normal ${12 * this.ratio}px PingFang SC`;
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        headCells.forEach(i => {
          let { x, y, width, height, txt } = i;
          this.handlerPointRect(ctx, x, y, width, height, txt);
        });
      });
    },
    handlerDrawCenter() {
      let {
        centerCanvas,
        centerCells,
        wordsHead,
        cellWidth,
        cellHeight
      } = this;
      let ctx = this.$refs.centerCanvas.getContext("2d");
      centerCanvas.ctx = ctx;
      centerCanvas.canvasHeight = 100 * cellHeight + 1;
      centerCanvas.canvasWidth = wordsHead.length * cellWidth + cellWidth / 2;
      for (let i = 0; i < 100; i++) {
        let line = [];
        for (let j = 0; j < wordsHead.length; j++) {
          let cell = {
            x: j * cellWidth,
            y: i * cellHeight,
            width: cellWidth,
            height: cellHeight,
            txt: ""
          };
          line.push(cell);
        }
        centerCells.push(line);
      }
      this.$nextTick(() => {
        ctx.lineWidth = 1 * this.ratio;
        ctx.strokeStyle = "#cecece";
        ctx.font = `normal ${12 * this.ratio}px PingFang SC`;
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        centerCells.forEach(z => {
          z.forEach(i => {
            let { x, y, width, height, txt } = i;
            this.handlerPointRect(ctx, x, y, width, height, txt);
          });
        });
      });
    },
    handlerDrawLeft() {
      let { numCellWidth, cellHeight, leftCanvas, letfCells } = this;
      let ctx = this.$refs.leftCanvas.getContext("2d");
      leftCanvas.ctx = ctx;
      // let letfCells = [];
      for (let i = 1; i <= 100; i++) {
        let cell = {
          x: 0,
          y: (i - 1) * cellHeight,
          width: numCellWidth,
          height: cellHeight,
          txt: i
        };
        letfCells.push(cell);
      }
      leftCanvas.canvasWidth = numCellWidth + 1;
      leftCanvas.canvasHeight = letfCells.length * cellHeight + 1;
      this.$nextTick(() => {
        ctx.lineWidth = 1 * this.ratio;
        ctx.strokeStyle = "#cecece";
        ctx.font = `normal ${12 * this.ratio}px PingFang SC`;
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        letfCells.forEach(i => {
          let { x, y, width, height, txt } = i;
          this.handlerPointRect(ctx, x, y, width, height, txt, "num");
        });
      });
    },
    handlerLineAddFixed(x, y) {
      return [(x + 0.5) * this.ratio, (y + 0.5) * this.ratio];
    },
    handlerPointAddFixed(x, y) {
      return [x * this.ratio, y * this.ratio];
    },
    handlerPointRect(ctx, x, y, width, height, txt, type) {
      ctx.beginPath();
      ctx.moveTo(...this.handlerLineAddFixed(x, y));
      ctx.lineTo(...this.handlerLineAddFixed(x + width, y));
      ctx.moveTo(...this.handlerLineAddFixed(x + width, y));
      ctx.lineTo(...this.handlerLineAddFixed(x + width, y + height));
      ctx.moveTo(...this.handlerLineAddFixed(x, y));
      ctx.lineTo(...this.handlerLineAddFixed(x, y + height));
      ctx.moveTo(...this.handlerLineAddFixed(x, y + height));
      ctx.lineTo(...this.handlerLineAddFixed(x + width, y + height));
      ctx.stroke();
      if (txt.toString().length > 0) {
        if (!type) {
          ctx.fillText(
            txt,
            ...this.handlerPointAddFixed(x + width / 2, (y + height) / 2)
          );
        } else if (type == "num") {
          ctx.fillText(
            txt,
            ...this.handlerPointAddFixed(x + width / 2, y + height / 2)
          );
        }
      }
    }
  },
  mounted() {
    this.init();
  }
};
</script>
<style lang='scss'>
.home {
  width: 100%;
  height: 100%;
  .header {
    height: 40px;
    background-color: pink;
    width: 100%;
  }
  .main {
    // margin-top: 20px;
    position: relative;
    .top {
      margin-left: 40px;
      // height: 25px;
      position: absolute;
      overflow: hidden;
      .topCanvas {
        position: absolute;
      }
    }
    .left {
      position: absolute;
      overflow: hidden;
      .leftCanvas {
        position: absolute;
      }
    }
    .center {
      position: absolute;
      overflow: hidden;
      .centerCanvas {
        position: absolute;
      }
      .clickCell {
        border: 2px solid #2a83fa;
        // box-shadow: 0 0 0 1px #2a83fa;
        position: absolute;
        box-sizing: border-box;
      }
    }
  }
}
</style>
