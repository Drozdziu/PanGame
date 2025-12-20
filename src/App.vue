<script setup lang="ts">
import { onMounted, ref } from 'vue';
import cardsDeckJSON from './cardsDeck.json'
import CardComp from './components/CardComp.vue';
interface card {
  name: string,
  value: number,
};
const cardsDeck : card[] = cardsDeckJSON;
const playerDeck = ref<card[]>([]);
const bot1Deck = ref<card[]>([]);
const bot2Deck = ref<card[]>([]);
const bot3Deck = ref<card[]>([]);
const mainDeck = ref<card[]>([]);
const decks = [playerDeck, bot1Deck, bot2Deck, bot3Deck];
mainDeck.value.push({ name: 'koszulka', value: -1 });

const wait = (ms: number) => new Promise<void>(resolve => setTimeout(resolve, ms));

onMounted(() => { // rozdanie kart
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 6; j++) {
      let rand = Math.floor(Math.random() * cardsDeck.length)
      if (cardsDeck[rand]) decks[i]?.value.push(cardsDeck[rand])
      cardsDeck.splice(rand, 1);
    }
    decks[i]!.value = sortDeck(decks[i]?.value!)
  }
  for (let j = 0; j < playerDeck.value.length; j++) { // czy gracz posiada 9 kier
    if (playerDeck.value[j]?.name == 'NineOfHearts') return
  }
  startBotPlay();
})
const playCard = (playingCard: card, index: number) => { // rzucenie karty
  let deckLength = mainDeck.value.length;
  if (mainDeck.value[0]?.value == -1) { // poczatek gry
    if (playingCard.name == 'NineOfHearts') {
      mainDeck.value[0] = playingCard
      playerDeck.value.splice(index, 1);
      startBotPlay();
    } else {
      startBotPlay();
    }
  }
  else { // normalny rzut
    if (playingCard.value >= mainDeck.value[deckLength - 1]!.value) {
      mainDeck.value.push(playingCard)
      playerDeck.value.splice(index, 1);
      startBotPlay();
    }
  }
}
const sortDeck = (deck: card[]) => { // bubble sort talii
  for (let i = 0; i < deck.length - 1; i++) {
    for (let j = 0; j < (deck.length - i - 1); j++) {
      if (deck[j]!.value <= deck[j + 1]!.value) {
        let temp = deck[j]!;
        deck[j] = deck[j + 1]!;
        deck[j + 1] = temp;
      }
    }
  }
  return deck;
}
async function startBotPlay() { // kolej botow
  for (let i = 1; i < 4; i++) {
    if (decks[i]?.value.length! > 0) {
      await wait(1000);
      const result = botPlay(decks[i]?.value!);
      if (result) decks[i]!.value = result;
    }
  }
  if(playerDeck.value.length == 0) startBotPlay();
}
const botPlay = (deck: card[]) => { // zagranie bota
  let deckLength = mainDeck.value.length;
  for (let i = deck.length - 1; i >= 0; i--) {
    if (mainDeck.value[0]?.value == -1) { // poczatek gry
      for (let j = 0; j < deck.length; j++) {
        if (deck[j]?.name == 'NineOfHearts') {
          mainDeck.value[0] = deck[j]!
          deck.splice(j, 1);
          return deck;
        }
      }
      return deck;
    }
    else if (deck[i]?.value! >= mainDeck.value[deckLength - 1]?.value!) { // normalna gra
      mainDeck.value.push(deck[i]!);
      deck.splice(i, 1);
      deck = sortDeck(deck);
      return deck;
    }
  }
  return take3cards(deck, false) // jak nie moze zagrac
}
const take3cards = (deck: card[], playerPlay: boolean) => { // wziecie 3 kart z talii
  let deckLength = mainDeck.value.length;
  for (let i = deckLength - 1; i > deckLength - 4; i--) {
    if (mainDeck.value[i]?.name == 'NineOfHearts') return
    deck.push(mainDeck.value[i]!);
    mainDeck.value.splice(i, 1);
  }
  deck = sortDeck(deck);
  if (playerPlay == true) startBotPlay();
  return deck;
}
</script>

<template>
  <div id="board">
    <div id="first" class="cards horizontal">
      <CardComp v-for="card in bot1Deck" :card="card.name" :isPlayer="false" />
    </div>
    <div class="cards vertical">
      <CardComp v-for="card in bot2Deck" :card="card.name" :isPlayer="false" />
    </div>
    <div class="cards mainDeck">
      <img :src="`http://localhost:5173/src/assets/Cards/${c.name}.png`" v-for="c in mainDeck.slice(-3)"></img>
    </div>
    <div class="cards vertical">
      <CardComp v-for="card in bot3Deck" :card="card.name" :isPlayer="false" />
    </div>
    <div id="first" class="cards horizontal">
      <CardComp v-for="(card, i) in playerDeck" :card="card.name" :isPlayer="true" @click="playCard(card, i)" />
      <button @click="take3cards(playerDeck, true)">Take 3 cards</button>
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

.mainDeck img {
  margin-left: -40px;
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
