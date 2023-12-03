<script setup lang="ts">
import { TheChessboard, type BoardApi, type SquareKey } from 'vue3-chessboard';
import { Engine } from './EngineArrows';

let boardAPI: BoardApi | undefined;
let engine: Engine | undefined;
let boardConfig = {
  events: {
    select: () => {
      if (engine.bestMove) {
        boardAPI?.drawMove(
          engine.bestMove.slice(0, 2) as SquareKey,
          engine.bestMove.slice(2, 4) as SquareKey,
          'paleBlue',
        );
      }
    },
    move: () => {
      boardAPI.hideMoves();
    },
  },
};

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
  <TheChessboard
    :board-config="boardConfig"
    @board-created="handleBoardCreated"
    @move="handleMove"
  />
</template>
