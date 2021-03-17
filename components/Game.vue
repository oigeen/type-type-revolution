<template>
  <v-container>
    <div>
      <div id="top-goal-line" class="goal-line"></div>
      <div id="bottom-goal-line" class="goal-line"></div>
    </div>
    <v-row>
      <v-col v-for="(letter, i) in words[currentWord]" :key="i">
        <span @click="getY" class="letter">{{ letter }}</span>
      </v-col>
    </v-row>
    <v-btn @click="pauseGame">{{ paused ? 'continue' : 'pause' }}</v-btn>
  </v-container>
</template>

<script>
export default {
  data: () => {
    return {
      bpm: 100,
      currentWord: -1,
      keydown: false,
      paused: false,
      words: [
        'SCOTLAND',
        'IRELAND',
        'WALES',
        'ENGLAND',
        'NORWAY',
        'ICELAND',
        'SWEDEN',
        'FINLAND',
        'DENMARK',
      ],
    }
  },
  computed: {
    beatLengthInSeconds: function () {
      return 60 / this.bpm
    },
  },
  methods: {
    getY(event) {
      alert(
        'Letter Y: ' +
          event.target.getBoundingClientRect().y +
          ' | Top Goal Y: ' +
          document.getElementById('top-goal-line').getBoundingClientRect().y
      )
    },
    calculateLetterPosition(letter) {
      let letterY = letter.getBoundingClientRect().y
      let topY = document
        .getElementById('top-goal-line')
        .getBoundingClientRect().y
      let bottomY = document
        .getElementById('bottom-goal-line')
        .getBoundingClientRect().y
      if (letterY - topY < 15 && letterY - topY > 0) {
        letter.classList.add('letter-green')
        this.updateScore(100)
      } else if (topY - letterY < 40 && letterY < bottomY) {
        letter.classList.add('letter-yellow')
        this.updateScore(50)
      } else {
        letter.classList.add('letter-red')
        this.updateScore(-50)
      }
    },
    handleLetterAnimationEnd() {
      let element = document.getElementById('active-letter')
      element.removeEventListener('animationend', this.handleLetterAnimationEnd)
      if (!this.isFinalLetter(element)) {
        this.nextActiveLetter()
      }
      element.classList.add('letter-red')
      this.updateScore(-50)
    },
    generateWords() {
      this.currentWord++
      let letters
      let delay = 0
      this.$nextTick(() => {
        letters = document.getElementsByClassName('letter')
        letters[0].id = 'active-letter'
        letters.forEach((element, i) => {
          element.addEventListener(
            'animationend',
            this.handleLetterAnimationEnd,
            element,
            letters,
            i
          )
          element.classList.remove('no-animation')
          element.classList.add('initial-animation')
          element.style.animationPlayState = 'running'
          element.style.animationDelay = delay + 's'
          element.style.animationDuration =
            this.beatLengthInSeconds * 4 +
            (this.beatLengthInSeconds * 4) / 5 +
            's'
          delay += this.beatLengthInSeconds
        })
        setTimeout(() => {
          if (!this.paused && this.currentWord < this.words.length - 1) {
            this.resetLetters(letters)
            this.generateWords()
          }
        }, 8000)
      })
    },
    incorrectKeyPress() {
      this.updateScore(-50)
    },
    isFinalLetter(element) {
      return (
        element.parentNode == element.parentNode.parentNode.lastElementChild
      )
    },
    nextActiveLetter() {
      let currentActiveLetterElement = document.getElementById('active-letter')
      currentActiveLetterElement.id = ''

      let parentRow = currentActiveLetterElement.parentNode.parentNode
      let position = Array.prototype.indexOf.call(
        parentRow.children,
        currentActiveLetterElement.parentNode
      )
      let nextEl = parentRow.children[position + 1].children[0]
      nextEl.id = 'active-letter'
    },
    pauseGame() {
      this.paused = !this.paused
      document.getElementsByClassName('letter').forEach((letter) => {
        letter.style.animationPlayState == 'paused'
          ? (letter.style.animationPlayState = 'running')
          : (letter.style.animationPlayState = 'paused')
      })
    },
    keyDownHandler(event) {
      if (this.keydown) return
      this.keydown = true
      let key = event.key.toUpperCase()
      let currentActiveLetterElement = document.getElementById('active-letter')
      if (currentActiveLetterElement) {
        if (currentActiveLetterElement.innerHTML == key) {
          currentActiveLetterElement.style.animationPlayState = 'paused'
          this.calculateLetterPosition(currentActiveLetterElement)
          if (!this.isFinalLetter(currentActiveLetterElement)) {
            this.nextActiveLetter()
          }
        } else {
          this.incorrectKeyPress()
        }
      }
    },
    resetLetters(letters) {
      letters.forEach((letter) => {
        letter.className = ''
        void letter.offsetWidth
        letter.classList.add('letter')
      })
    },
    updateScore(score) {
      this.$emit('updateScore', score)
    },
  },
  mounted() {
    document.addEventListener('keydown', this.keyDownHandler)
    document.addEventListener('keyup', () => (this.keydown = false))
    this.generateWords()
  },
}
</script>

<style lang="scss" scoped>
#bottom-goal-line {
  margin-top: 50px;
}
.goal-line {
  height: 2px;
  width: 100%;
  background-color: white;
}
.letter {
  position: absolute;
  bottom: 0;
  font-size: 50px;
  line-height: 50px;
}
.initial-animation {
  animation-name: animation-initial;
  animation-timing-function: linear;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
}
.no-animation {
  animation: none !important;
}

.letter-green {
  color: green;
}
.letter-yellow {
  color: yellow;
}
.letter-red {
  color: red;
}
@keyframes animation-initial {
  0% {
    bottom: 0px;
  }
  100% {
    bottom: calc(100vh - 160px);
  }
}
</style>