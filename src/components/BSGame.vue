<template>
  <button @click="ready">{{ readyMsg }}</button>
  <br>
  <BSBoard v-for="board in boards" :key="board.id" ref="boards" @clicked="handleClick"></BSBoard>
</template>

<script>
import BSBoard from "./BSBoard.vue"

export default {
  name: "BSGame",
  components: {
    BSBoard
  },
  async created() {
    this.readyMsg = "Not Ready..."
    this.player = {
      id: 0,
      state: "NR",
      inGameName: window.inGameName,
      hitBoard: [],
      turn: false
    };
    this.boards = [];
    this.playerCount = 0;
    for (let i = 0; i < 10; i++) {
      this.player.hitBoard.push([]);
      for (let j = 0; j < 10; j++) {
        this.player.hitBoard[i].push(0);
      }
    }
    this.shooting = false;
    this.shootX = 0;
    this.shootY = 0;
    this.tileMap = new Map();
    this.tileMap.set(0, ".")
        .set(1, ".")
        .set(2, "O")
        .set(3, "X");
    await this.initPlayer();
    await this.initGameEvent();
  },
  data() {
    return {
      boards: this.boards,
      playerCount: this.playerCount,
      readyMsg: this.readyMsg
    }
  },
  methods: {
    handleClick(e) {
      if (this.player.state === "NR") {
        let currentP = this.$refs.boards[this.player.id];
        currentP.board[e.x][e.y] = "#";
        return;
      }
      if (this.player.turn) {
        this.shooting = true;
        this.shootX = e.x;
        this.shootY = e.y;
      }
    },
    ready() {
      if (this.readyMsg === "Not Ready...") {
        this.readyMsg = "Ready!";
        this.player.state = "R";
      } else {
        this.readyMsg = "Not Ready...";
        this.player.state = "NR";
      }
      this.update();
    },
    async initPlayer() {
      let response = fetch("https://demo.httprelay.io/mcast/IntOverflowBS" + window.serverCode);
      const p2 = new Promise((_r, rej) => setTimeout(() => rej("p2"), 20000));
      let resp;
      try {
        resp = await Promise.race([response, p2]);
      } catch (e) {
        let requestOptions = {
          method: "POST",
          headers: {"Content-Type": "application/json"},
          body: '{"players": []}'
        };
        await fetch("https://demo.httprelay.io/mcast/IntOverflowBS" + window.serverCode, requestOptions);
        await this.initPlayer();
        return;
      }
      let rdata = await (await resp).text();
      let msg = JSON.parse(rdata);
      this.player.id = msg.players.length;
      msg.players.push(this.player);
      let requestOptions = {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify(msg)
      };
      await fetch("https://demo.httprelay.io/mcast/IntOverflowBS" + window.serverCode, requestOptions);
    },
    async update() {
      const response = await fetch("https://demo.httprelay.io/mcast/IntOverflowBS" + window.serverCode);
      const rdata = await response.text();
      let msg = JSON.parse(rdata);
      this.updatePlayerState(msg);
      if (this.shooting) {
        this.handleShot(msg);
      }
      this.updateBoards(msg);
      console.log(JSON.stringify(msg));
      let requestOptions = {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify(msg)
      };
      await fetch("https://demo.httprelay.io/mcast/IntOverflowBS" + window.serverCode, requestOptions);
      if (this.player.state === "P") setTimeout(this.update, 200);
    },
    updatePlayerState(msg) {
      let ans = 0;
      for (let i = 0; i < msg.players.length; i++) {
        if (msg.players[i].turn) ans++;
      }
      if (ans !== 1 && this.player.id === 0) {
        msg.players[0].turn = true;
      }
      this.player.turn = msg.players[this.player.id].turn;
      let currentP = this.$refs.boards[this.player.id];
      let updatedHitBoard = msg.players[this.player.id].hitBoard;
      for (let i = 0; i < 10; i++) {
        for (let j = 0; j < 10; j++) {
          if (updatedHitBoard[i][j] !== 1) continue;
          if (currentP.board[i][j] === "#") {
            currentP.board[i][j] = "X";
            updatedHitBoard[i][j] = 3;
            continue;
          }
          currentP.board[i][j] = "O";
          updatedHitBoard[i][j] = 2;
        }
      }
      msg.players[this.player.id] = this.player;
      msg.players[this.player.id].hitBoard = updatedHitBoard;
    },
    handleShot(msg) {
      if (!this.player.turn) return;
      for (let i = 0; i < msg.players.length; i++) {
        if (i === this.player.id) continue;
        msg.players[i].hitBoard[this.shootX][this.shootY] = 1;
      }
      msg.players[(this.player.id + 1) % (msg.players.length)].turn = true;
      msg.players[this.player.id].turn = false;
      this.player.turn = false;
      this.shooting = false;
    },
    updateBoards(msg) {
      let currentP;
      for (let i = 0; i < msg.players.length; i++) {
        currentP = this.$refs.boards[i];
        if (i === this.player.id) continue;
        for (let j = 0; j < 10; j++) {
          for (let k = 0; k < 10; k++) {
            currentP.board[j][k] = this.tileMap.get(msg.players[i].hitBoard[j][k]);
          }
        }
      }
    },
    async initGameEvent() {
      let response = await fetch("https://demo.httprelay.io/mcast/IntOverflowBS" + window.serverCode);
      let resp = await response.text();
      console.log(resp);
      let msg = JSON.parse(resp);
      let ans = true;
      for (let i = 0; i < msg.players.length; i++) {
        if (msg.players[i].state !== "R" && msg.players[i].state !== "P") {
          ans = false;
        }
        if (i >= this.playerCount) {
          this.playerCount++;
          this.boards.push(msg.players[i]);
          await this.$nextTick();
          this.$refs.boards[i].inGameName = msg.players[i].inGameName;
        }
      }
      if (ans) {
        this.player.state = "P";
        this.readyMsg = "Playing...";
        await this.update();
        return;
      }
      setTimeout(this.initGameEvent, 1000);
    }
  }
}
</script>

<style scoped>
</style>
