<template>
  <div class="BSChatBox">
    <br>
    <div>Send Message</div>
    <br>
    <input id="ChatName" ref="ChatName">
    <br>
    <input id="ChatInput" ref="ChatInput">
    <br>
    <div id="Chat" ref="Chat">
      {{ chatText }}
    </div>
    <br>
    <button id="SendButton" class="button" @click="send(this.$refs.ChatName.value + ': ' + this.$refs.ChatInput.value)">
      Send
    </button>
  </div>
</template>

<script>
/* eslint-disable */
let chatLog = [];
let offset = 0;
export default {
  name: 'BSChatBox',
  data() {
    return {
      chatLogs: chatLog,
      msgOffset: offset
    }
  },
  chatText: "",
  created() {
    this.send("User has joined the room!");
    this.update();
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
    },
    async update() {
      const response = await fetch("https://demo.httprelay.io/mcast/IntOverflowBSChat");
      const rdata = await response.text();
      let split = rdata.split("\n");
      let number = split.shift();
      if (!chatLog[0]) offset = -number;
      for (let i = 0; i < split.length; i++) {
        chatLog[number + i + offset] = split[i];
      }
      this.chatText = chatLog.join("\n");
      console.log(chatLog.join("\n"));
      setTimeout(this.update, 500);
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
