<template>
    <div class="container">
      <button class="button is-primary is-fullwidth is-relative"
              id="main-button"
              @click="buttonClick"
              @mousedown="touchStart"
              @mouseup="touchEnd"
              @touchstart="touchStart"
              @touchend="touchEnd">
        <span id="main-button-text" v-html="buttonText()"></span>
        <div class="is-absolute is-centered" style="top: 0; left: 0; right: 0;">
          Level: {{ this.displayedLevel }}
          &#xb7;
          Tries: {{ tries }}
        </div>
      </button>
    </div>
</template>

<script>

const STDDEV = 80;
const LEVELS = [
  [300, 250, 300, 250, 300],
  [100, 250, 300, 250, 100],
  [100, 250, 100, 250, 100, 250, 250, 250, 250, 250, 250],
]

export default {
  data () {
    return {
      currentLevel: 0,
      tries: 0,
      currentState: 'TEACH',
      timer: [],
      touchTime: null,
      pauseTime: null
    };
  },
  computed: {
    displayedLevel () {
      return this.currentLevel + 1
    },
    matchWidth () {
      return this.timer.slice(
        Math.max(
          this.timer.length - this.levelPattern.length,
          0
        )
      )
    },
    levelPattern () {
      if (this.currentLevel < LEVELS.length) {
        return LEVELS[this.currentLevel % LEVELS.length]
      }

      // Deterministic Pseudo-Random Number Generator
      // TODO: Make more human-made levels

      let prng = Math.sin(this.currentLevel) * 10000
      prng -= Math.floor(prng)

      prng *= 10**5
      prng *= 10**(Math.floor(this.currentLevel / 10))

      let levelSeed = "" + Math.round(prng)

      let rndLevel = []
      for (let i = 0; i < levelSeed.length; i++) {
        let ii = parseInt(levelSeed[i])

        let val = [100, 100, 300, 500][ii % 4]

        rndLevel.push(val)
        rndLevel.push(250)
      }

      console.log(rndLevel.slice(0, -1))
      return rndLevel.slice(0, -1)
    }
  },
  methods: {
    buttonText () {
      if (this.currentState == 'TEACH') {
        return "Level " + this.displayedLevel;
      }

      let text = "";

      if (this.currentState == 'RETRY') {
        text = `
          Try Again!
          <br/>
          <br/>
        `;
      }

      let idx = 1;
      for (let i = 0; i < this.matchWidth.length; i++) {
        idx = 1 - idx;

        let tx = ""
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

          if (Math.abs(this.matchWidth[i] - this.levelPattern[i]) > 1.5 * STDDEV) {
            color = 'danger'
          } else if (Math.abs(this.matchWidth[i] - this.levelPattern[i]) > STDDEV) {
            color = 'warning'
          }
        } else {
          // Pause

          if (this.matchWidth[i] - this.levelPattern[i] > 2 * STDDEV) {
            tx = "&#x2307;"
            color = 'danger'
          } else {
            continue
          }
        }

        text += `
          <span class="has-text-${color}">${tx}</span>
        `;
      }

      return text;
    },
    buttonClick (e) {
      if (this.currentState != 'LEARN') {
        this.vibrate();
        this.currentState = 'LEARN';
        this.tries += 1;
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
    validateTime () {
      if (this.timer.length < this.levelPattern.length) {
        return;
      }

      let std = 0.0;
      let ct = 0
      let tooLongPause = false
      for (let i = 0; i < this.levelPattern.length; i++) {
        if (i % 2 == 0) {
          // Touch
          std += (this.matchWidth[i] - this.levelPattern[i])**2
          ct += 1
        } else {
          // Pause
          if (this.matchWidth[i] - this.levelPattern[i] > 2 * STDDEV) {
            tooLongPause = true
            // Note: Shorter pauses are fine
          }
        }
      }
      std = (std / ct) ** 0.5

      if (std < STDDEV && !tooLongPause) {
        this.nextLevel()
      } else {
        this.retry()
      }
    },
    retry () {
      navigator.vibrate([100, 120, 100]);
      this.currentState = 'RETRY'
      this.pauseTime = null
      this.touchTime = null
    },
    nextLevel () {
      navigator.vibrate(300);
      this.currentLevel += 1;
      this.currentState = 'TEACH'
      this.timer = [];
      this.touchTime = null;
      this.pauseTime = null;
    },
    vibrate (lvl) {
      if (lvl == null)
        lvl = this.levelPattern

      let lvlDuration = lvl;
      if (Array.isArray(lvl)) {
        lvlDuration = lvl.reduce((a,b) => a+b)
      }

      console.log(lvlDuration)

      navigator.vibrate(lvl)
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

.is-relative {
  position: relative !important;
}

.is-absolute {
  position: absolute !important;
}
</style>
