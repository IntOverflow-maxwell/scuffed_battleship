<template>
  <div> {{ this.inGameName }}'s board:</div>
  <canvas ref="Board"
          height="500"
          width="500"
          @click="handleClick"
          @mouseenter="this.mouseInside = true"
          @mouseleave="this.mouseInside = false"
          @mousemove="getMousePos"></canvas>
  <br/>
</template>

<script>
export default {
  name: 'BSBoard',
  props: {
    msg: String
  },
  data() {
    return {
      board: this.board,
      inGameName: this.inGameName
    }
  },
  created() {
    this.inGameName = "";
    this.buttonX = 0;
    this.buttonY = 0;
    this.mouseX = 0;
    this.mouseY = 0;
    this.mouseInside = false;
    this.ctx = undefined;
    this.board = [];
    for (let i = 0; i < 10; i++) {
      this.board.push([]);
      for (let j = 0; j < 10; j++) {
        this.board[i].push(".");
      }
    }
    this.colorMap = new Map();
    this.colorMap.set(".", "rgba(0, 0, 128, 1)")
        .set("#", "rgba(170, 170, 170, 1)")
        .set("X", "rgba(255, 0, 40, 1)")
        .set("O", "rgba(0, 0, 255, 1)");
  },
  async mounted() {
    await this.$nextTick();
    this.ctx = this.$refs.Board.getContext("2d");
    this.updateCanvas();
  },
  methods: {
    handleClick() {
      this.$emit("clicked", {
        x: this.buttonX,
        y: this.buttonY
      })
    },
    getMousePos(evt) {
      let canvas = this.$refs.Board;
      let rect = canvas.getBoundingClientRect(),
          scaleX = canvas.width / rect.width,
          scaleY = canvas.height / rect.height;  // relationship bitmap vs. element for y
      this.mouseX = (evt.clientX - rect.left) * scaleX;
      this.mouseY = (evt.clientY - rect.top) * scaleY;
      this.buttonX = Math.floor(this.mouseX / 50);
      this.buttonY = Math.floor(this.mouseY / 50);
    },
    updateCanvas() {
      this.ctx.clearRect(0, 0, this.$refs.Board.width, this.$refs.Board.height);
      for (let i = 0; i < 10; i++) {
        for (let j = 0; j < 10; j++) {
          this.ctx.beginPath();
          this.ctx.rect(i * 50, j * 50, 50, 50);
          this.ctx.fillStyle = this.colorMap.get(this.board[i][j]);
          this.ctx.fill();
          this.ctx.closePath();
        }
      }
      this.ctx.beginPath();
      for (let i = 0; i < 10; i++) {
        for (let j = 0; j < 10; j++) {
          this.ctx.strokeStyle = "rgba(255,255,255,0.2)";
          this.ctx.strokeRect(i * 50, j * 50, 50, 50);
        }
      }
      this.ctx.closePath();
      this.ctx.beginPath();
      if (this.mouseInside) {
        this.ctx.rect(this.buttonX * 50, this.buttonY * 50, 50, 50);
        this.ctx.fillStyle = "rgba(255,255,255,0.2)";
        this.ctx.fill();
      }
      this.ctx.closePath();
      window.requestAnimationFrame(this.updateCanvas);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
