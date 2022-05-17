<template>
  <div class="rightSidebar">
    <div v-if="this.state1 !== 'BSChatBox'">
      Room number:
      <input ref="serverCode" @change="updateCode"/>
      <br>
      In game name:
      <input ref="IGN" @change="updateIGN"/>
    </div>
    <component :is="state1">Enter the room number and your in game name first ya dingus</component>
  </div>
  <div class="main">
    <component :is="state2"></component>
    <button v-if="this.state1 !== 'BSChatBox'" @click="this.state1 = 'BSChatBox'; this.state2 = 'BSGame'">I'm done with
      entering the server code and my IGN
    </button>
  </div>
  <div class="header">
    <img id="badIcon" src="./assets/android-chrome-192x192.png"/>
    <div class="headerBlock">BattleShip (BS for short)</div>
    <div class="headerBlock"> These links don't work</div>
    <div class="headerBlock"> lol</div>
    <div class="headerBlock"> No links for you!</div>
  </div>
</template>

<script>
import BSChatBox from "./components/BSChatbox";
import BSGame from "@/components/BSGame";

window.serverCode = "";
window.inGameName = "";
export default {
  name: 'App',
  components: {
    BSChatBox,
    BSGame
  },
  created() {
    this.state1 = "div";
    this.state2 = "div";
    this.readyMsg = "Not Ready";
  },
  data() {
    return {
      state1: this.state1,
      state2: this.state2,
      readyMsg: this.readyMsg
    }
  },
  methods: {
    updateCode() {
      window.serverCode = this.$refs.serverCode.value;
    },
    updateIGN() {
      window.inGameName = this.$refs.IGN.value;
    }
  }
}
</script>

<style>
body {
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: grid;
  grid-template-areas:
      "header header"
      "main rightSidebar";
  grid-template-rows: calc(90px + 1rem) auto;
  grid-template-columns: min(80%, 800px) auto;
  grid-gap: 0;
  top: 0;
  left: 0;
}

.main {
  grid-area: main;
  padding-left: 20px;
  padding-right: 20px;
}

.rightSidebar {
  grid-area: rightSidebar;
  padding-right: 20px;
  padding-left: 20px;
  border-left: 1px solid midnightblue;
}

.header {
  text-align: left;
  grid-area: header;
  background: midnightblue;
  color: ivory;
  position: fixed;
  width: 100%;
  display: flex;
  flex-direction: row;
}

.headerBlock {
  border-left: 1px solid lightblue;
  padding-left: 10px;
  margin-left: 10px;
  padding-top: 20px;
  padding-bottom: 20px;
  margin-top: 10px;
  margin-bottom: 10px;
}

#badIcon {
  margin-left: 20px;
  margin-right: 20px;
  margin-top: 10px;
  width: calc(1rem + 40px);
  height: calc(1rem + 40px);
}
</style>
