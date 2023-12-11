<script setup>
import { ref } from 'vue';

let count = ref(0);
</script>

<template>
  <main>
    <container class="container">
      <h4>The current Count: </h4>
      <h1>{{ count }}</h1>
      <button @click="count++">+</button>
      <button @click="count--">-</button>
    </container>
  </main>
</template>