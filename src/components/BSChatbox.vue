<template>
  <div class="BSChatBox">
    <br>
    <div>Send Message</div>
    <br>
    <input id="ChatName" ref="ChatName">
    <br>
    <input id="ChatInput" ref="ChatInput">
    <br>
    <button id="SendButton" class="button" @click="send(this.$refs.ChatName.value + ': ' + this.$refs.ChatInput.value)">
      Send
    </button>
    <br>
    <div id="Chat" ref="Chat">
      {{ this.chatText }}
    </div>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  name: 'BSChatBox',
  created() {
    this.finished = 0;
    this.chatLog = [];
    this.chatText = "";
    this.send("User has joined the room!");
    this.update();
    console.log(this);
  },
  data() {
    return {
      finished: this.finished,
      chatLog: this.chatLog,
      chatText: this.chatText
    }
  },
  methods: {
    async send(msg) {
      const response = await fetch("https://demo.httprelay.io/mcast/IntOverflowBSChat");
      const rdata = await response.text();
      let split = rdata.split("\n");
      let number = split.shift();
      if (split.length > 5) {
        split.shift();
        number++;
      }
      let ndata = number + "\n" + split.join("\n") + "\n" + msg;
      let requestOptions = {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: ndata
      };
      await fetch("https://demo.httprelay.io/mcast/IntOverflowBSChat", requestOptions);
      this.$refs.ChatInput.value = "";
    },
    async update() {
      let response = fetch("https://demo.httprelay.io/mcast/IntOverflowBSChat");
      const p2 = new Promise((_r, rej) => setTimeout(() => rej("p2"), 30000));
      let resp;
      try {
        resp = await Promise.race([response, p2]);
      } catch (e) {
        let requestOptions = {
          method: "POST",
          headers: {"Content-Type": "application/json"},
          body: "0"
        };
        await fetch("https://demo.httprelay.io/mcast/IntOverflowBSChat", requestOptions);
        setTimeout(this.update, 10);
        return;
      }
      let rdata = await (await resp).text();
      console.log(rdata);
      let split = rdata.split("\n");
      let number = split.shift();
      for (let i = 0; i < split.length; i++) {
        if (i + number > this.finished) {
          this.chatLog.push(split[i]);
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
</style>
