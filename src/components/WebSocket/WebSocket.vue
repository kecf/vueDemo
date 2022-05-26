<template>
  <button @click="sendMessageToServer">给后台发送信息</button>
</template>

<script>
export default {
  name: "WebSocket",
  data() {
    return {
      wsIsRun: false,
      webSocket: null,
      ws: '',
      wsTimer: null,
    }
  },
  async mounted() {
    this.wsIsRun = true
    this.wsInit()
  },
  methods: {
    sendMessageToServer() {
      if (this.webSocket.readyState === 1) {
        this.webSocket.send('connection OK')
      } else {
        throw Error('connection error')
      }
    },
    wsInit() {
      this.ws = 'ws://localhost:8080/websocket/kcf'
      if (!this.wsIsRun) return
      this.wsDestroy()
      this.webSocket = new WebSocket(this.ws)
      this.webSocket.addEventListener('open', this.wsOpenHandler)
      this.webSocket.addEventListener('message', this.wsMessageHandler)
      this.webSocket.addEventListener('close', this.wsCloseHanler)
    },
    wsOpenHandler() {
      console.log('ws建立连接成功')
    },
    wsMessageHandler(e) {
      console.log('wsMessageHandler')
      console.log(e)
    },
    wsCloseHanler(event) {
      console.log(event, 'ws关闭')
      this.wsInit()
    },
    wsDestroy() {
      if (this.webSocket !== null) {
        this.webSocket.removeEventListener('open', this.wsOpenHandler)
        this.webSocket.removeEventListener('message', this.wsMessageHandler)
        this.webSocket.removeEventListener('close', this.wsCloseHanler)
        this.webSocket.close()
        this.webSocket = null
      }
    },

  }
}
</script>

<style scoped>

</style>