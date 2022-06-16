<template>
  <div>
    <p>{{ wsMessage }}</p>
    <p>{{ heartbeatTime }}</p>
    <button @click="sendMessageToServer">给后台发送信息</button>
    <table>
      <caption style="font-size: 24px">疑似仓位</caption>
      <thead>
      <tr>
        <th style="width: 150px">position name</th>
        <th style="width: 150px">event id</th>
        <th style="width: 200px">event time</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="[key, value] in this.JsonsSuspected" :key="key">
        <td style="text-align: center; color: darkslategrey" >{{ value.positionName }}</td>
        <td style="text-align: center; color: darkslategrey" >{{ value.eventId }}</td>
        <td style="text-align: center; color: darkslategrey" >{{ value.eventTime }}</td>
      </tr>
      </tbody>
    </table>
    <table>
      <caption style="font-size: 24px">warning仓位</caption>
      <thead>
      <tr>
        <th style="width: 150px">position name</th>
        <th style="width: 150px">event id</th>
        <th style="width: 200px">event time</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="[key, value] in this.JsonsWarning" :key="key">
        <td style="text-align: center; color: orange" >{{ value.positionName }}</td>
        <td style="text-align: center; color: orange" >{{ value.eventId }}</td>
        <td style="text-align: center; color: orange" >{{ value.eventTime }}</td>
      </tr>
      </tbody>
    </table>
    <table>
      <caption style="font-size: 24px">error仓位</caption>
      <thead>
      <tr>
        <th style="width: 150px">position name</th>
        <th style="width: 150px">event id</th>
        <th style="width: 200px">event time</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="[key, value] in this.JsonsError" :key="key">
        <td style="text-align: center; color: red" >{{ value.positionName }}</td>
        <td style="text-align: center; color: red" >{{ value.eventId }}</td>
        <td style="text-align: center; color: red" >{{ value.eventTime }}</td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: "PositionAlarm",
  data() {
    return {
      wsIsRun: false,
      webSocket: null,
      ws: '',
      wsTimer: null,
      wsMessage: '此处为接收到的后端信息',
      heartbeatTime:'此处为心跳信号时间',
      JsonsSuspected:null,
      JsonsWarning:null,
      JsonsError:null,
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
      this.JsonsSuspected = new Map()
      this.JsonsWarning = new Map()
      this.JsonsError = new Map()
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
          case "heartbeat":
            this.heartbeatTime = wsJson["eventTime"]
            break
          case "suspected":
            this.JsonsSuspected.set(wsJson["positionName"], wsJson)
            break
          case "warning":
            this.JsonsWarning.set(wsJson["positionName"], wsJson)
            break
          case "error":
            this.JsonsError.set(wsJson["positionName"], wsJson)
            break
          case "normal":
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