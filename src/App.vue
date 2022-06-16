<template>
  <div>
<!--    <WebSocket></WebSocket>-->
    <PositionAlarm></PositionAlarm>
  </div>
</template>

<script>
import PositionAlarm from './components/PositionAlarm/PositionAlarm'
export default {
  name: "index",
  components: {
    PositionAlarm
  },
}
</script>
