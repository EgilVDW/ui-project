<script setup>
import { ref } from 'vue';

const title = ref('skriftracer');
const displayTitle = ref('');

const playTitle = () => {
  for (let i = 0; i < title.value.length; i++) {
    setTimeout(() => {
      displayTitle.value += title.value[i];
    }, i * 100)
  }
};

playTitle()
</script>

<template>
  <header>
    <h1>{{ displayTitle }}|</h1>
  </header>
</template>
