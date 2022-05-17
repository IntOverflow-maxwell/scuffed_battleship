<template>
  <div class="BSChatBox">
    <br>
    <div>Send Message</div>
    <br>
    <div id="Chat" ref="Chat">
      {{ this.chatText }}
    </div>
    <br>
    <textarea id="ChatInput" ref="ChatInput" placeholder="Say something..."></textarea>
    <br>
    <button id="SendButton" class="button" @click="send(getMsg())">
      Send
    </button>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  name: 'BSChatBox',
  created() {
    this.finished = undefined;
    this.chatLog = [];
    this.chatText = "";
    this.lastData = "0";
    this.send(window.inGameName + " has joined the room!");
    this.update();
  },
  data() {
    return {
      finished: this.finished,
      chatLog: this.chatLog,
      chatText: this.chatText,
      lastData: this.lastData
    }
  },
  methods: {
    getMsg() {
      let msg = window.inGameName + ': ' + this.$refs.ChatInput.value;
      if (this.$refs.ChatInput.value === "") {
        msg = "";
      }
      return msg;
    },
    async send(msg) {
      if (msg === "") {
        return;
      }
      const response = await fetch("https://demo.httprelay.io/mcast/IntOverflowBSChat" + window.serverCode);
      const rdata = await response.text();
      let split = rdata.split("\n");
      let number = split.shift();
      if (split.length > 5) {
        split.shift();
        number++;
      }
      let ndata = number + "\n" + split.join("\n") + "\n" + btoa(msg);
      if (split.length === 0) {
        ndata = number + "\n" + btoa(msg);
      }
      let requestOptions = {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: ndata
      };
      await fetch("https://demo.httprelay.io/mcast/IntOverflowBSChat" + window.serverCode, requestOptions);
      this.$refs.ChatInput.value = "";
    },
    async update() {
      let response = fetch("https://demo.httprelay.io/mcast/IntOverflowBSChat" + window.serverCode);
      const p2 = new Promise((_r, rej) => setTimeout(() => rej("p2"), 20000));
      let resp;
      try {
        resp = await Promise.race([response, p2]);
      } catch (e) {
        let requestOptions = {
          method: "POST",
          headers: {"Content-Type": "application/json"},
          body: this.lastData
        };
        await fetch("https://demo.httprelay.io/mcast/IntOverflowBSChat" + window.serverCode, requestOptions);
        setTimeout(this.update, 10);
        return;
      }
      let rdata = await (await resp).text();
      this.lastData = rdata;
      let split = rdata.split("\n");
      let number = parseInt(split.shift());
      if (typeof this.finished === "undefined") {
        this.finished = number;
      }
      for (let i = 0; i < split.length; i++) {
        if (i + number >= this.finished) {
          this.chatLog.push(atob(split[i]));
          this.finished++;
        }
      }
      this.chatText = this.chatLog.join("\n");
      setTimeout(this.update, 1000);
    }
  }
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.BSChatBox {
  grid-area: sidebar;
}

#Chat {
  overflow-y: scroll;
  width: auto;
  height: 10rem;
  border: 1px solid black;
  text-align: left;
  white-space: pre;
}
</style>
