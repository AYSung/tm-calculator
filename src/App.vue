<script setup>
import { ref } from 'vue';
import RBCX from './components/RBCX.vue';
import TPE from './components/TPE.vue';

const selectedView = ref('TPE')
</script>

<template>
  <header style="width: 450px;">
    <h1>TPE/PLEX Calculator</h1>
  </header>
  <!-- <select v-model="selectedView">
    <option value="TPE">Therapeutic Plasma Exchange (TPE/PLEX)</option>
    <option value="RBCX">Red Blood Cell Exchange (RBCX)</option>
    <option value="CCI">Corrected Count Increment (CCI)</option>
  </select> -->
  <TPE v-if="selectedView == 'TPE'"/>
  <!-- <RBCX v-if="selectedView == 'RBCX'"/> -->
  <footer>
    by Andrew Y. Sung (last updated November 2025)
  </footer>
</template>

<style scoped>
header {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 1rem;
}

h1 {
  font-size: 3rem;
}

footer {
  margin-top: auto;
  padding-top: 4rem;
  text-align: center;
}

button.selected {
    background-color: rgb(240, 220, 39);
}

select {
  font-size: 1rem;
  font-weight: bold;
  padding: 0.5rem 1rem;
  border-radius: 1.5rem;
  background-color: rgb(240, 220, 39);
}
</style>
