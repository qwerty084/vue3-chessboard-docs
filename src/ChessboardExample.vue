<script setup lang="ts">
import { ref } from 'vue';
import { TheChessboard, type BoardApi } from 'vue3-chessboard';

let boardAPI: BoardApi | undefined;
const opening = ref('');

async function getOpening() {
  opening.value = await boardAPI?.getOpeningName();
}
</script>

<template>
  <div>
    <p>Opening: {{ opening }}</p>
  </div>
  <section class="sect">
    <button class="button" @click="boardAPI?.toggleOrientation()">
      Toggle orientation
    </button>
    <button class="button" @click="boardAPI?.resetBoard()">Reset</button>
    <button class="button" @click="boardAPI?.undoLastMove()">Undo</button>
    <button class="button" @click="boardAPI?.toggleMoves()">Threats</button>
    <button class="button" @click="getOpening">Opening</button>
  </section>
  <TheChessboard @board-created="(api) => (boardAPI = api)" />
</template>

<style scoped>
.sect {
  margin-bottom: 1rem;
  display: flex;
  justify-content: center;
  gap: 3%;
}
</style>
