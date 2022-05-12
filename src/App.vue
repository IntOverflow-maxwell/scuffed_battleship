<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <BSChatBox/>
</template>

<script>
import BSChatBox from "./components/BSChatbox";

export default {
  name: 'App',
  components: {
    BSChatBox
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
