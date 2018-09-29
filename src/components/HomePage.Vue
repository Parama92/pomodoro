<template>
    <b-container class="container">
        <b-row class="mb-4">
            <b-col>
                <img :class="{ rotateIt : animate}" id="tomato" src="../assets/tomato.gif" width="30%">
            </b-col>
        </b-row>
        <b-row class="justify-content-md-center mb-4">
            <b-col md="3">
                <span class="pomodoro">Pomodoro #{{ pomodoro }}</span>
                <div id="clock" v-if="showClock" :class="{ blink : blinkIt }">{{ timer }}</div>
            </b-col>
        </b-row>
        <b-row>
            <b-col><TimeSetter v-bind:start="running.short" @started="startTimer" @pause="pauseClock" @stopped="stopClock" button="Short"></TimeSetter></b-col>
            <b-col><TimeSetter v-bind:start="running.pomodoro" @started="startTimer" @pause="pauseClock" @stopped="stopClock" button="Pomodoro"></TimeSetter></b-col>
            <b-col><TimeSetter v-bind:start="running.long" @started="startTimer" @pause="pauseClock" @stopped="stopClock" button="Long"></TimeSetter></b-col>
        </b-row>
        <!-- Before encapsulating this behaviour in the component -->
        <!-- <b-row>
            <b-col class="text-center">
                <b-button v-show="brk.short" @click="startBreak">Start short break</b-button>
                <b-button v-show="running.short.status">Skip short break</b-button>
            </b-col>
            <b-col class="text-center">
                <b-button @click="startPomodoro" v-if="!running.pomodoro.status">Start Pomodoro</b-button>
                <b-button @click="pausePomodoro" v-else>Pause Pomodoro</b-button>
                <b-button @click="stopPomodoro" v-show="clockStatus">Stop Pomodoro</b-button>
            </b-col>
            <b-col class="text-center">
                <b-button v-show="brk.long" @click="startBreak">Start long break</b-button>
                <b-button v-show="running.long.status">Skip long break</b-button>
            </b-col>
        </b-row> -->
    </b-container>

</template>

<script>
import TimeSetter from './TimeSetter.vue'
/* eslint-disable */

export default {
  name: 'HomePage',
  components: {
    TimeSetter
  },
  data () {
    return {
      pomodoro: 1,
      showClock: false,
      clock: 25,
      timer: '',
      timerId: 0,
      animate: false,
      buttonType: '',
      running: {
        pomodoro: {
          show: true,
          status: false,
          val: 25
        },
        short: {
          show: false,
          status: false,
          val: 5
        },
        long: {
          show: false,
          status: false,
          val: 30
        }
      }
    }
  },
  computed: {
    //   clockStatus(){
    //     return this.showClock && this.timer.trim() !== '00 : 00'
    //   }
    blinkIt () { // if the timer counts down below '00 : 10', the timer starts blinking.
      let [min, sec] = this.timer.split(':')
      return (parseInt(min) === 0 && parseInt(sec) <= 10 && parseInt(sec) !== 0)
    }
  },
  methods: {
    startClock () {
      this.animate = false
      if (!this.running[this.buttonType].status) { // checks if clock is running
        if (!['', '00 : 00'].includes(this.timer.trim())) { // checks if timer has ended or yet to begin. Enters block if check returns false
          let timer = this.timer.split(':') // splits timer on ':' to form an array of [min, sec]
          this.running[this.buttonType].status = true // the running status of the clock is changed to true
          this.countDown(parseInt(timer[0].trim()), parseInt(timer[1].trim())) // the countDown timer is started with updated minute and second values
        } else { // if timer has ended or yet to begin
          for (let [state, val] of Object.entries(this.running)) {
            val.show = state === 'pomodoro' ? true : state === this.buttonType // set all running status flags to false
          }
          this.showClock = true
          this.runCountDown() // start countDown
        }
      }
    },
    pauseClock () {
      if (this.running[this.buttonType].status) {
        clearTimeout(this.timerId)
        this.running[this.buttonType].status = false
      }
    },
    stopClock (buttonType) {
      buttonType = buttonType.toLowerCase()
      clearTimeout(this.timerId) // clear the timeout function so that that the timer doesn't get renewed
      this.running[buttonType].status = false // set the running flag to false
      this.showClock = false
      this.timer = ''
    },
    startTimer (data) {
      let buttonType = data.buttonType.toLowerCase() // define a local variable to store the buttonType obtained from the child component
      if (this.running[buttonType] === undefined) { // if the buttonType is not recognised throw an error
        throw new Error(`Button type: '${data.buttonType}' not recognised`)
      } else {
        this.buttonType = buttonType // update global variable
        this.running[buttonType].val = this.clock = data.time // update the clock and timer settings
        this.startClock() // start the clock
      }
    },
    runCountDown () {
      if (this.running[this.buttonType] !== undefined) { // checks if the buttonType exists in the 'running' object
        if (!this.running[this.buttonType].status) { // checks if the timer for this buttonType is running
          this.running[this.buttonType].status = true // sets the timer status to true
          this.countDown() // starts countDown
        }
      }
    },
    countDown (min = this.clock, sec = 0) {
      // this.running.pomodoro.status = true
      if (min > 0 || sec > 0) { // enters only if timer is not '00 : 00'
        if (sec === 0) { // if sec is 0 reduce min by one and set sec to 59
          min--
          sec = 59
        } else {
          sec--
        }
        this.timer = `${this.format(min)} : ${this.format(sec)}` // update the value of timer
        this.timerId = setTimeout(() => {
          this.countDown(min, sec) // call this function recursively after every second
        }, 1000)
      } else { // when timer = '00 : 00'
        // this.running.pomodoro.status = false
        // this.updatePomodoro();
        this.running[this.buttonType].status = false // set the running status to false
        if (this.buttonType === 'pomodoro') { this.updatePomodoro() } // update other features if the pomodoro counter ends
      }
    },
    format (time) {
      return time < 10 ? '0' + time : time
    },
    updatePomodoro () {
      this.animate = true
      if (this.pomodoro === 4) { // if at 4th pomodoro start long break else start short break
        this.running.long.show = true
      } else {
        this.running.short.show = true
        this.pomodoro++
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .container{
        position: relative;
    }
    .rotateIt{
        animation-name: rotate;
        animation-duration: 1.5s;
        animation-iteration-count: infinite;
    }
    @keyframes rotate {
        0% {transform: rotate(20deg);}
        25% {transform: rotate(0deg);}
        50% {transform: rotate(-20deg);}
        75% {transform: rotate(0deg);}
        100% {transform: rotate(20deg);}
    }
    .pomodoro{
        font-size:0.7em;
        color:#2c3e50;
    }
    .blink{
        animation-name: blink;
        animation-duration: 1s;
        animation-iteration-count: infinite;
    }
    @keyframes blink{
        0% {color: #2c3e5041; background-color: #2c3e50;}
        50% {color: #2c3e50; background-color: #2c3e5041;}
        100% {color: #2c3e5041; background-color: #2c3e50;}
    }
    #clock{
        color:#2c3e50;
        font-size: 3.2em;
        background-color: #2c3e5041;
        box-shadow: #2c3e50 2px 2px 5px;
    }
</style>
