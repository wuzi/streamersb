<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
  export default {
    name: 'streamer-sb'
  }
</script>

<style>
  #app {
    font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
  }

  body {
    overflow-y: hidden;
  }

  ::-webkit-scrollbar-track
  {
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
    box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
    background-color: #F5F5F5;
  }

  ::-webkit-scrollbar
  {
    width: 5px;
    background-color: #F5F5F5;
  }

  ::-webkit-scrollbar-thumb
  {
    background-color: #f96332;
  }
</style>
