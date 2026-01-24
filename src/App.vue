<script setup>
import { ref, watch } from 'vue';
import AppBots from './AppBots.vue';
import AppPeer from './AppPeer.vue';
const site = ref(0);
const peerIsShown = ref(false);
watch(site, (newVal) => {
  if (newVal === 2) {
    peerIsShown.value = true;
    console.log(peerIsShown.value, "nowa wartosc");
  }
});
const playerLeave = () =>{ site.value = 0; }
</script>
<template>
    <div v-if="site==1">
        <AppBots/>
        <button @click="site=0">Leave</button>
    </div>
    <div v-else-if="site==2">
        <AppPeer :siteIsShown="peerIsShown" @leave="playerLeave"/>
        <button @click="site=0, peerIsShown=false" >Leave</button>
    </div>
    <div v-else>
        <h1>Choose what variant you want to play</h1>
        <button @click="site=1">With bots</button>
        <button @click="site=2">With players</button>
    </div>
</template>