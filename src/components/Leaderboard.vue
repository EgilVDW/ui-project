<script setup>
import { ref } from 'vue';

const leaderboard = ref([]);
const leaderboardError = ref(false);

const getLeaderboard = () => {
  leaderboardError.value = false;
  fetch('https://mmd-3sem-default-rtdb.europe-west1.firebasedatabase.app/leaderboard.json', {
    method: 'GET'
  })
  .then((rawResponse) => {
    return rawResponse.json();
  })
  .then((response) => {
    leaderboard.value.length = 0;
    Object.values(response).forEach(score => {
      leaderboard.value.push(score);
    });
    leaderboard.value.sort((a, b) => {
      return b.score - a.score;
    })
  })
  .catch(() => {
    leaderboardError.value = true;
  })
};

getLeaderboard();
</script>

<template>
  <main>
    <div class="container">
      <h2>Leaderboard</h2>
      <div v-if="!leaderboardError" class="box" v-for="(score) in leaderboard">
        <div class="flex-row">
        <p>
          {{ score.name }}
        </p>
        <p>
          {{ score.score }} wpm
        </p>
        </div>
      </div>
      <p v-else="leaderBoard">Error: Could not connect to database</p>
    </div>
  </main>
</template>
