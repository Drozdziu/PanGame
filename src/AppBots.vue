<script setup lang="ts">
import { onMounted, ref } from 'vue';
import cardsDeckJSON from './cardsDeck.json'
import CardComp from './components/CardComp.vue';
import AlertComp from './components/AlertComp.vue';
interface card {
  name: string,
  value: number,
};
const cardsDeck = ref<card[]>([...cardsDeckJSON]);
const playerDeck = ref<card[]>([]);
const bot1Deck = ref<card[]>([]);
const bot2Deck = ref<card[]>([]);
const bot3Deck = ref<card[]>([]);
const mainDeck = ref<card[]>([]);
const decks = ref([playerDeck, bot1Deck, bot2Deck, bot3Deck]);

const canPlay = ref<boolean>(true);

const wait = (ms: number) => new Promise<void>(resolve => setTimeout(resolve, ms));

onMounted(() => {
  dealTheCards();
})
const playCard = (playingCard: card, index: number): void => { // playing card by player
  if (canPlay.value) {
    let deckLength = mainDeck.value.length;
    if (mainDeck.value[0]?.value == undefined) { // game start
      if (playingCard.name == 'NineOfHearts') {
        mainDeck.value[0] = playingCard
        playerDeck.value.splice(index, 1);
        startcardPlay();
      } else {
        startcardPlay();
      }
    }
    else { // normal play
      if (playingCard.value >= mainDeck.value[deckLength - 1]!.value) {
        if (fourCards(playerDeck.value, playingCard.value) == 4 && confirm("Would you like to play 4 cards at once?")) {
          let deckLength = playerDeck.value.length, saveIndex = 0;
          for (let i = 0; i < deckLength; i++) {
            if (playerDeck.value[i]?.value == playingCard.value) {
              mainDeck.value.push(playerDeck.value[i]!)
              saveIndex = i;
            }
          }
          playerDeck.value.splice(saveIndex - 3, 4);
          startcardPlay();
        }
        else {
          mainDeck.value.push(playingCard)
          playerDeck.value.splice(index, 1);
          startcardPlay();
        }
      }
    }
  }
}
const sortDeck = (deck: card[]): card[] => { // bubble sort deck
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
async function startcardPlay() { // bots turn
  canPlay.value = false;
  if (playerDeck.value.length > 0) {
    for (let i = 1; i < 4; i++) {
      if (decks.value[i]?.value.length! > 0) {
        await wait(1000);
        const result = cardPlay(decks.value[i]?.value!);
        if (result) decks.value[i]!.value = result;
      }
    }
    if (playerDeck.value.length == 0) {
      return;
    }
  }
  canPlay.value = true;
}
const cardPlay = (deck: card[]): card[] => { // playing card by bot
  let deckLength = mainDeck.value.length;
  for (let i = deck.length - 1; i >= 0; i--) {
    if (mainDeck.value[0]?.value == undefined) { // game start
      for (let j = 0; j < deck.length; j++) {
        if (deck[j]?.name == 'NineOfHearts') {
          mainDeck.value[0] = deck[j]!
          deck.splice(j, 1);
          return deck;
        }
      }
      return deck;
    }
    else if (deck[i]?.value! >= mainDeck.value[deckLength - 1]?.value!) { // normal game
      if (fourCards(deck, deck[i]?.value!) == 4) {
        let deckLength = deck.length, saveIndex = 0;
        for (let i = 0; i < deckLength; i++) {
          if (deck[i]?.value == deck[i]?.value!) {
            mainDeck.value.push(deck[i]!)
            saveIndex = i;
          }
        }
        deck.splice(saveIndex - 3, 4);
        return deck;
      }
      else {
        mainDeck.value.push(deck[i]!);
        deck.splice(i, 1);
        deck = sortDeck(deck);
        return deck;
      }

    }
  }
  return take3cards(deck, false) // when he cant play
}
const take3cards = (deck: card[], playerPlay: boolean): card[] => { // take 3 cards from deck
  let deckLength = mainDeck.value.length;
  for (let i = deckLength - 1; i > deckLength - 4; i--) {
    if (mainDeck.value[i]?.name == 'NineOfHearts') {
      if (playerPlay == true) startcardPlay();
      return sortDeck(deck);
    }
    deck.push(mainDeck.value[i]!);
    mainDeck.value.splice(i, 1);
  }
  if (playerPlay == true) startcardPlay();
  return sortDeck(deck);
}
const fourCards = (deck: card[], value: number): number => {
  let result = 0;
  for (let i = 0; i < deck.length; i++) {
    if (deck[i]?.value == value) result++;
  }
  return result;
}
const dealTheCards = (): void => { // dealing cards
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 6; j++) {
      let rand = Math.floor(Math.random() * cardsDeck.value.length)
      if (cardsDeck.value[rand]) decks.value[i]?.value.push(cardsDeck.value[rand])
      cardsDeck.value.splice(rand, 1);
    }
    decks.value[i]!.value = sortDeck(decks.value[i]?.value!)
  }
  for (let j = 0; j < playerDeck.value.length; j++) { // whether the player has 9 hearts
    if (playerDeck.value[j]?.name == 'NineOfHearts') return;
  }
  startcardPlay();
}
const restartGame = (par: boolean) => {
  if (par) {
    cardsDeck.value = [...cardsDeckJSON];
    playerDeck.value = [];
    bot1Deck.value = [];
    bot2Deck.value = [];
    bot3Deck.value = [];
    mainDeck.value = [];
    decks.value = [playerDeck, bot1Deck, bot2Deck, bot3Deck];
    dealTheCards();
  }
}
function getCardUrl(name : string) {
  return new URL(`./assets/Cards/${name}.png`, import.meta.url).href
}
</script>

<template>
  <div id="board">
    <div id="first" class="cards horizontal">
      <CardComp v-for="card in bot2Deck" :card="card.name" :isPlayer="false" />
    </div>
    <div class="cards vertical">
      <CardComp v-for="card in bot1Deck" :card="card.name" :isPlayer="false" />
    </div>
    <div class="cards mainDeck">
      <img v-for="c in mainDeck.slice(-3)" v-if="playerDeck.length > 0" :src="getCardUrl(c.name)"/>
      <AlertComp @restart="restartGame" v-else />
    </div>
    <div class="cards vertical">
      <CardComp v-for="card in bot3Deck" :card="card.name" :isPlayer="false" />
    </div>
    <div id="first" class="cards horizontal">
      <CardComp v-for="(card, i) in playerDeck" :card="card.name" :isPlayer="true" @click="playCard(card, i)" />
      <button @click="take3cards(playerDeck, true)" v-if="canPlay">Take 3 cards</button>
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
