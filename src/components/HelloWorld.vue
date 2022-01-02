<template>
    <div class="container">
      <button class="button is-primary is-fullwidth"
              id="main-button"
              @click="buttonClick"
              @mousedown="touchStart"
              @mouseup="touchEnd"
              @touchstart="touchStart"
              @touchend="touchEnd">
        <span id="main-button-text" v-html="buttonText()"></span>
      </button>
    </div>
</template>

<script>

const STDDEV = 80;

const LEVELS = [
  [250, 250, 250, 250, 250],
  [50, 250, 300, 250, 50],
  [50, 250, 50, 250, 50, 250, 250, 250, 250, 250, 250],
]

export default {
  data () {
    return {
      currentLevel: 0,
      currentState: 'TEACH',
      timer: [],
      touchTime: null,
      pauseTime: null
    };
  },
  computed: {
    matchWidth () {
      return this.timer.slice(
        Math.max(
          this.timer.length - this.levelPattern.length,
          0
        )
      )
    },
    levelPattern () {
      return LEVELS[this.currentLevel % LEVELS.length]
    }
  },
  methods: {
    buttonText () {
      if (this.currentState == 'TEACH') {
        return "Level " + (this.currentLevel + 1);
      }

      let text = "";

      if (this.currentState == 'RETRY') {
        text = `
          Try Again!
          <br/>
          <br/>
        `;
      }

      let idx = 0;
      for (let i = 0; i < this.matchWidth.length; i++) {
        let tx = ""
        let uline = false
        let color = 'white'

        if (idx == 0) {
          // Touch
          if (this.matchWidth[i] - this.levelPattern[i] > STDDEV) {
            tx = "&darr;"
          } else if (this.levelPattern[i] - this.matchWidth[i] > STDDEV) {
            tx = "&uarr;"
          } else {
            tx = "&#x2713;"
          }
        } else {
          // Pause
          uline = true
          if (this.matchWidth[i] - this.levelPattern[i] > STDDEV) {
            tx = "&#x21AE;"
          } else if (this.levelPattern[i] - this.matchWidth[i] > STDDEV) {
            tx = "&harr;"
          } else {
            tx = "&#x2713;"
          }
        }

        if (Math.abs(this.matchWidth[i] - this.levelPattern[i]) > 1.5 * STDDEV) {
          color = 'danger'
        } else if (Math.abs(this.matchWidth[i] - this.levelPattern[i]) > STDDEV) {
          color = 'warning'
        }

        text += `
          <span class="has-text-${color} ${uline ? "is-underlined" : ""}">${tx}</span>
        `;

        idx = 1 - idx;
      }

      return text;
    },
    buttonClick (e) {
      if (this.currentState != 'LEARN') {
        this.vibrate();
        this.currentState = 'LEARN';
      }

      e.preventDefault();
    },
    touchStart () {
      if (this.currentState != 'LEARN') {
        return
      }

      if (this.timer.length >= this.levelPattern.length) {
        this.timer = []
        this.pauseTime = null
        this.touchTime = null
      }

      if (this.pauseTime != null) {
        let time = new Date() - this.pauseTime;
        this.timer.push(time);

        this.pauseTime = null;
      }

      if (this.touchTime == null) {
        this.touchTime = new Date();
      }
    },
    touchEnd (e) {
      if (this.currentState != 'LEARN') {
        return
      }

      if (this.touchTime != null) {
        let time = new Date() - this.touchTime;
        this.timer.push(time);

        this.touchTime = null;
      }

      this.pauseTime = new Date();

      this.validateTime();

      e.preventDefault();
    },
    getStd () {
      if (this.timer.length < this.levelPattern.length) {
        return null;
      }

      let std = 0.0;
      for (let i = 0; i < this.levelPattern.length; i++) {
        std += (this.matchWidth[i] - this.levelPattern[i])**2
      }
      std = (std / this.levelPattern.length) ** 0.5

      return std;

    },
    validateTime () {
      if (this.timer.length < this.levelPattern.length) {
        return;
      }

      if (this.getStd() < STDDEV) {
        this.nextLevel()
      } else {
        this.currentState = 'RETRY'
        this.pauseTime = null
        this.touchTime = null
      }
    },
    nextLevel () {
      this.currentLevel += 1;
      this.currentState = 'TEACH'
      this.timer = [];
      this.touchTime = null;
      this.pauseTime = null;
    },
    vibrate () {
      navigator.vibrate(this.levelPattern)
    },
  }
}
</script>

<style lang="scss">
#main-button {
  height: calc(100vh - 40px);
  margin-top: 20px;
}

#main-button-text {
  font-size: 2rem;
  font-weight: 600;
}

.is-underlined {
  text-decoration: underline !important;
}
</style>
