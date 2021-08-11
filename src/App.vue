<style lang="scss">
body{
  width: 100%;
  height: 100%;
}
#app{
  width:100%;
  height: 100%;
  position: absolute;
}
*{
  margin: 0;
  padding: 0;
}
</style>

<style scoped lang="scss">
$sep-w : 2px;

#chess-clock{
  width: 98%;
  height: 100%;
  margin: auto 0;

  #clocks{
    position: relative;
    margin-top: 10%;
    height: 50%;
    width: 100%;
    #clock1, #clock2 {
      width: 49%;
      display: inline-block;
      box-sizing: border-box;
      position: absolute
    }
    #clock1{
      left: 0;
    }
    #clock2{
      right: 0;
    }
    #clock-sep{
      position: absolute;
      text-align: center;
      height: 100%;
      width: 1px;
      left: 50%;
      margin-left: -$sep-w/2;
      border-left: $sep-w solid grey;
    }
  }
}

#chess-clock-mobile{
  width: 98%;
  height: 100%;
  margin: auto 0;

  .reversed {
    transform: rotate(180deg);
    transform-origin: center;
  }

  .clock {
    width: 100%;
    height: 44%;
    display: inline-block;
    position: relative;
    &>*{
      width:100%;
      height:50%;
      display: inline-block;
      box-sizing: border-box;
      position: relative;
      &.small{
        transform: scale(.8);
        //transform-origin: center;
      }
    }
  }
  #clock-sep{
    position: relative;
    text-align: center;
    height: 12%;
    width: 100%;
    vertical-align : middle;
    &.pause{
      background-color:#a1b7ff
    }
    div{
      display: table;
      width: 100%;
      height: 100%;
      span{
        display: table-cell;
        width: 50%;
        //height: 100%;
        font-size: 50px;
        text-align: center;
        vertical-align: middle;

      }
    }
  }
  

}
</style>

<template>
  <div v-if="showSettings">
    <label for="time">Temps initial</label><input id="time" type="number" v-model="initTime_"/><br/>
    <label for="inc">Incrément</label><input id="inc" type="number" v-model="incTime_"/><br/>
    <label for="max">Temps Max</label><input id="max" type="number" v-model="maxTime_"/><br/>
    <input type="button" value="ok" @click="showSettings=false"/>
  </div>
  
  <div v-if="!showSettings && desktop" id="chess-clock">
    <div class="buttons">
      <input type="button" value="settings" @click="showSettings=true"/>
      <input type="button" value="reset" @click="resetAction"/>
    </div>
    <div id="clocks">
      <Clock id="clock1" :running="c1Running" @flag="flag(1)" v-model:time="time1" v-model:flag="flag1"></Clock>
      <div id="clock-sep"></div>
      <Clock id="clock2" :running="c2Running" @flag="flag(2)" v-model:time="time2" v-model:flag="flag2"></Clock>
    </div>
  </div>
  
  <div v-if="!showSettings && !desktop" id="chess-clock-mobile">
    <div class="clock reversed"  @click="() => updateMobile(1)">
      <Clock :running="false" :time="time2" :flag="flag2" :passiveRunning="c2Running" class="small"></Clock>
      <Clock :running="c1Running" v-model:time="time1" v-model:flag="flag1"></Clock>
    </div>
    <div id="clock-sep" class="pause" v-if="c1Running || c2Running" @click="() => updateMobile(null)">
      <div>
        <span>Pause</span>
        <span class="reversed">Pause</span>
      </div>
    </div>
    <div id="clock-sep" v-else>
      <input type="button" value="settings" @click="showSettings=true"/>
      <input type="button" value="reset" @click="resetAction"/>
    </div>
    <div class="clock"  @click="() => updateMobile(2)">
      <Clock :running="false" :time="time1" :flag="flag1" :passiveRunning="c1Running" class="small"></Clock>
      <Clock :running="c2Running" v-model:time="time2" v-model:flag="flag2"></Clock>
    </div>
  </div>
</template>

<script>

import { ref, onMounted, watch, watchEffect, toRefs, computed } from 'vue'
import Clock from './components/Clock.vue'
import MobileDetect from 'mobile-detect'

export default {
  name : 'App',
  components : {
    Clock
  },
  setup(props) {
    const c1Running = ref(false)
    const c2Running = ref(false)

    const initTime_ = ref(localStorage.getItem('initTime') || 600)
    const initTime = computed( () => initTime_.value * 1000 )
    const incTime_ = ref(localStorage.getItem('incTime') || 0)
    const incTime = computed( () => incTime_.value * 1000 )
    const maxTime_ = ref(localStorage.getItem('maxTime') || 600)
    const maxTime = computed( () => maxTime_.value * 1000 )

    const time1 = ref(initTime.value)
    const time2 = ref(initTime.value)

    const flag1 = ref(false)
    const flag2 = ref(false)
    const flag = computed( () => flag1.value ? 1 : flag2.value ? 2 : 0 )

    let currentPlayer = 0

    function update(player) {
      if(!player) {
        c1Running.value = false
        c2Running.value = false
        currentPlayer = player
        return
      }
      console.log({
        'currentPlayer' : currentPlayer,
      });
      if(player == 1) {
        if(currentPlayer == 1) {
          return
        }
        c1Running.value = false
        c2Running.value = true
        if(currentPlayer != 0) {
          time1.value += incTime.value
          if(time1.value > maxTime.value) {
            time1.value = maxTime.value
          }
          
        }
        currentPlayer = 1
        return
      }
      if(player == 2) {
        if(currentPlayer == 2) {
          return
        }
        c1Running.value = true
        c2Running.value = false
        if(currentPlayer != 0) {
          time2.value += incTime.value
          if(time2.value > maxTime.value) {
            time2.value = maxTime.value
          }
        }
        currentPlayer = 2
        return
      }
    }

    function updateMobile(player) {
      if(flag.value && player){
        return
      }
      update(player)
    }

    function reset(){
      update(0)
      time1.value = initTime.value
      time2.value = initTime.value
      flag1.value = false
      flag2.value = false
    }

    watchEffect( () => {
      reset()
      const _1 = incTime.value
      const _2 = maxTime.value
      localStorage.setItem('initTime', initTime_.value)
      localStorage.setItem('incTime', incTime_.value)
      localStorage.setItem('maxTime', maxTime_.value)
    })

    const md = new MobileDetect(window.navigator.userAgent)
    
    const r = {
      c1Running,
      c2Running,
      time1,
      time2,
      flag1,
      flag2,
      initTime_,
      initTime,
      incTime_,
      incTime,
      maxTime_,
      maxTime,
      
      showSettings : ref(true),
      resetAction(){
        reset()
        document.activeElement.blur()
      },

      desktop : ref(md.mobile() === null),

      updateMobile,
    }



    
    onMounted( () => {
      addEventListener('keyup', (ev) => {
        if(r.showSettings.value) {
          ev.stopPropagation()
          return;
        }
        
        console.log({msg: 'Key event', ev});
        if(flag.value) {
          ev.stopPropagation()
          return 
        }
        
        if(ev.code == 'ControlLeft') {
          update(1)
          ev.stopPropagation()
          return
        }
        else if(ev.code == 'Space') {
          update(null)
          ev.stopPropagation()
          return
        }
        else if(ev.code == 'NumpadEnter'){
          update(2)
          ev.stopPropagation()
          return
        }
      })
    })

    console.log({
      'r.desktop.value' : r.desktop.value,
    });
    

    return r
  }
}
</script>

