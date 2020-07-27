<template>
  <div class="home" ref="home">
    <div class="header">
      <div class="toolBar"></div>
      <div class="cellVal">
        <div class="cell">{{ clickCellInfo.position }}</div>
        <div class="val">{{ clickCellInfo.txt }}</div>
      </div>
    </div>
    <div class="main">
      <div
        class="allSelect"
        :style="{
          width: numCellWidth + 0.5 + 'px',
          height: cellHeight + 0.5 + 'px'
        }"
      ></div>
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
        >
          <div
            class="cellIpt"
            contenteditable
            v-if="cellIpt"
            ref="cellIpt"
            @blur="handlerCellIptBlur"
            @input="handlerCellIptIput"
          ></div>
        </div>
        <!-- 纵向滚动条 -->
        <div class="rightSideBar" ref="rightSideBar">
          <div
            class="scrollBar"
            :style="{
              top: rightScrollBar.top + 'px',
              height: rightScrollBar.height + 'px',
              opacity: scrollBarOpacity
            }"
            @mousedown="handlerScrollBarDragMove($event, true)"
          ></div>
        </div>
        <!-- 横向滚动条 -->
        <div class="bottomScrollBar" ref="bottomScrollBar">
          <div
            class="scrollBar"
            :style="{
              left: bottomScrollBar.left + 'px',
              width: bottomScrollBar.width + 'px',
              opacity: scrollBarOpacity
            }"
            @mousedown="handlerScrollBarDragMove($event, false)"
          ></div>
        </div>
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
      // 滚动条相关
      scrollBar: {
        x: 0,
        y: 0
      },
      rightScrollBar: {
        top: 0,
        height:
          ((window.innerHeight - 60 - 25) / (25 * 100)) *
          (window.innerHeight - 60 - 25)
      },
      bottomScrollBar: {
        left: 0,
        width:
          ((window.innerWidth - 40) / (120 * 26)) * (window.innerWidth - 40)
      },
      scrollBarMousedownPosition: {},
      // 滚动条透明度
      scrollBarOpacity: 0,
      scrollBarTimer: null,
      //表格数据
      centerCells: [],
      letfCells: [],
      headCells: [],
      //当前单击的cell
      clickCell: {
        x: 0,
        y: 0,
        show: false
      },
      //当前单击的cell信息
      clickCellInfo: {
        x: 0,
        y: 0,
        position: "",
        xNum: 0,
        yNum: 0,
        width: 0,
        height: 0,
        txt: ""
      },
      //当前双击的cell信息
      dbClickCellInfo: {
        x: 0,
        y: 0,
        position: "",
        xNum: 0,
        yNum: 0,
        width: 0,
        height: 0,
        txt: ""
      },
      //显示输入框
      cellIpt: false
    };
  },
  computed: {
    copyClickCellInfo() {
      return JSON.parse(JSON.stringify(this.clickCellInfo));
    },
    centerHeight() {
      return window.innerHeight - 60 - 25;
    },
    centerWidth() {
      return window.innerWidth - this.numCellWidth;
    }
  },
  methods: {
    init() {
      this.handlerDrawHead();
      this.handlerDrawLeft();
      this.handlerDrawCenter();
      this.handlerDomResize();
      this.handlerDomAddEvents();
    },
    //单元格输入的内容
    handlerCellIptBlur(e) {
      // console.log("blur");
      //ctx.measureText(txt).width
      let {
        centerCells,
        centerCanvas: { ctx },
        // clickCellInfo: ,
        dbClickCellInfo: { xNum, yNum, x, y, width, height },
        cellWidth,
        cellHeight,
        handlerPointAddFixed,
        handlerClearRect,
        handlerPointRect,
        ratio
      } = this;
      let txt = e.target.innerHTML;
      ctx.fillStyle = "rgb(38, 38, 38);";
      if (txt === centerCells[yNum][xNum].txt) {
        e.target.innerHTML = "";
      } else {
        centerCells[yNum][xNum].txt = txt;
        // 清除区域内容
        handlerClearRect(ctx, x, y, cellWidth, cellHeight);
        //补充上清除的边框
        handlerPointRect(ctx, x, y, width, height, "", null, "rgb(38, 38, 38)");
        // this.handlerPointHalfRect(ctx, x, y, width, height);
        // this.$nextTick(() => {
        this.handlerDrawTxt(ctx, x, y, width, height, txt);
        // });
        e.target.innerHTML = "";
      }
    },
    handlerCellIptIput(e) {
      this.clickCellInfo.txt += e.data;
    },
    handlerClearRect(ctx, x, y, cellWidth, cellHeight) {
      let { handlerPointAddFixed } = this;
      ctx.clearRect(
        ...handlerPointAddFixed(x + 1, y + 1),
        ...handlerPointAddFixed(cellWidth, cellHeight)
      );
      ctx.clearRect(
        ...handlerPointAddFixed(x, y),
        ...handlerPointAddFixed(cellWidth, cellHeight)
      );
    },
    //选中单元格
    handlerCenterCanvasMousedown(e) {
      // console.log(111);
      // this.handlerCenterClick(e);
      e.preventDefault();
      this.mousedownXY = {
        x: e.clientX,
        y: e.clientY
      };
      window.addEventListener("mousemove", this.handlerCanvasMousemove);
    },
    handlerCenterCanvasMouseup(e) {
      // console.log(222);
      e.preventDefault();
      let {
        mousedownXY: { x, y }
      } = this;
      let { clientX, clientY } = e;
      if (x == clientX && y == clientY) {
        this.handlerCenterClick(e);
      }
      window.removeEventListener("mousemove", this.handlerCanvasMousemove);
      return false;
    },
    handlerCanvasMousemove(e) {
      // console.log(e);
    },
    //双击单元格
    handlerDbClick() {
      this.cellIpt = true;
      // console.log("db");
      let {
        centerCells,
        clickCellInfo: { xNum, yNum }
      } = this;
      this.dbClickCellInfo = Object.assign({}, this.clickCellInfo);
      let range = window.getSelection();
      this.$nextTick(() => {
        this.$refs.cellIpt.focus();
        this.$refs.cellIpt.innerHTML = centerCells[yNum][xNum].txt;
        range.selectAllChildren(this.$refs.cellIpt);
        range.collapseToEnd();
      });
    },
    //单击单元格
    handlerCenterClick(e) {
      // console.log(333);
      // console.log(e);
      let {
        cellWidth,
        cellHeight,
        clickCell,
        clickCellInfo,
        scrollBar,
        cellIpt,
        centerCells,
        numCellWidth
      } = this;
      //判断当前是否编辑
      if (cellIpt) {
        this.cellIpt = false;
      }
      let { clientX, clientY } = e;
      let offsetX = clientX - numCellWidth;
      let offsetY = clientY - 60 - cellHeight;
      // let obj = {};
      clickCell.x = Math.floor(offsetX / cellWidth) * cellWidth + scrollBar.x;
      clickCellInfo.x = Math.floor(offsetX / cellWidth) * cellWidth;
      clickCell.y = Math.floor(offsetY / cellHeight) * cellHeight + scrollBar.y;
      clickCellInfo.y = Math.floor(offsetY / cellHeight) * cellHeight;
      clickCell.show = true;
      clickCellInfo.position = `${
        this.wordsHead[Math.floor(offsetX / cellWidth)]
      }${Math.floor(offsetY / cellHeight) + 1}`;
      clickCellInfo.xNum = Math.floor(offsetX / cellWidth);
      clickCellInfo.yNum = Math.floor(offsetY / cellHeight);
      clickCellInfo.width = cellWidth;
      clickCellInfo.height = cellHeight;
      clickCellInfo.txt =
        centerCells[clickCellInfo.yNum][clickCellInfo.xNum].txt;
    },
    handlerDomAddEvents() {
      window.addEventListener(
        "mousewheel",
        e => {
          this.handlerWheel(e);
        },
        { passive: false }
      );
      this.$refs.center.addEventListener(
        "dblclick",
        this.handlerDbClick,
        false
      );
      this.$refs.center.addEventListener(
        "mousedown",
        this.handlerCenterCanvasMousedown,
        false
      );
      this.$refs.center.addEventListener(
        "mouseup",
        this.handlerCenterCanvasMouseup,
        false
      );
    },
    handlerWheel(e) {
      e.preventDefault();
      e.returnValue = false;
      let { handlerScrollX, handlerScrollY } = this;
      this.scrollBarOpacity = 1;
      window.clearTimeout(this.scrollBarTimer);
      this.scrollBarTimer = window.setTimeout(() => {
        this.scrollBarOpacity = 0;
      }, 2000);
      const { deltaX, deltaY } = e;
      this.$nextTick(() => {
        if (Math.abs(deltaX) > Math.abs(deltaY)) {
          handlerScrollX(deltaX);
        } else {
          handlerScrollY(deltaY);
        }
      });
    },
    //横向滚动
    handlerScrollX(deltaX) {
      let {
        scrollBar,
        topCanvas,
        clickCell,
        clickCellInfo,
        bottomScrollBar,
        centerWidth
      } = this;
      scrollBar.x = scrollBar.x - deltaX;
      bottomScrollBar.left += (deltaX / (26 * 120)) * centerWidth;
      if (
        -scrollBar.x + parseInt(this.$refs.top.style.width) >=
        topCanvas.canvasWidth
      ) {
        scrollBar.x = -(
          topCanvas.canvasWidth - parseInt(this.$refs.top.style.width)
        );
        bottomScrollBar.left = centerWidth - bottomScrollBar.width;
      } else if (scrollBar.x >= 0) {
        scrollBar.x = 0;
        bottomScrollBar.left = 0;
      }
      if (clickCell.show) {
        clickCell.x = clickCellInfo.x + scrollBar.x;
      }
    },
    //纵向滚动
    handlerScrollY(deltaY) {
      let {
        scrollBar,
        leftCanvas,
        clickCell,
        clickCellInfo,
        rightScrollBar,
        centerHeight
      } = this;
      scrollBar.y -= deltaY;
      rightScrollBar.top += (deltaY / (100 * 25)) * (centerHeight - 3);
      if (
        -scrollBar.y + parseInt(this.$refs.left.style.height) >=
        leftCanvas.canvasHeight
      ) {
        scrollBar.y = -(
          leftCanvas.canvasHeight - parseInt(this.$refs.left.style.height)
        );
        rightScrollBar.top = centerHeight - 3 - rightScrollBar.height;
      } else if (scrollBar.y >= 0) {
        scrollBar.y = rightScrollBar.top = 0;
      }
      if (clickCell.show) {
        clickCell.y = clickCellInfo.y + scrollBar.y;
      }
    },
    // 拖动滚动条
    handlerScrollBarDragMove(e, type) {
      this.scrollBarMousedownPosition = {
        x: e.clientX,
        y: e.clientY,
        type
      };
      this.scrollBarOpacity = 1;
      window.clearTimeout(this.scrollBarTimer);
      document.addEventListener("mousemove", this.handlerMoveWidth);
      document.addEventListener("mouseup", () => {
        document.removeEventListener("mousemove", this.handlerMoveWidth);
        this.scrollBarTimer = window.setTimeout(() => {
          this.scrollBarOpacity = 0;
        }, 2000);
      });
    },
    // 根据滚动条滚动距离计算top,left
    handlerMoveWidth(e) {
      let {
        scrollBarMousedownPosition: bar,
        rightScrollBar,
        bottomScrollBar,
        centerHeight,
        scrollBar,
        leftCanvas,
        centerWidth,
        topCanvas,
        cellWidth,
        wordsHead
      } = this;
      if (bar.type) {
        let width = e.clientY - bar.y;
        bar.y = e.clientY;
        rightScrollBar.top += width;
        scrollBar.y -= (width / (centerHeight - 3)) * (100 * 25);
        if (rightScrollBar.top >= centerHeight - 3 - rightScrollBar.height) {
          rightScrollBar.top = centerHeight - 3 - rightScrollBar.height;
          scrollBar.y = -(
            leftCanvas.canvasHeight - parseInt(this.$refs.left.style.height)
          );
        } else if (rightScrollBar.top <= 0) {
          scrollBar.y = rightScrollBar.top = 0;
        }
      } else {
        let width = e.clientX - bar.x;
        bar.x = e.clientX;
        bottomScrollBar.left += width;
        scrollBar.x -= (width / centerWidth) * cellWidth * wordsHead.length;
        if (bottomScrollBar.left <= 0) {
          scrollBar.x = bottomScrollBar.left = 0;
        } else if (
          bottomScrollBar.left >=
          centerWidth - bottomScrollBar.width
        ) {
          bottomScrollBar.left = centerWidth - bottomScrollBar.width;
          scrollBar.x = -(
            topCanvas.canvasWidth - parseInt(this.$refs.top.style.width)
          );
        }
      }
    },
    handlerDomResize() {
      this.$refs.left.style.height = window.innerHeight - 60 - 26 + "px";
      this.$refs.center.style.height = window.innerHeight - 60 - 26 + "px";
      this.$refs.top.style.width = window.innerWidth - this.numCellWidth + "px";
      this.$refs.center.style.width =
        window.innerWidth - this.numCellWidth + "px";
    },
    handlerDrawHead() {
      let {
        cellWidth,
        cellHeight,
        topCanvas,
        wordsHead,
        headCells,
        handlerPointRect
      } = this;
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
        // ctx.fillStyle = "#FAFAFA";
        ctx.font = `normal ${12 * this.ratio}px PingFang SC`;
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        headCells.forEach(i => {
          let { x, y, width, height, txt } = i;
          handlerPointRect(ctx, x, y, width, height, txt, "#fafafa");
        });
      });
    },
    handlerDrawCenter() {
      let {
        centerCanvas,
        centerCells,
        wordsHead,
        cellWidth,
        cellHeight,
        handlerPointRect
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
            handlerPointRect(
              ctx,
              x,
              y,
              width,
              height,
              txt,
              null,
              "rgb(38, 38, 38)"
            );
          });
        });
      });
    },
    handlerDrawLeft() {
      let {
        numCellWidth,
        cellHeight,
        leftCanvas,
        letfCells,
        handlerPointRect
      } = this;
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
          handlerPointRect(ctx, x, y, width, height, txt, "#fafafa");
        });
      });
    },
    handlerLineAddFixed(x, y) {
      return [(x + 0.5) * this.ratio, (y + 0.5) * this.ratio];
    },
    handlerPointAddFixed(x, y) {
      return [x * this.ratio, y * this.ratio];
    },
    //绘制完整单元格
    handlerPointRect(ctx, x, y, width, height, txt, bgc, color) {
      ctx.fillStyle = bgc ? bgc : "#fff";
      ctx.beginPath();
      ctx.moveTo(...this.handlerLineAddFixed(x, y));
      ctx.lineTo(...this.handlerLineAddFixed(x + width, y));
      ctx.lineTo(...this.handlerLineAddFixed(x + width, y + height));
      ctx.lineTo(...this.handlerLineAddFixed(x, y + height));
      ctx.lineTo(...this.handlerLineAddFixed(x, y));
      ctx.fill();
      ctx.closePath();
      ctx.stroke();
      ctx.fillStyle = color ? color : "#000";
      if (txt.toString().length > 0) {
        ctx.fillText(
          txt,
          ...this.handlerPointAddFixed(x + width / 2, y + height / 2)
        );
      }
    },
    //绘制一半单元格
    handlerPointHalfRect(ctx, x, y, width, height) {
      ctx.strokeStyle = "#cecece";
      ctx.beginPath();
      ctx.moveTo(...this.handlerLineAddFixed(x, y));
      ctx.lineTo(...this.handlerLineAddFixed(x + width, y));
      ctx.moveTo(...this.handlerLineAddFixed(x, y));
      ctx.lineTo(...this.handlerLineAddFixed(x, y + height));
      ctx.closePath();
      ctx.stroke();
    },
    //绘制彩色边框 #2a83fa
    handlerDrawBorder(ctx, x, y, x1, y1) {
      ctx.strokeStyle = "#2a83fa";
      ctx.beginPath();
      ctx.moveTo(...this.handlerLineAddFixed(x, y));
      ctx.lineTo(...this.handlerLineAddFixed(x1, y1));
      ctx.closePath();
      ctx.stroke();
      ctx.strokeStyle = "#cecece";
    },
    //绘制文字
    handlerDrawTxt(ctx, x, y, width, height, txt) {
      let { cellWidth, ratio } = this;
      ctx.textAlign = "left";
      ctx.fillText(
        txt,
        ...this.handlerPointAddFixed(x + 5, y + height / 2),
        (cellWidth - 5) * ratio
      );
    },
    // 绘制点击单元格的行列头
    handlerDrawClickRowCol(val) {
      let { x, y, width, height, xNum, yNum } = val;
      let {
        leftCanvas,
        topCanvas,
        handlerPointAddFixed,
        handlerClearRect,
        numCellWidth,
        handlerPointRect,
        handlerDrawBorder,
        wordsHead
      } = this;
      let rowPosition = [0, y];
      let colPosition = [x, 0];
      // 清除当前行列头
      handlerClearRect(leftCanvas.ctx, ...rowPosition, numCellWidth, height);
      handlerClearRect(topCanvas.ctx, ...colPosition, width, height);
      // // 绘画新的行列头
      handlerPointRect(
        leftCanvas.ctx,
        ...rowPosition,
        numCellWidth,
        height,
        yNum + 1,
        "#E6E6E6"
      );
      handlerDrawBorder(
        leftCanvas.ctx,
        numCellWidth,
        y,
        numCellWidth,
        y + height
      );
      handlerPointRect(
        topCanvas.ctx,
        ...colPosition,
        width,
        height,
        wordsHead[xNum],
        "#E6E6E6"
      );
      handlerDrawBorder(topCanvas.ctx, x, height, x + width, height);
    },
    // 清除单元格行列头
    handlerClearOldClickRowCol(val, isX, isY) {
      let { x, y, width, height, xNum, yNum } = val;
      let {
        leftCanvas,
        topCanvas,
        handlerPointAddFixed,
        handlerClearRect,
        numCellWidth,
        handlerPointRect,
        wordsHead
      } = this;
      let rowPosition = [0, y];
      let colPosition = [x, 0];
      if (isX && isY) {
        // 清除当前行列头
        handlerClearRect(leftCanvas.ctx, ...rowPosition, numCellWidth, height);
        handlerClearRect(topCanvas.ctx, ...colPosition, width, height);
        // // 绘画行列头
        handlerPointRect(
          leftCanvas.ctx,
          ...rowPosition,
          numCellWidth,
          height,
          yNum + 1,
          "#fafafa"
        );
        handlerPointRect(
          topCanvas.ctx,
          ...colPosition,
          width,
          height,
          wordsHead[xNum],
          "#fafafa"
        );
      } else if (isX && !isY) {
        handlerClearRect(leftCanvas.ctx, ...rowPosition, numCellWidth, height);
        handlerPointRect(
          leftCanvas.ctx,
          ...rowPosition,
          numCellWidth,
          height,
          yNum + 1,
          "#fafafa"
        );
      } else if (!isX && isY) {
        handlerClearRect(topCanvas.ctx, ...colPosition, width, height);
        handlerPointRect(
          topCanvas.ctx,
          ...colPosition,
          width,
          height,
          wordsHead[xNum],
          "#fafafa"
        );
      }
    }
  },
  watch: {
    copyClickCellInfo: {
      handler: function(newVal, oldVal) {
        this.handlerDrawClickRowCol(newVal);
        if (oldVal.position) {
          if (newVal.x != oldVal.x && newVal.y != oldVal.y) {
            this.handlerClearOldClickRowCol(oldVal, true, true);
          } else if (newVal.x == oldVal.x && newVal.y != oldVal.y) {
            this.handlerClearOldClickRowCol(oldVal, true);
          } else if (newVal.x != oldVal.x && newVal.y == oldVal.y) {
            this.handlerClearOldClickRowCol(oldVal, null, true);
          }
        }
      },
      deep: true
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
    height: 60px;
    // background-color: pink;
    width: 100%;
    .toolBar {
      height: 40px;
      box-sizing: border-box;
      border-top: 1px solid #e8e8e8;
      border-bottom: 1px solid #e8e8e8;
    }
    .cellVal {
      height: 20px;
      display: flex;
      .cell {
        width: 40px;
        padding-left: 7px;
        text-align: center;
        line-height: 20px;
        height: 100%;
      }
      .val {
        flex: 1;
        margin-left: 24px;
        white-space: pre;
        overflow-x: auto;
        overflow-y: hidden;
        height: 100%;
        line-height: 20px;
      }
    }
  }
  .main {
    // margin-top: 20px;
    position: relative;
    .allSelect {
      position: absolute;
      left: 0px;
      top: 0px;
      border: 1px solid #cecece;
      box-sizing: border-box;
      z-index: 11;
    }
    .top {
      margin-left: 40px;
      // height: 25px;
      position: absolute;
      overflow: hidden;
      .topCanvas {
        position: absolute;
        z-index: 10;
      }
    }
    .left {
      position: absolute;
      overflow: hidden;
      .leftCanvas {
        position: absolute;
        z-index: 10;
      }
    }
    .center {
      position: absolute;
      overflow: hidden;
      .centerCanvas {
        position: absolute;
        // z-index: 10;
      }
      .clickCell {
        border: 2px solid #2a83fa;
        // box-shadow: 0 0 0 1px #2a83fa;
        position: absolute;
        box-sizing: border-box;
        z-index: 11;
        // background: #eff9ff;
        .cellIpt {
          z-index: 12;
          position: absolute;
          left: 0;
          top: 0;
          right: 0;
          bottom: 0;
          padding: 5px 6px;
          overflow: hidden;
          word-break: break-word;
          white-space: pre;
          width: 100%;
          background-color: rgb(255, 255, 255);
          color: rgb(38, 38, 38);
          resize: none;
          font-size: 12px;
          line-height: 1.14;
          outline: none;
          text-decoration: none;
          box-sizing: border-box;
        }
      }
      .rightSideBar {
        position: absolute;
        right: 3px;
        top: 0px;
        width: 10px;
        bottom: 3px;
        // background: pink;
        .scrollBar {
          position: absolute;
          width: 10px;
          background: rgba(0, 0, 0, 0.3);
          border-radius: 10px;
          cursor: pointer;
        }
      }
      .bottomScrollBar {
        position: absolute;
        bottom: 3px;
        left: 0px;
        height: 10px;
        right: 0px;
        .scrollBar {
          position: absolute;
          height: 10px;
          background: rgba(0, 0, 0, 0.3);
          border-radius: 10px;
          cursor: pointer;
        }
      }
    }
  }
}
</style>
