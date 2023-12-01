<script setup lang="ts">
import { TheChessboard, type BoardApi } from 'vue3-chessboard';
import { Engine } from './Engine';

let boardAPI: BoardApi | undefined;
let EngineC: Engine | undefined;
let boardConfig = {
  events: {
    // select: () => {
    //   if (EngineC.bestMove) {
    //     boardAPI?.drawMove(
    //       EngineC.bestMove.slice(0, 2),
    //       EngineC.bestMove.slice(2, 4),
    //       'paleBlue'
    //     );
    //   }
    // },
    // move: () => {
    //   boardAPI.hideMoves();
    // },
  },
};

function handleBoardCreated(boardApi: BoardApi) {
  boardAPI = boardApi;

  EngineC = new Engine(boardApi);
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

  EngineC?.sendPosition(moves.join(' '));
}
</script>

<template>
  <TheChessboard
    @board-created="handleBoardCreated"
    @move="handleMove"
    :board-config="boardConfig"
    :player-color="'white'"
  />
</template>
