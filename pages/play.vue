<template>
  <v-container>
    <ScoreCounter :score="score" />
    <Game @updateScore="updateScore" />
  </v-container>
</template>

<script>
import ScoreCounter from '@/components/ScoreCounter.vue'
import Game from '@/components/Game.vue'
export default {
  components: { ScoreCounter, Game },
  data: () => {
    return {
      score: 0,
    }
  },
  methods: {
    updateScore(addScore) {
      this.score += addScore
    },
  },
}
</script>

<style lang="scss" scoped>
</style>