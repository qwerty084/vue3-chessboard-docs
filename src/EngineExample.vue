<script setup lang="ts">
import { TheChessboard, type BoardApi } from 'vue3-chessboard';
import { Engine } from './Engine';

let boardAPI: BoardApi | undefined;
let engine: Engine | undefined;

function handleBoardCreated(boardApi: BoardApi) {
  boardAPI = boardApi;

  engine = new Engine(boardApi);
}

function handleMove() {
  const history = boardAPI.getHistory(true);

  const moves = history.map((move) => {
    if (typeof move === 'object') {
      return move.lan;
    } else {
      return move;
    }
  });

  engine?.sendPosition(moves.join(' '));
}
</script>

<template>
  <div class="btn-container">
    <button class="button" @click="boardAPI?.resetBoard">Reset</button>
  </div>
  <TheChessboard
    @board-created="handleBoardCreated"
    @move="handleMove"
    :player-color="'white'"
  />
</template>

<style scoped>
.btn-container {
  display: flex;
  justify-content: center;
  margin-bottom: 16px;
}
</style>
