<script setup lang="ts">
import { computed } from 'vue';
const props = defineProps(['card'])
const cardType = computed(() => {
  if (!props.card) return '';
  return new URL(`../assets/Cards/${props.card}.png`, import.meta.url).href;
});
// console.log(props.card)
</script>

<template>
    <img :src=cardType v-on:click="console.log(props.card)"></img>
</template>

<style>
  img{
    /* height: 10vh; */
    margin: 0;
    padding: 0;
    border: 3px solid black;
    border-radius: 5px;
    /* aspect-ratio: 3 / 4; */
  }
</style>
