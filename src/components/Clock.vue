
<style scoped lang="scss">

$box-w : 450px;
$val-w : 80px;
$sval-w : 50px;
$sval-mw : 10px;
$sep-w : 30px;

$val-fh: 80px;
$sval-fh: $val-fh * .5;
$sep-fh : $val-fh;

div.clock{
  //background-color: red;
  .box{
    //background-color: yellow;
    &.running{
      background-color: #DDDDDD;
    }
    display: block;
    text-align: center;
    height: 150px;
    width: $box-w;
    transform-origin: top left;
    &.flag{
      background-color: red;
    }
    .indicator {
      padding: -5px;
      position: absolute;
      display: block;
      top: 0;
      left: 0;
      box-sizing: border-box;
      border: 5px solid #363636;
      width: 100%;
      height: 100%;
    }

    >*{
      display: inline-block;
    }
    .hours{
      width: $val-w;
      text-align: right;
    }
    .h{
      width: $sep-w + 20px;
    }
    .minutes{
      width: $val-w;
      text-align: right;
    }
    .m{
      width: $sep-w;
    }
    .seconds{
      width: $val-w;
      text-align: right;
    }
    .s{
      width: $sep-w;
    }
    .decimals{
      width: sval-w;
      text-align: left;
      margin-left: $sval-mw;
    }
    .val{
      font-size: $val-fh;
    }
    .sval{
      font-size: $sval-fh;
    }
    .sep{
      font-size: $sep-fh;
    }
  }
}

</style>

<template>
  <div ref="clock" class="clock">
    <div ref="box" class="box" :class="{
      running : passiveRunning === null ? running : passiveRunning,
      flag : flag
    }">
      <div v-if="hours > 0" class="hours val">{{hours}}</div>
      <div v-if="hours > 0" class="h sep">h</div>
      <div class="minutes val">{{minutes}}</div>
      <div class="m sep">:</div>
      <div class="seconds val">{{seconds}}</div>
      <div class="s sep">"</div>
      <div v-if="hours == 0 && minutes == 0 && seconds < 20" class="decimals sval">{{decimals}}</div>
      <div class="indicator" v-if="passiveRunning === null ? running : passiveRunning"></div>
    </div>
  </div>
  
</template>

<script>

import { ref, onMounted, onBeforeUnmount, watch, watchEffect, toRefs, computed } from 'vue'

const boxBaseW = 450

export default {
  name : 'Clock',
  props : {
    running: Boolean,
    small: {
      type: Boolean,
      default: false,
    },
    passiveRunning: {
      type: Boolean,
      default: null,
    },
    time: Number,
    flag: Boolean,
  },
  emits : ['update:flag', 'update:time'],
  setup(props, {emit}) {
    const hours = computed( () => Math.floor(props.time / 3600000))
    const minutes = computed( () => Math.floor(props.time / 60000) % 60)
    const seconds = computed( () => Math.floor(props.time / 1000) % 60)
    const decimals = computed( () => Math.floor(props.time / 100) % 10)

    let wasRunning = false
    let cancelId = null
    
    let startTimestamp = null
    
    function update(){
      const now = Date.now()
      let new_val = props.time - (Date.now() - startTimestamp)
      if(new_val < 0) {
        emit('update:flag', true)
        new_val = 0
      }
      startTimestamp = now
      emit('update:time', new_val)
    }

    watchEffect( () => {
      const now = Date.now()
      if(props.running) {
        if(wasRunning) {
          return;
        }
        startTimestamp = now
        cancelId = setInterval( () => update(), 100 )
        wasRunning = true
      }
      else {
        if(! wasRunning) {
          return;
        }
        clearInterval(cancelId)
        wasRunning = false
      }
    })
    
    const clock = ref(null)
    const box = ref(null)
    const boxSizeObserver = new ResizeObserver( () => {
      const rect = clock.value.getBoundingClientRect()
      const k = rect.width / boxBaseW
      box.value.style.transform = "scale("+k+")"
      
    })
    onMounted( () => {
      boxSizeObserver.observe(clock.value)
    })
    onBeforeUnmount( () => {
      boxSizeObserver.disconnect()
    })
    
    const r = {
      hours,
      minutes,
      seconds,
      decimals,

      box,
      clock,
    }
    return r
  }
}
</script>

