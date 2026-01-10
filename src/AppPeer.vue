<script setup>
import { onMounted, ref } from 'vue';
import Peer from "peerjs";
import cardsDeckJSON from './cardsDeck.json'
// interface card {
//   name: string,
//   value: number,
// };

const peer = ref(null);
const conn = ref(null);
const peerID = ref(null)
const remoteId = ref(null);

const playerDeck = ref([]);
const enemyDeck = ref([]);
const mainDeck = ref([]);
const cardsDeck = ref([...cardsDeckJSON]);
const decks = ref([playerDeck, enemyDeck]);
const canPlay = ref(false);

onMounted(() => {
  peer.value = new Peer(Math.random().toString(36).slice(2, 6));
  peer.value.on("open", id => {
    console.log(id, "twoje id")
    peerID.value = id;
  });
  peer.value.on("connection", connection => {
    conn.value = connection;
    setupConnection(connection);
  });
})

const setupConnection = (connection) => { // 🔹 Obsługa połączenia
  connection.on("open", () => {
    console.log("Połączono z:", connection.peer);
  });
  connection.on("data", data => { // otrzymanie od innego gracza informacji
    switch (data.type) {
      case "playerPlayer":
        enemyDeck.value = data.deck;
        break;
      case "enemyPlayer":
        playerDeck.value = data.deck;
        break;
      case "mainDeck":
        mainDeck.value = data.deck;
        break;
      case "turn":
        canPlay.value = data.turn;
        break;
    }
    playerDeck.value = sortDeck(playerDeck.value);
  });
}
const sendMessage = () => {
  if (!conn.value) return;
  conn.value.send({ type: "enemyPlayer", deck: enemyDeck.value });
  conn.value.send({ type: "playerPlayer", deck: playerDeck.value });
  conn.value.send({ type: "mainDeck", deck: mainDeck.value });
  conn.value.send({ type: "turn", turn: !canPlay.value })
}
const connectToPeer = () => { // click button 
  if (!remoteId.value) return;

  const connection = peer.value.connect(remoteId.value);
  conn.value = connection;
  setupConnection(connection);
  connection.on("open", () => {
    dealTheCards();
    sendMessage();
    for (let i = 0; i < playerDeck.value.length; i++) {
      if (playerDeck.value[i].name == 'NineOfHearts') {
        canPlay.value = true;
        conn.value.send({ type: "turn", turn: !canPlay.value })
        return;
      }
    }
  });
}
const playCard = ((card, i) => {
  if (canPlay.value) {
    if (mainDeck.value[0] == undefined) {
      if (card.name == 'NineOfHearts') {
        canPlay.value = false;
        mainDeck.value.push(card)
        playerDeck.value.splice(i, 1)
        canPlay.value = false;
        sendMessage()
      }
    }
    else if (card.value >= mainDeck.value[mainDeck.value.length - 1].value) {
      if (fourCards(playerDeck.value, card.value) == 4) {
        let deckLength = playerDeck.value.length, saveIndex = 0;
        for (let i = 0; i < deckLength; i++) {
          if (playerDeck.value[i].value == card.value) {
            mainDeck.value.push(playerDeck.value[i])
            saveIndex = i;
          }
        }
        playerDeck.value.splice(saveIndex - 3, 4);
        canPlay.value = false;
        sendMessage();
      }
      else {
        canPlay.value = false;
        mainDeck.value.push(card)
        playerDeck.value.splice(i, 1)
        canPlay.value = false;
        sendMessage()
      }

    }
  }
})
const dealTheCards = () => { // dealing cards
  for (let i = 0; i < 2; i++) {
    for (let j = 0; j < 12; j++) {
      let rand = Math.floor(Math.random() * cardsDeck.value.length)
      if (cardsDeck.value[rand]) decks.value[i].value.push(cardsDeck.value[rand])
      cardsDeck.value.splice(rand, 1);
    }
    decks.value[i].value = sortDeck(decks.value[i].value)
  }
  mainDeck.value = cardsDeck.value;
}
const sortDeck = (deck) => { // bubble sort deck
  for (let i = 0; i < deck.length - 1; i++) {
    for (let j = 0; j < (deck.length - i - 1); j++) {
      if (deck[j].value <= deck[j + 1].value) {
        let temp = deck[j];
        deck[j] = deck[j + 1];
        deck[j + 1] = temp;
      }
    }
  }
  return deck;
}
const take3cards = (deck) => { // take 3 cards from deck
  let deckLength = mainDeck.value.length;
  canPlay.value = false;
  if (deckLength > 1) {
    for (let i = deckLength - 1; i > deckLength - 4; i--) {
      if (mainDeck.value[i]?.name == 'NineOfHearts') {
        sendMessage();
        return sortDeck(deck);
      }
      deck.push(mainDeck.value[i]);
      mainDeck.value.splice(i, 1);
    }
    sendMessage()
    return sortDeck(deck);
  }
}
const fourCards = (deck, value) => {
  let result = 0;
  for (let i = 0; i < deck.length; i++) {
    if (deck[i].value == value) result++;
  }
  return result;
}
</script>

<template>
  <div id="connect" v-if="!conn">
    <span>Your id: {{ peerID }}</span>
    <input v-model="remoteId" type="text">
    <button @click="connectToPeer">Connect</button>
  </div>
  <div id="board" v-else>
    <div id="first" class="cards horizontal">
      <img :src="`http://localhost:5173/src/assets/Cards/koszulka.png`" v-for="(card, i) in enemyDeck"></img>
    </div>
    <!-- <div class="cards vertical">
      <img :src="`http://localhost:5173/src/assets/Cards/koszulka.png`" v-for="(card, i) in enemyDeck"></img>
    </div> -->
    <div class="cards mainDeck">
      <img :src="`http://localhost:5173/src/assets/Cards/${c.name}.png`" v-for="c in mainDeck.slice(-3)"></img>
    </div>
    <!-- <div class="cards vertical">
      <img :src="`http://localhost:5173/src/assets/Cards/koszulka.png`" v-for="(card, i) in enemyDeck"></img>
    </div> -->
    <div id="first" class="cards horizontal">
      <img :src="`http://localhost:5173/src/assets/Cards/${card.name}.png`" v-for="(card, i) in playerDeck"
        class="playerCards" @click="playCard(card, i)"></img>
      <button @click="take3cards(playerDeck)" v-if="canPlay">Take 3 cards</button>
    </div>
  </div>
</template>

<style scoped>
#board {
  display: grid;
  /* grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr); */
  grid-template-columns: repeat(1, 1fr);
  gap: 50px;
  min-height: 70vh;
  min-width: 800px;
  border: 5px solid white;
  background-color: green;
  padding: 10px;
}

#connect {
  display: flex;
  flex-direction: column;
  gap: 50px;
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

img {
  margin: 0;
  padding: 0;
  border: 3px solid black;
  border-radius: 5px;
  transition: 200ms;
}

.playerCards:hover {
  scale: 1.1;
}
</style>
