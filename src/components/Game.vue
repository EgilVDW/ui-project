<script setup>
import { ref, watch } from 'vue';
import { useRouter } from 'vue-router';

let quotes = [];
const quote = ref('');
const quoteError = ref(false);
const quoteFound = ref(false);

const getQuote = () => {
  fetch('https://mmd-3sem-default-rtdb.europe-west1.firebasedatabase.app/quotes.json', {
    method: 'GET'
  })
  .then((rawResponse) => {
    return rawResponse.json();
  })
  .then((response) => {
    Object.values(response).forEach(quote => {
      quotes.push(quote);
    });
    quote.value = quotes[Math.floor(Math.random() * quotes.length)].text;
  })
  .catch(() => {
    quoteError.value = true;
  })
  .finally(() => {
    if (!quoteError.value) {
      quoteFound.value = true;
    }
  })
};

getQuote();

const inputQuote = ref('');
const inputDisabled = ref(false);

let timerInterval;
let rawTimer;
const displayTimer = ref('');
let timerActive;

const startTimer = () => {
  let startTime = new Date();
  timerInterval = setInterval(() => {
    rawTimer = (new Date() - startTime) / 1000;
    displayTimer.value = Math.floor(rawTimer);
  })
  timerActive = true;
};

const stopTimer = () => {
  clearInterval(timerInterval);
};

const score = ref('');
let showScore;

const getScore = () => {
  score.value = (quote.value.split(' ').length / rawTimer * 60).toFixed(0);
  showScore = true;
};

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

const name = ref('');
const formDisabled = ref(false);
const formSubmitted = ref(false);
const nameError = ref(false);
const nameResponse = ref('');

const isAlphanumeric = (input) => {
  if (/^\w+$/.test(input)) {
    return true;
  }
};

const onSubmitForm = () => {
  if (!formDisabled.value) {
    if (name.value.length <= 20 && name.value !== '' && isAlphanumeric(name.value)) {
      formDisabled.value = true;
      nameError.value = false;
      nameResponse.value = '';
      fetch('https://mmd-3sem-default-rtdb.europe-west1.firebasedatabase.app/leaderboard.json', {
        method: 'POST',
        body: JSON.stringify({
          score: score.value,
          name: name.value
        })
      })
      .then(() => {
        getLeaderboard();
      })
      .catch(() => {
        formDisabled.value = false;
        nameError.value = true;
        nameResponse.value = 'Error: Could not connect to database'
      })
      .finally(() => {
        if (!nameError.value) {
          formSubmitted.value = true;
        }
      })
    }
    else {
      nameError.value = true;
      if (name.value.length > 20) {
        nameResponse.value = 'Name cannot exceed 20 characters';
      }
      else if (name.value === '') {
        nameResponse.value = 'Name must contain at least 1 character';
      }
      else {
        nameResponse.value = 'Name must contain valid characters';
      }
    } 
  }
};

watch(inputQuote, () => {
  if (!timerActive) {
    startTimer();
  }
  if (inputQuote.value === quote.value) {
    inputDisabled.value = true;
    stopTimer();
    getScore();
  }
});

const errorColor = ref('#7F7');
const successColor = ref('#F77');

const charColor = (index) => {
  if (inputQuote.value.charAt(index) === quote.value[index] && inputQuote.value.charAt(index) !== '') {
    return errorColor.value;
  }
  else if (inputQuote.value.charAt(index) !== quote.value[index] && inputQuote.value.charAt(index) !== '') {
    return successColor.value;
  }
};

const charDecoration = (index) => {
  if (inputQuote.value.charAt(index) !== '') {
    return 'underline';
  }
  else {
    return 'none';
  }
};

const router = useRouter();

const restart = () => {
  router.go();
};
</script>

<template>
  <main>
    <h2 v-if="!quoteError && !quoteFound" id="loading">Loading...</h2>
    <div v-else class="container">
      <div v-if="quoteFound">
        <div class="box">
          <h2 inert>
            <span v-for="(character, index) in quote" :key="index" :style="{ color: charColor(index), textDecoration: charDecoration(index) }">
              {{ character }}
            </span>
          </h2>
        </div>
        <input type="text" placeholder="Begin by typing the above text" onpaste="return false;" ondrop="return false;" autocomplete="off" spellcheck="false" v-model="inputQuote" :disabled="inputDisabled"/>
        <div class="flex-row">
          <div>
            <p v-if="timerActive">
              Time: {{ displayTimer }} seconds
            </p>
          </div>
          <button @click="restart">Restart</button>
        </div>
        <p v-if="showScore">
          {{ score }} words per minute!
        </p>
      </div>
      <div v-else class="flex-row">
        <h2>Error: Could not connect to database</h2>
        <button @click="restart">Retry</button>
      </div>
      <div v-if="showScore">
        <form v-if="!formSubmitted" v-on:submit.prevent="onSubmitForm">
          <div class="flex-row" id="submitScore">
            <input type="text" placeholder="Enter name" maxlength="20" spellcheck="false" v-model="name" :disabled="formDisabled"></input>
            <button type="submit" :disabled="formDisabled">Submit</button>
          </div>
        </form>
        <p v-if="nameError">{{ nameResponse }}</p>
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
    </div>
  </main>
</template>
