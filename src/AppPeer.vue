<script setup>
import { onBeforeUnmount, onMounted, ref, watch } from 'vue';
import Peer from "peerjs";
import cardsDeckJSON from './cardsDeck.json'
import AlertComp from './components/AlertComp.vue';
const props = defineProps(['siteIsShown']);
const emit = defineEmits(['leave']);

const peer = ref(null);
const conn = ref(null);
const connsArr = ref([]);
const peerID = ref(null);
const remoteId = ref(null);
const playersConns = ref([]);
const siteIsShown = props.siteIsShown

const playerDeck = ref([]);
const enemyDeck = ref([]);
const enemyDeck2 = ref([]);
const enemyDeck3 = ref([]);
const mainDeck = ref([]);
const cardsDeck = ref([...cardsDeckJSON]);
const decks = ref([playerDeck, enemyDeck]);
const canPlay = ref(false);
const username = ref("");
const enemyUsername = ref("test");

// --------------- Component life cycle ---------------

onMounted(() => {
  peer.value = new Peer(Math.random().toString(36).slice(2, 6));
  peer.value.on("open", id => {
    peerID.value = id;
  });
  peer.value.on("connection", connection => {
    playersConns.value.push(connection)
    conn.value = connection;
    conn.value.send({ type: "username", name: username})
    setupConnection(connection);
  });
})
window.addEventListener("beforeunload", () => { // closing window
  conn.value.send({type: "playerLeft", data: siteIsShown})
  conn.value.close();
  peer.value.destroy();
});
onBeforeUnmount(()=>{ // quit game by button
  if(!remoteId.value) return
  conn.value.send({type: "playerLeft", data: siteIsShown})
  conn.value.close();
  peer.value.destroy();
})

// --------------- Peer.js functions ---------------

const setupConnection = (connection) => { // Connection logic
  connection.on("open", () => {
    console.log("Połączono z:", connection.peer);
    conn.value.send({ type: "username", name: username.value})
  });
  connection.on("close", () => {
    conn.value.send({type: "playerLeft", data: siteIsShown})
    conn.value = null;
  });
  connection.on("error", err => {
    conn.value = null;
  });
  connection.on("data", data => { // getting data from player
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
      case "username":
        enemyUsername.value = data.name;
        break;
      case "playerLeft":
        alert("Player left the game")
        emit('leave')
        conn.value = null;
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
  if (!remoteId.value){
    alert("Invalid id")
    return;
  } 
  const connection = peer.value.connect(remoteId.value);
  connection.on("open", () => {
    conn.value = connection;
    setupConnection(connection);
    dealTheCards();
    sendMessage();
    conn.value.send({ type: "username", name: username.value})
    whoStarts();
  });
  connection.on("error", err => {
    console.log("Connection error:", err);
    if (err.type === "peer-unavailable") {
      alert("Invalid id");
    }
  });
}

// --------------- Game logic ---------------

const playCard = ((card, i) => { // play card by player
  if (canPlay.value) {
    if (mainDeck.value[0] == undefined) {
      if (card.name == 'NineOfHearts') {
        canPlay.value = false;
        mainDeck.value.push(card)
        playerDeck.value.splice(i, 1)
        sendMessage()
      }
    }
    else if (card.value >= mainDeck.value[mainDeck.value.length - 1].value) {
      if (fourCards(playerDeck.value, card.value) == 4 && confirm("Would you like to play 4 cards at once?")) {
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
  if (deckLength > 1) {
    canPlay.value = false;
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
const fourCards = (deck, value) => { // is there 4 same value cards
  let result = 0;
  for (let i = 0; i < deck.length; i++) {
    if (deck[i].value == value) result++;
  }
  return result;
}
const whoStarts = () =>{
  for (let i = 0; i < playerDeck.value.length; i++) {
    if (playerDeck.value[i].name == 'NineOfHearts') {
      canPlay.value = true;
      conn.value.send({ type: "turn", turn: !canPlay.value })
      return;
    }
  }
}

const restartGame = (par) => { // restarting game
  if (par) {
    cardsDeck.value = [...cardsDeckJSON];
    playerDeck.value = [];
    enemyDeck.value = [];
    mainDeck.value = [];
    decks.value = [playerDeck, enemyDeck];
    dealTheCards();
    whoStarts();
    sendMessage();
  }
  else{
    emit('leave');
    conn.value.send({type: "playerLeft", data: siteIsShown})
    conn.value.close();
    peer.value.destroy();
  }
}
function getCardUrl(name) { return new URL(`./assets/Cards/${name}.png`, import.meta.url).href}

// const connections = {};

// peer.on("connection", conn => {
//   connections[conn.peer] = conn;

//   conn.on("data", data => {
//     console.log("od", conn.peer, data);
//   });
// });
// function broadcast(msg) {
//   Object.values(connections).forEach(c => c.send(msg));
// }


</script>

<template>
  <div id="connect" v-if="!conn">
    <span>Your id: {{ peerID }}</span>
    <input v-model="username" type="text" placeholder="Your username">
    <input v-model="remoteId" type="text" placeholder="enter id here">
    <button @click="connectToPeer">Connect</button>
  </div>
  <div id="board" v-else>
    <p>{{ enemyUsername }}</p>
    <div id="first" class="cards horizontal">
      <img :src="getCardUrl('koszulka')" v-for="(card, i) in enemyDeck"></img>
    </div>
    <!-- <div class="cards vertical">
      <img :src="getCardUrl('koszulka')" v-for="(card, i) in enemyDeck1"></img>
    </div> -->
    <div class="cards mainDeck">
      <img v-for="c in mainDeck.slice(-3)" v-if="playerDeck.length > 0" :src="getCardUrl(c.name)"/>
      <AlertComp @restart="restartGame" v-else /></img>
    </div>
    <!-- <div class="cards vertical">
      <img :src="getCardUrl('koszulka')" v-for="(card, i) in enemyDeck3"></img>
    </div> -->
    <div id="first" class="cards horizontal">
      <img :src="getCardUrl(card.name)" v-for="(card, i) in playerDeck"
        class="playerCards" @click="playCard(card, i)"></img>
      <button @click="take3cards(playerDeck)" v-if="canPlay">Take 3 cards</button>
    </div>
    <p>{{ username }}</p>
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
button{margin: 0;}
</style>
