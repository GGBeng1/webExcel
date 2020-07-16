<template>
  <div class="home">
    <div class="header">
      header
    </div>
    <div class="main">
      <canvas
        ref="canvas"
        :width="canvasWidth * ratio"
        :height="canvasHeight * ratio"
        :style="{
          width: canvasWidth + 'px',
          height: canvasHeight + 'px'
        }"
      ></canvas>
    </div>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      ratio: window.devicePixelRatio,
      canvasWidth: 0,
      canvasHeight: 0,
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
      ]
    };
  },
  methods: {
    init() {
      this.handlerCheckData();
    },
    handlerCheckData() {
      // this.wordsHead
      // let x = 0;
      // let y = 0;
      let row = [];
      for (let i = 0; i < 2; i++) {
        let line = [
          {
            x: 0,
            y: i * 20
          },
          {
            x: 80 * this.wordsHead.length,
            y: i * 20
          }
        ];
        // console.log(line);
        row.push(line);
      }
      let col = [];
      for (let i = 0; i < this.wordsHead.length + 1; i++) {
        let line = [
          {
            x: i * 80,
            y: 0
          },
          {
            x: i * 80,
            y: 20
          }
        ];
        // console.log(line);
        col.push(line);
      }
      // console.log(row);
      this.canvasWidth = this.wordsHead.length * 80 + 1;
      this.canvasHeight = 100;
      this.$nextTick(() => {
        let ctx = this.$refs.canvas.getContext("2d");
        ctx.lineWidth = 1 * this.ratio;
        ctx.strokeStyle = "#cecece";
        ctx.beginPath();
        row.forEach(i => {
          ctx.moveTo(...this.handlerLineAddFixed(i[0].x, i[0].y));
          ctx.lineTo(...this.handlerLineAddFixed(i[1].x, i[1].y));
        });
        col.forEach(i => {
          ctx.moveTo(...this.handlerLineAddFixed(i[0].x, i[0].y));
          ctx.lineTo(...this.handlerLineAddFixed(i[1].x, i[1].y));
        });
        ctx.stroke();
      });
    },
    handlerLineAddFixed(x, y) {
      return [(x + 0.5) * this.ratio, (y + 0.5) * this.ratio];
    }
  },
  mounted() {
    this.init();
  }
};
</script>
<style lang='scss'>
.home {
  .header {
    height: 40px;
    background-color: pink;
  }
  .main {
    margin-top: 20px;
  }
}
</style>
