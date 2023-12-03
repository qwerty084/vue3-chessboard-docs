<script setup lang="ts">
import { TheChessboard, type BoardApi } from 'vue3-chessboard';

let boardAPI: BoardApi | undefined;
</script>

<template>
  <section class="sect">
    <button class="button" @click="boardAPI?.viewStart">Start</button>
    <button class="button" @click="boardAPI?.viewPrevious">Previous</button>
    <button class="button" @click="boardAPI?.viewNext">Next</button>
    <button class="button" @click="boardAPI?.stopViewingHistory">End</button>
  </section>
  <TheChessboard @board-created="(api) => (boardAPI = api)" />
</template>
