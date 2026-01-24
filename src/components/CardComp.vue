<script setup lang="ts">
import { computed} from 'vue';
const props = defineProps(['card', 'isPlayer'])
const cardType = computed(() => {
  if(props.isPlayer){
    if (!props.card) return '';
    return new URL(`../assets/Cards/${props.card}.png`, import.meta.url).href;
  }
  else return new URL(`../assets/Cards/koszulka.png`, import.meta.url).href;
});
</script>

<template>
    <img :src=cardType class="playerCards" v-if="props.isPlayer" ></img>
    <img :src=cardType v-else ></img>
</template>

<style>
  img{
    margin: 0;
    padding: 0;
    border: 3px solid black;
    border-radius: 5px;
    transition: 200ms;
  }
  .playerCards:hover{
    scale: 1.05;
  }
</style>
