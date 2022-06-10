<template>
  <div>
    <p>{{ wsMessage }}</p>
    <button @click="sendMessageToServer">给后台发送信息</button>
    <table>
      <caption style="font-size: 24px">上一交易日信号</caption>
      <thead>
        <tr>
          <th style="width: 150px">signal name</th>
          <th style="width: 150px">event ID</th>
          <th style="width: 200px">event time</th>
          <th style="width: 150px">event source</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="jsonObject in this.JsonsPrevDay" :key="jsonObject.eventId">
          <td style="text-align: center" v-text="jsonObject.signalName"></td>
          <td style="text-align: center" v-text="jsonObject.eventId"></td>
          <td style="text-align: center" v-text="jsonObject.eventTime"></td>
          <td style="text-align: center" v-text="jsonObject.eventSource"></td>
        </tr>
      </tbody>
    </table>
    <table>
      <caption style="font-size: 24px">本交易日信号</caption>
      <thead>
      <tr>
        <th style="width: 150px">signal name</th>
        <th style="width: 150px">event ID</th>
        <th style="width: 200px">event time</th>
        <th style="width: 150px">event source</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="jsonObject in this.JsonsToday" :key="jsonObject.eventId">
        <td style="text-align: center" v-text="jsonObject.signalName"></td>
        <td style="text-align: center" v-text="jsonObject.eventId"></td>
        <td style="text-align: center" v-text="jsonObject.eventTime"></td>
        <td style="text-align: center" v-text="jsonObject.eventSource"></td>
      </tr>
      </tbody>
    </table>
    <table>
      <caption style="font-size: 24px">异常信号</caption>
      <thead>
      <tr>
        <th style="width: 150px">signal name</th>
        <th style="width: 150px">event source</th>
        <th style="width: 150px">event type</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="[key, value] in this.JsonsError" :key="key">
        <td style="text-align: center; color: red" v-text="value.signalName"></td>
        <td style="text-align: center; color: red" v-text="value.eventSource"></td>
        <td style="text-align: center; color: red" v-text="value.type"></td>
      </tr>
      </tbody>
    </table>
  </div>
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
      wsMessage: '此处为接收到的后端信息',
      JsonsPrevDay: null,
      JsonsToday: null,
      JsonsError: null,
      colorMap: null
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
      this.JsonsPrevDay = []
      this.JsonsToday = []
      this.JsonsError = new Map();
      this.colorMap = new Map();
      this.colorMap.set('error', 'red')
      this.colorMap.set('late', 'blue')
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
      try {
        let wsJson = JSON.parse(e.data)
        switch (wsJson["type"]) {
          case "prevDay":
            this.JsonsPrevDay.push(wsJson)
            break
          case "today":
            this.JsonsToday.push(wsJson)
            break
          case "error":
            this.JsonsError.set(wsJson["signalName"], wsJson)
              console.log(wsJson["signalName"])
            break
          case "late":
            this.JsonsError.set(wsJson["signalName"], wsJson)
            break
          case "normal":
            if (this.JsonsError.has(wsJson["signalName"])) {
              this.JsonsError.delete(wsJson["signalName"])
            }
            break
        }
        // this.$forceUpdate()
      } catch (exception) {
        this.wsMessage = e.data
        console.log("接收到后端信息：" + e.data)
      }
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