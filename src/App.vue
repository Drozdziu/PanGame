<script setup lang="ts">
import { onMounted, ref } from 'vue';
import CardComp from './components/CardComp.vue';
interface card {
  name: string,
  value: number,
};
const cardsDeck: card[] = [{ name: 'AceOfHearts', value: 5 }, { name: 'AceOfDiamonds', value: 5 }, { name: 'AceOfSpades', value: 5 }, { name: 'AceOfClubs', value: 5 },
{ name: 'KingOfHearts', value: 4 }, { name: 'KingOfDiamonds', value: 4 }, { name: 'KingOfSpades', value: 4 }, { name: 'KingOfClubs', value: 4 },
{ name: 'QueenOfHearts', value: 3 }, { name: 'QueenOfDiamonds', value: 3 }, { name: 'QueenOfSpades', value: 3 }, { name: 'QueenOfClubs', value: 3 },
{ name: 'JackOfHearts', value: 2 }, { name: 'JackOfDiamonds', value: 2 }, { name: 'JackOfSpades', value: 2 }, { name: 'JackOfClubs', value: 2 },
{ name: 'TenOfHearts', value: 1 }, { name: 'TenOfDiamonds', value: 1 }, { name: 'TenOfSpades', value: 1 }, { name: 'TenOfClubs', value: 1 },
{ name: 'NineOfHearts', value: 0 }, { name: 'NineOfDiamonds', value: 0 }, { name: 'NineOfSpades', value: 0 }, { name: 'NineOfClubs', value: 0 },
];
const playerDeck = ref<card[]>([]);
const bot1Deck = ref<card[]>([]);
const bot2Deck = ref<card[]>([]);
const bot3Deck = ref<card[]>([]);
onMounted(() => {
  const decks = [playerDeck, bot1Deck, bot2Deck, bot3Deck];
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 6; j++) {
      let rand = Math.floor(Math.random() * cardsDeck.length)
      if (cardsDeck[rand]) decks[i]?.value.push(cardsDeck[rand])
      cardsDeck.splice(rand, 1);
    }
  }
})
</script>

<template>
  <div id="board">
    <div id="first" class="cards horizontal">
      <CardComp v-for="card in bot1Deck" :card="card.name" />
    </div>
    <div class="cards vertical">
      <CardComp v-for="card in bot2Deck" :card="card.name" />
    </div>
    <div class="cards">
      <img src="./assets/Cards/koszulka.png"></img>
    </div>
    <div class="cards vertical">
      <CardComp v-for="card in bot3Deck" :card="card.name" />
    </div>
    <div id="first" class="cards horizontal">
      <CardComp v-for="card in playerDeck" :card="card.name" />
    </div>
  </div>

</template>

<style scoped>
#board {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  gap: 50px;
  min-height: 70vh;
  min-width: 800px;
  border: 5px solid white;
  background-color: green;
  padding: 10px;
}

#first {
  grid-column: 1 / -1;
}

.cards {
  display: flex;
  justify-content: center;
  align-items: center;
}

.cards.horizontal img {
  margin-left: -30px;
}

.cards.vertical {
  display: flex;
  flex-direction: column;
}

.cards.vertical img {
  transform: rotate(90deg);
  margin-bottom: -60px;
  margin-top: -60px;
}
</style>
