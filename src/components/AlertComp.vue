<script setup lang="ts">
const emit = defineEmits(['restart']);
const playAgain = (play: boolean) =>{
    emit('restart', play);
}
</script>

<template>
    <div id="alert">
        <h1>Game over!</h1>
        <h2>Do you want play again?</h2>
        <div>
            <button @click="playAgain(true)">Yes</button>
            <button @click="playAgain(false)">No</button>
        </div>
    </div>
</template>

<style scoped>
  #alert{
    display: flex;
    flex-direction: column;
    border-radius: 10px;
    max-width: 500px;
    max-height: 300px;
    background-color: white;
    color: black;
    padding: 5px;
  }
  h1, h2{
    margin: 2px;
  }
  button{
    margin: 20px;
    margin-bottom: 0px;
    background-color: black;
    color: green;
    transition: 300ms;
  }
  button:hover{
    background-color: green;
    color: black;
    scale: 1.1;
  }
</style>
