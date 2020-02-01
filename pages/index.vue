<template>
  <div class="container">
    <div class
    <div class="main-app">
      <div v-if="between" class="review">
        <div class="correct-message">
          <div v-if="recentInfo.wasCorrect">Correct!</div>
          <div v-else>Incorrect.</div>
        </div>
        <div class="options">
          <img
            v-if="recentInfo.mode === 1"
            v-for="item in recentInfo.options"
            :src="
              require(`@/assets/img/${item.category}/${item.key}.${
                item.type === 'png' ? 'png' : 'svg'
              }`)
            "
            :class="
              'item' +
                (recentInfo.correct.key === item.key
                  ? ' marked-correct'
                  : recentInfo.guess.key === item.key
                  ? ' marked-incorrect'
                  : '') +
                (recentInfo.guess.key === item.key ? ' marked-selected' : '')
            "
          />
          <div
            v-if="recentInfo.mode === 2"
            v-for="item in recentInfo.options"
            class="item"
          >
            {{ item.name }}
          </div>
        </div>
        <div class="challenge">
          Which one of these is
          <span v-if="recentInfo.mode === 1" class="challenge-name">{{
            recentInfo.correct.name
          }}</span>
          <img
            v-if="recentInfo.mode === 2"
            :src="
              require(`@/assets/img/${recentInfo.correct.category}/${
                recentInfo.correct.key
              }.${recentInfo.correct.type === 'png' ? 'png' : 'svg'}`)
            "
            class="item"
          />?
        </div>
        <div class="summary">
          {{ score.correctCount }} / {{ score.attemptCount }}
        </div>
        <button @click="finishReview()" class="next-button">
          Next Question
        </button>
      </div>
      <div v-else class="question">
        <div class="options">
          <img
            v-if="mode === 1"
            v-for="item in options"
            @click="clickItem(item)"
            :src="
              require(`@/assets/img/${item.category}/${item.key}.${
                item.type === 'png' ? 'png' : 'svg'
              }`)
            "
            class="item item-clickable"
          />
          <div
            v-if="mode === 2"
            v-for="item in options"
            @click="clickItem(item)"
            class="item item-clickable"
          >
            {{ item.name }}
          </div>
        </div>
        <div class="challenge">
          Which one of these is
          <span v-if="mode === 1" class="challenge-name">{{
            correct.name
          }}</span>
          <img
            v-if="mode === 2"
            :src="
              require(`@/assets/img/${correct.category}/${correct.key}.${
                correct.type === 'png' ? 'png' : 'svg'
              }`)
            "
            class="item"
          />?
        </div>
        <div class="summary">
          {{ score.correctCount }} / {{ score.attemptCount }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const list = require('@/assets/list.json')

const randomElement = (arr) => arr[Math.floor(Math.random() * arr.length)]

const shuffle = (a) => {
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[a[i], a[j]] = [a[j], a[i]]
  }
  return a
}

let items = []

for (const category of list) {
  category.items.forEach((item) => {
    item.category = category.categoryName
  })
  items = items.concat(category.items)
}

items = items.filter((item) => item.key) // remove observers

export default {
  data() {
    return {
      correct: { key: 'default', name: 'Default' },
      options: [],
      score: {
        correctCount: 0,
        attemptCount: 0
      },
      mode: 1,
      between: false,
      recentInfo: {
        options: [],
        correct: {},
        wasCorrect: false
      }
    }
  },
  created() {
    this.options = shuffle(items).slice(0, 3)
    this.correct = randomElement(this.options)
  },
  methods: {
    clickItem(item) {
      if (item.key === this.correct.key) {
        this.score.correctCount++
        items = items.filter(({ key }) => key !== this.correct.key)
        this.newQuestion(true, item)
      } else {
        this.newQuestion(false, item)
      }
      this.score.attemptCount++
    },
    newQuestion(previousWasCorrect, guess) {
      this.recentInfo = {
        options: this.options.slice(),
        correct: this.correct,
        wasCorrect: previousWasCorrect,
        mode: this.mode,
        guess
      }

      this.mode = randomElement([1])
      this.between = true
      this.options = shuffle(items).slice(0, 3)
      this.correct = randomElement(this.options)
    },
    finishReview() {
      this.between = false
    }
  }
}
</script>

<style>
.main-app {
  width: 100vw;
  height: 80vh;
  padding: 10vh 0;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.options {
  display: flex;
  flex-direction: row;
  margin: 40px 0;
}

.item {
  width: 200px;
  height: auto;
  margin: 0 20px;
  border: 4px solid rgba(0, 0, 0, 0);
  transition: 0.5s all;
  padding: 10px;
}

.item-clickable:hover {
  cursor: pointer;
  opacity: 0.8;
  transform: translateY(-6px);
  box-shadow: 4px 0 10px 0 rgba(0, 0, 0, 0.3);
}

.summary {
  margin: 40px 0;
}

.challenge-name {
  font-weight: bold;
}

.marked-correct {
  border: 4px solid rgba(61, 156, 113, 0.7);
}

.marked-incorrect {
  border: 4px solid rgba(232, 63, 21, 1);
}

.marked-selected {
  transform: translateY(-6px);
  box-shadow: 4px 0 10px 0 rgba(0, 0, 0, 0.3);
}

.next-button {
  background: #fff;
  border: 2px solid #000;
  font-size: 24px;
  padding: 20px;
  transition: 0.5s all;
}

.next-button:hover {
  cursor: pointer;
  transform: translateY(-6px);
  box-shadow: 4px 0 10px 0 rgba(0, 0, 0, 0.3);
}

.review {
  position: relative;
}

.correct-message {
  position: absolute;
}
</style>
