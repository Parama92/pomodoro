<template>
    <b-container>
        <b-row>
            <b-col class="text-center"><font-awesome-icon @click="plus" :icon="plusIcon" class="pointer" /></b-col>
        </b-row>
        <b-row>
            <b-col class="text-center">{{ time }}</b-col>
        </b-row>
        <b-row>
            <b-col class="text-center"><font-awesome-icon @click="minus" :icon="minusIcon" class="pointer" /></b-col>
        </b-row>
        <b-row>
            <b-col class="text-center">
                <b-button v-show="start.show && notSkipped" @click="startRunning">{{ startOrPause }} {{ buttonType }}</b-button>
                <b-button v-show="running || pause" @click="stopRunning">{{ skipOrStop }} {{ buttonType }}</b-button>
            </b-col>
        </b-row>
    </b-container>
    <!-- <span @click="minus">
        <font-awesome-icon :icon="minusIcon" />
    </span>
        &nbsp;{{time}}&nbsp;
    <span @click="plus">
        <font-awesome-icon :icon="plusIcon" />
    </span> -->
</template>

<script>
import FontAwesomeIcon from '@fortawesome/vue-fontawesome'
import { faSortUp, faSortDown } from '@fortawesome/fontawesome-free-solid'

export default {
  name: 'TimeSetter',
  components: {
    FontAwesomeIcon
  },
  props: ['start','button'],
  computed: {
    skipOrStop(){ // Skip applies to breaks whereas stop applies to pomodoros
        return this.button.toLowerCase() == 'pomodoro'? 'Stop' : 'Skip'
    },
    startOrPause(){
        return  this.running? 'Pause' : 'Start'
    },
    buttonType(){ // formats the button 
        return this.button + (this.button.toLowerCase() == 'pomodoro'? '' : ' Break')
    },
    plusIcon () {
        return faSortUp
    },
    minusIcon () {
        return faSortDown
    },
    running() { // keeps track of the running status of the clock based on the 'start' value
        return this.start.status
    }
  },
  data () {
    return {
      time: this.start.val,
      // running: this.start.status,
      pause: false,
      notSkipped: true
    }
  },
  methods: {
    minus () { // method to decrease the counter value
        if (this.time === 1) {
            this.time = 60
        } else {
            this.time--
        }
    },
    plus () { // method to increase the counter value
        if (this.time === 60) {
            this.time = 1
        } else {
            this.time++
        }
    },
    startRunning () {
        if (this.running) { // if the clock is in running state, then must pause the clock
            this.$emit('pause')
            this.pause = true
        } else { // if the clock is not in running state, then must start the clock
            this.pause = false
            this.$emit('started', { time:this.time, buttonType: this.button })
        }
    },
    stopRunning () { // stop the clock
        this.pause = false
        this.$emit('stopped', this.button)
        this.notSkipped = this.button.toLowerCase() == 'pomodoro' // if it is one of the breaks, make sure that the start button is not visible anymore when the timer is skipped.
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .pointer {
        cursor:pointer;
    }
    .text-center{
        font-size: 3em;
    }
</style>
